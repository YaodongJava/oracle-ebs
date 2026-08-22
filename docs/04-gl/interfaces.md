# Oracle General Ledger 接口实现案例

## 1. 业界常用场景

| 场景 | 推荐接口 | 实施要点 |
| --- | --- | --- |
| 薪资、资金、费用系统生成总账凭证 | `GL_INTERFACE` + Journal Import | 每批使用独立 `GROUP_ID`，源系统单号写入 `REFERENCE*` |
| 多 ERP/海外系统汇总凭证 | 汇总层暂存表 + `GL_INTERFACE` | 同时传批次控制总额、币种和账簿，导入前验证借贷平衡 |
| 大批量人工调整 | Web ADI | 使用受控 Layout、List of Values 和职责权限 |
| 子账会计传总账 | SLA Transfer to GL | AP/AR/FA 等子账不应绕过 SLA 直接拼装 GL 分录 |
| 外部系统实时记账 | ISG 暴露受控并发程序或自定义服务 | 接口服务只入暂存/接口表，后台异步 Journal Import |

## 2. 导入前主数据与期间校验

```sql
SELECT gl.ledger_id,
       gl.name ledger_name,
       gl.currency_code,
       gps.period_name,
       gps.closing_status
  FROM gl_ledgers gl
  JOIN gl_period_statuses gps
    ON gps.set_of_books_id = gl.ledger_id
   AND gps.application_id = 101             -- General Ledger
 WHERE gl.ledger_id = :p_ledger_id
   AND gps.period_name = :p_period_name;

SELECT gcc.code_combination_id,
       gcc.enabled_flag,
       gcc.detail_posting_allowed_flag,
       gcc.start_date_active,
       gcc.end_date_active
  FROM gl_code_combinations gcc
 WHERE gcc.code_combination_id = :p_ccid
   AND gcc.chart_of_accounts_id = :p_coa_id;
```

`CLOSING_STATUS='O'` 通常表示 Open；实际允许状态还应结合 Open Future Enterable Periods、Data Access Set 和账户有效期判断。

## 3. `GL_INTERFACE` 具体实现

### 3.1 生成批次号并写入借贷行

```sql
DECLARE
  l_group_id NUMBER := gl_interface_control_s.NEXTVAL;
  l_user_id  NUMBER := fnd_global.user_id;
BEGIN
  -- 借方行
  INSERT INTO gl_interface (
    status,
    ledger_id,
    accounting_date,
    currency_code,
    date_created,
    created_by,
    actual_flag,
    user_je_source_name,
    user_je_category_name,
    group_id,
    code_combination_id,
    entered_dr,
    entered_cr,
    reference1,
    reference4,
    reference10
  ) VALUES (
    'NEW',
    :p_ledger_id,
    :p_accounting_date,
    :p_currency_code,
    SYSDATE,
    l_user_id,
    'A',
    'XX EXTERNAL',
    'Adjustment',
    l_group_id,
    :p_debit_ccid,
    :p_amount,
    NULL,
    :p_external_batch_id,
    :p_external_document_id,
    'External integration debit'
  );

  -- 贷方行
  INSERT INTO gl_interface (
    status,
    ledger_id,
    accounting_date,
    currency_code,
    date_created,
    created_by,
    actual_flag,
    user_je_source_name,
    user_je_category_name,
    group_id,
    code_combination_id,
    entered_dr,
    entered_cr,
    reference1,
    reference4,
    reference10
  ) VALUES (
    'NEW',
    :p_ledger_id,
    :p_accounting_date,
    :p_currency_code,
    SYSDATE,
    l_user_id,
    'A',
    'XX EXTERNAL',
    'Adjustment',
    l_group_id,
    :p_credit_ccid,
    NULL,
    :p_amount,
    :p_external_batch_id,
    :p_external_document_id,
    'External integration credit'
  );

  COMMIT;
  dbms_output.put_line('GROUP_ID=' || l_group_id);
END;
/
```

生产实现应先在自定义暂存表保存原始消息和幂等键，再由单一工作进程写 `GL_INTERFACE`。不要直接写 `GL_JE_HEADERS`、`GL_JE_LINES` 或 `GL_BALANCES`。

### 3.2 外币分录

外币日记账必须按实例规则提供 `CURRENCY_CONVERSION_TYPE`、`CURRENCY_CONVERSION_DATE` 和汇率，或者确保 GL Daily Rates 能派生汇率：

```sql
UPDATE gl_interface
   SET currency_conversion_type = :p_conversion_type,
       currency_conversion_date = :p_conversion_date,
       currency_conversion_rate = :p_conversion_rate
 WHERE group_id = :p_group_id
   AND currency_code <> :p_ledger_currency
   AND status = 'NEW';
```

该更新只能作为同一接口工作单元在 Journal Import 前执行，不应在 Import 已运行后修补数据。

## 4. 批次控制与幂等校验

```sql
-- 每个 Ledger、Currency、Group 必须借贷平衡
SELECT ledger_id,
       currency_code,
       group_id,
       SUM(NVL(entered_dr, 0)) total_dr,
       SUM(NVL(entered_cr, 0)) total_cr,
       SUM(NVL(entered_dr, 0) - NVL(entered_cr, 0)) difference
  FROM gl_interface
 WHERE group_id = :p_group_id
 GROUP BY ledger_id, currency_code, group_id
HAVING ABS(SUM(NVL(entered_dr, 0) - NVL(entered_cr, 0))) > 0.00001;

-- 提交前防止同一外部单据再次入接口
SELECT COUNT(*) duplicate_count
  FROM gl_interface
 WHERE user_je_source_name = 'XX EXTERNAL'
   AND reference1 = :p_external_batch_id
   AND reference4 = :p_external_document_id;
```

只查 `GL_INTERFACE` 不能覆盖已导入数据。可靠幂等应以自定义消息表唯一约束为主，并在成功表中保存 `JE_BATCH_ID/JE_HEADER_ID`。

## 5. 提交 Journal Import

先在目标实例确认并发程序 `GLLEZL` 的参数顺序；补丁、Ledger/Data Access Set 设置可能使参数定义不同。

```sql
SELECT cp.concurrent_program_name,
       dfa.column_seq_num,
       dfa.end_user_column_name,
       dfa.required_flag
  FROM fnd_concurrent_programs cp
  JOIN fnd_descr_flex_column_usages dfa
    ON dfa.application_id = cp.application_id
   AND dfa.descriptive_flexfield_name = '$SRS$.' || cp.concurrent_program_name
 WHERE cp.concurrent_program_name = 'GLLEZL'
   AND dfa.enabled_flag = 'Y'
 ORDER BY dfa.column_seq_num;
```

确认参数后，可封装为受控程序提交：

```sql
DECLARE
  l_request_id NUMBER;
BEGIN
  fnd_global.apps_initialize(:p_user_id, :p_resp_id, :p_resp_appl_id);

  l_request_id := fnd_request.submit_request(
    application => 'SQLGL',
    program     => 'GLLEZL',
    description => NULL,
    start_time  => NULL,
    sub_request => FALSE,
    argument1   => TO_CHAR(:p_interface_run_id),
    argument2   => TO_CHAR(:p_access_set_id),
    argument3   => 'XX EXTERNAL',
    argument4   => TO_CHAR(:p_ledger_id),
    argument5   => TO_CHAR(:p_group_id),
    argument6   => 'N',
    argument7   => 'N'
  );

  IF l_request_id = 0 THEN
    raise_application_error(-20040, fnd_message.get);
  END IF;
  COMMIT;
  dbms_output.put_line('REQUEST_ID=' || l_request_id);
END;
/
```

上例是常见参数骨架，不是可跳过目标实例核对的固定签名。若实例要求 `GL_INTERFACE_CONTROL`，应通过标准 Journal Import 提交流程创建 Interface Run，而不是猜测参数或自行更新控制状态。

## 6. 错误排查与成功对账

```sql
-- 接口状态及错误代码分布
SELECT status, COUNT(*) line_count
  FROM gl_interface
 WHERE group_id = :p_group_id
 GROUP BY status
 ORDER BY status;

-- 成功导入后按来源和外部批次定位 Journal
SELECT gjb.je_batch_id,
       gjb.name batch_name,
       gjh.je_header_id,
       gjh.name journal_name,
       gjh.status,
       gjh.period_name,
       SUM(NVL(gjl.entered_dr, 0)) entered_dr,
       SUM(NVL(gjl.entered_cr, 0)) entered_cr
  FROM gl_je_batches gjb
  JOIN gl_je_headers gjh ON gjh.je_batch_id = gjb.je_batch_id
  JOIN gl_je_lines gjl ON gjl.je_header_id = gjh.je_header_id
 WHERE gjh.je_source = 'XX EXTERNAL'
   AND EXISTS (
         SELECT 1
           FROM gl_import_references gir
          WHERE gir.je_header_id = gjl.je_header_id
            AND gir.je_line_num = gjl.je_line_num
            AND gir.reference_1 = :p_external_batch_id
       )
 GROUP BY gjb.je_batch_id, gjb.name, gjh.je_header_id,
          gjh.name, gjh.status, gjh.period_name;
```

Journal Import Execution Report 是错误代码的首要解释来源。常见问题包括期间未开、Source/Category 未定义、CCID 无效、外币汇率缺失、借贷不平和 Data Access Set 无权限。

## 7. 实施控制清单

- 为外部来源建立独立 Journal Source/Category，并明确是否允许冻结、审批和保留 Import Reference。
- 每个消息保存 `CORRELATION_ID`、源单号、`GROUP_ID`、Request ID、Journal ID 和批次控制总额。
- 重试只重试暂存层失败消息；不要复制仍在处理或结果未知的 `GL_INTERFACE` 行。
- 把“导入成功”“审批完成”“已过账”作为不同业务状态，分别对账。
- 日记账成功导入后仍需根据公司政策执行审批、过账和反冲。

## 8. 关联文档

- [GL 日记账、审批与过账](journals.md)
- [GL 业务流程](process.md)
- [公共 SLA](../01-common/sla.md)
- [GL 常用表](tables.md)
- [通用接口治理](../01-common/interfaces.md)

## 9. 官方参考

- [Oracle General Ledger Implementation Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48747/)
- [Oracle E-Business Suite Integrated SOA Gateway Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e20925/)
- [Oracle E-Business Suite eTRM User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/)
