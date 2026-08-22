# Oracle Cash Management、Payments 与 E-Business Tax 接口实现案例

## 1. 业界常用场景

| 场景 | 推荐接口 | 实施方法 |
| --- | --- | --- |
| 银行对账单 MT940/BAI2/CAMT.053 | Bank Statement Loader/Open Interface | 原始文件归档 → 映射 → Header/Line Interface → Import/Reconcile |
| 银行日内流水 CAMT.052 | Intra-Day Bank Statement Loader | 使用 Statement Timestamp/Sequence 区分同日多批 |
| 银行代收/虚拟账号 | AR AutoLockbox | 银行文件进入 `AR_PAYMENTS_INTERFACE_ALL`，再 Validate/Post |
| 付款文件与银行回执 | Oracle Payments（IBY）Format/Transmission | 使用 Payment Process Profile、BI Publisher 格式、SFTP/HTTPS 和 ACK |
| 外部资金系统交易参与对账 | Reconciliation Open Interface | 实现 `CE_999_INTERFACE_V`，供 CE 自动/手工匹配 |
| AP/AR 交易计税 | EBTax Tax Determining Factors | 从 AP/AR 标准接口传税务分类和地点，不直接 DML `ZX_*` |

## 2. 银行文件落地与幂等

银行原始文件应保存为不可变归档，并记录哈希：

```sql
CREATE TABLE xxce_bank_files (
  bank_file_id       NUMBER        NOT NULL,
  bank_account_key   VARCHAR2(100) NOT NULL,
  statement_number   VARCHAR2(100),
  file_name          VARCHAR2(255) NOT NULL,
  file_sha256        VARCHAR2(64)  NOT NULL,
  file_date          DATE,
  status             VARCHAR2(30)  NOT NULL,
  request_id         NUMBER,
  error_message      VARCHAR2(2000),
  creation_date      DATE          DEFAULT SYSDATE NOT NULL,
  last_update_date   DATE          DEFAULT SYSDATE NOT NULL,
  CONSTRAINT xxce_bank_files_pk PRIMARY KEY (bank_file_id),
  CONSTRAINT xxce_bank_files_u1 UNIQUE (bank_account_key, file_sha256)
);
```

文件名不是可靠重复键。常用重复判断是 Bank Account + Statement Number/Sequence + File Hash；同一银行重新发送修正版时应保留版本关系，不覆盖原文件。

## 3. Bank Statement Open Interface

Oracle 官方接口由 `CE_STATEMENT_HEADERS_INT` 和 `CE_STATEMENT_LINES_INTERFACE` 组成。官方说明 Header 必填项包括 Statement Number、Bank Account Number、Statement Date 和 Org ID；使用 Loader 时 `ORG_ID` 可由程序填充。

### 3.1 写入对账单头

```sql
INSERT INTO ce_statement_headers_int (
  statement_number,
  bank_account_num,
  statement_date,
  currency_code,
  control_begin_balance,
  control_end_balance,
  control_total_dr,
  control_total_cr,
  record_status_flag,
  org_id
) VALUES (
  :p_statement_number,
  :p_bank_account_number,
  :p_statement_date,
  :p_currency_code,
  :p_opening_balance,
  :p_closing_balance,
  :p_total_debit,
  :p_total_credit,
  'N',
  :p_org_id
);
```

### 3.2 写入对账单行

```sql
INSERT INTO ce_statement_lines_interface (
  statement_number,
  bank_account_num,
  line_number,
  trx_date,
  effective_date,
  trx_code,
  bank_trx_number,
  amount,
  currency_code,
  trx_text
) VALUES (
  :p_statement_number,
  :p_bank_account_number,
  :p_line_number,
  :p_transaction_date,
  :p_value_date,
  :p_bank_transaction_code,
  :p_bank_reference,
  :p_signed_amount,
  :p_currency_code,
  :p_remittance_information
);
```

`CE_STATEMENT_LINES_INTERFACE` 不包含 Header 的 `ORG_ID` 和 `RECORD_STATUS_FLAG`；组织由 Header/Bank Account 确定。不同文件格式对金额正负、Debit/Credit 和交易码的定义不同。必须通过 Bank Transaction Codes 映射验证，不能直接用银行原始符号猜测 EBS 金额方向。

### 3.3 上线前校准字段

```sql
SELECT table_name,
       column_id,
       column_name,
       data_type,
       nullable
  FROM all_tab_columns
 WHERE owner = 'CE'
   AND table_name IN ('CE_STATEMENT_HEADERS_INT',
                      'CE_STATEMENT_LINES_INTERFACE')
 ORDER BY table_name, column_id;
```

若使用 Oracle 提供的 BAI2、SWIFT940、CAMT Loader，应优先复用标准 Mapping/Loader。只有银行专有格式才开发 Custom Loader；直接插表代码也应封装在 Loader 内，并接受目标实例列校验。

## 4. Import、Reconcile 与错误对账

```text
Interface N
→ Bank Statement Import validation
→ CE_STATEMENT_HEADERS / CE_STATEMENT_LINES
→ AutoReconciliation
→ Matched/Cleared/Reconciled
```

```sql
-- 接口状态
SELECT h.statement_number,
       h.bank_account_num,
       h.statement_date,
       h.record_status_flag,
       COUNT(l.line_number) line_count,
       SUM(l.amount) line_amount
  FROM ce_statement_headers_int h
  LEFT JOIN ce_statement_lines_interface l
    ON l.statement_number = h.statement_number
   AND l.bank_account_num = h.bank_account_num
 WHERE h.statement_number = :p_statement_number
   AND h.bank_account_num = :p_bank_account_number
 GROUP BY h.statement_number, h.bank_account_num,
          h.statement_date, h.record_status_flag;

-- 导入后的 Statement
SELECT csh.statement_header_id,
       csh.statement_number,
       csh.statement_date,
       csh.control_begin_balance,
       csh.control_end_balance,
       COUNT(csl.statement_line_id) line_count,
       SUM(csl.amount) line_amount
  FROM ce_statement_headers csh
  LEFT JOIN ce_statement_lines csl
    ON csl.statement_header_id = csh.statement_header_id
 WHERE csh.statement_number = :p_statement_number
   AND csh.bank_account_id = :p_bank_account_id
 GROUP BY csh.statement_header_id, csh.statement_number,
          csh.statement_date, csh.control_begin_balance,
          csh.control_end_balance;
```

Bank Statement Import 以整张 Statement 验证；任一行失败可能导致该 Statement 不导入。错误应从 Bank Statement Interface 页面、AutoReconciliation Execution Report 和请求日志联合定位。

## 5. 外部资金交易参与 CE 对账

当交易仍保存在外部 Treasury/支付平台、但需要和 EBS Bank Statement 匹配时，实现 `CE_999_INTERFACE_V`：

```sql
CREATE OR REPLACE VIEW ce_999_interface_v AS
SELECT x.external_transaction_id row_id,
       x.bank_account_id,
       x.transaction_number,
       x.transaction_date,
       x.currency_code,
       x.amount,
       x.status
  FROM xxtreasury_open_transactions x
 WHERE x.status IN ('AVAILABLE', 'CLEARED', 'RECONCILED');
```

上例只说明 View 封装方式，真实 View 必须严格实现 Oracle Cash Management User Guide 列出的全部列、数据类型和状态语义。目标银行账户还需启用 Use Reconciliation Open Interfaces 并配置 Matching Criteria、Float Status、Clear Status。

## 6. Oracle Payments 出站文件

生产付款链路应使用 IBY 标准对象：

```text
AP Selected Invoices
→ Payment Process Request
→ Payment Instruction
→ BI Publisher Payment Format
→ Transmission Configuration
→ Bank/SWIFT
→ Technical ACK + Business ACK
→ Payment/Clearing/Reconciliation
```

不要直接插入或更新 `IBY_*` 业务表生成付款。常用实施控制：

- 每个 Payment Instruction 只生成一个版本化文件，重传不重新付款；
- 文件使用 PGP/银行签名，SFTP/HTTPS 凭据进入 Wallet/Secrets，不写源码；
- 保存 File Hash、Instruction ID、Transmission ID、银行 Message ID 和 ACK 状态；
- 区分“文件传输成功”“银行接收”“银行受理”“资金结算”四种状态；
- Positive Pay、付款回执和退票文件独立建消息类型及幂等键。

```sql
SELECT ppr.payment_service_request_id,
       ppr.calling_app_doc_unique_ref1,
       ppr.payment_service_request_status,
       pi.payment_instruction_id,
       pi.payment_instruction_status,
       pi.generate_sep_remit_advice_flag
  FROM iby_pay_service_requests ppr
  LEFT JOIN iby_pay_instructions_all pi
    ON pi.payment_service_request_id = ppr.payment_service_request_id
 WHERE ppr.payment_service_request_id = :p_payment_service_request_id;
```

具体列应在目标实例 eTRM 复核；排查时还要查 Payments/Format/Transmission 并发日志和银行 ACK。

## 7. EBTax 接口实现

税由 EBTax 根据 Regime、Tax、Status、Rate、Party/Place、Product、Fiscal Classification 和税务日期判定。外部系统通常只传 Tax Determining Factors：

```sql
-- AP 发票接口行示例片段
INSERT INTO ap_invoice_lines_interface (
  invoice_id,
  invoice_line_id,
  line_number,
  line_type_lookup_code,
  amount,
  tax_classification_code,
  ship_to_location_id,
  org_id
) VALUES (
  :p_invoice_id,
  :p_invoice_line_id,
  :p_line_number,
  'ITEM',
  :p_amount,
  :p_tax_classification_code,
  :p_ship_to_location_id,
  :p_org_id
);

-- AR AutoInvoice 行可传相同思想的税分类
UPDATE ra_interface_lines_all
   SET tax_classification_code = :p_tax_classification_code
 WHERE interface_line_id = :p_interface_line_id;
```

是否允许源系统传税额、税率或手工税行取决于产品规则、Batch Source、Configuration Owner Tax Options 和法规。不要直接写 `ZX_LINES`、`ZX_DETAIL_TAX_LINES_GT` 等 EBTax 业务/临时表。

## 8. 常见问题

| 症状 | 常见原因 | 排查方法 |
| --- | --- | --- |
| Bank Statement Import Error | Bank Account 不唯一/未定义、币种/交易码/金额无效 | 查 Interface 页面和 Import Validation Report |
| 同日对账单重复 | 未使用 Statement Number/Timestamp/File Hash | 在 Landing 层做银行账户范围唯一约束 |
| 自动对账率低 | Transaction Code、匹配规则、Reference/日期容差不合理 | 按银行交易类型分组分析未匹配原因 |
| 付款文件已传但状态未知 | 只处理技术 ACK、缺业务 ACK | 建四级状态并用银行 Message ID 查询 |
| AP/AR 税未计算或错误 | Tax Classification、Party/Place、日期、Registration 缺失 | 查 EBTax Determining Factors，不手改 ZX 行 |

## 9. 关联文档

- [现金管理和银行接口](cash-forecast-interfaces.md)
- [EBTax](ebtax.md)
- [CE/EBTax 常用表](tables.md)
- [AR AutoLockbox](../03-ar/interfaces.md)

## 10. 官方参考

- [Oracle Cash Management User Guide: Bank Statement Open Interface](https://docs.oracle.com/cd/E26401_01/doc.122/e48900/T359831T359835.htm)
- [Oracle Cash Management User Guide: Bank Statement Validation](https://docs.oracle.com/cd/E26401_01/doc.122/e48900/T359831T359836.htm)
- [Oracle Cash Management User Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48900/)
- [Oracle E-Business Tax User Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48750/)
