# 子分类账会计（SLA）规则、事件与过账

## 会计链路

```text
Subledger Transaction
  → XLA Transaction Entity
  → Accounting Event（XLA_EVENTS）
  → Create Accounting
  → SLA Header/Lines（XLA_AE_HEADERS / XLA_AE_LINES）
  → GL Interface / GL Journal（GL_IMPORT_REFERENCES）
  → Post → GL Balances
```

SLA 将业务事件模型与会计规则分离。Event Class/Type 描述业务事件；Journal Line Type 定义借贷方和账户类型；Account Derivation Rule/Mapping Set 派生账户；Journal Lines Definition 组合行规则；Application Accounting Definition 按应用组装；Subledger Accounting Method 最终分配到 Ledger。

## 配置与发布

1. 先使用标准 seeded 方法验证业务流程，仅在确有法定/管理需求时复制并定制规则。
2. 明确 Source 的事件类别可用性，再定义 Mapping Set/ADR/JLT/JLD/AAD/SAM。
3. 检查优先级、条件、默认值和无匹配处理，使用 Accounting Definitions Inquiry 检查组装结果。
4. 在 Accounting Setup Manager 将方法分配到 Ledger，进行 Draft 会计测试后再 Final。
5. 测试会计、反冲、调整、外币、二级账簿、转 GL、汇总和 Drilldown。

## 常用 SQL

```sql
-- 以业务交易号跟踪 XLA；SOURCE_ID_INT_1 的含义按应用/实体而异
SELECT xte.application_id, xte.entity_code, xte.entity_id,
       xte.source_id_int_1, xe.event_id, xe.event_type_code,
       xe.event_status_code, xe.process_status_code,
       xah.ae_header_id, xah.accounting_entry_status_code,
       xah.gl_transfer_status_code, xah.accounting_date
  FROM xla_transaction_entities xte
  JOIN xla_events xe
    ON xe.application_id = xte.application_id
   AND xe.entity_id = xte.entity_id
  LEFT JOIN xla_ae_headers xah
    ON xah.application_id = xe.application_id
   AND xah.event_id = xe.event_id
 WHERE xte.application_id = :p_application_id
   AND xte.entity_code = :p_entity_code
   AND xte.source_id_int_1 = :p_transaction_id;

-- SLA 分录
SELECT xah.ae_header_id, xal.ae_line_num, xal.accounting_class_code,
       xal.code_combination_id, xal.entered_dr, xal.entered_cr,
       xal.accounted_dr, xal.accounted_cr, xal.description,
       xal.gl_sl_link_id, xal.gl_sl_link_table
  FROM xla_ae_headers xah
  JOIN xla_ae_lines xal
    ON xal.application_id = xah.application_id
   AND xal.ae_header_id = xah.ae_header_id
 WHERE xah.ae_header_id = :p_ae_header_id
 ORDER BY xal.ae_line_num;

-- 通过 GL_SL_LINK 查 GL 行
SELECT gjh.je_header_id, gjh.name, gjh.status, gjh.period_name,
       gjl.je_line_num, gir.gl_sl_link_id, gir.gl_sl_link_table
  FROM gl_import_references gir
  JOIN gl_je_headers gjh ON gjh.je_header_id = gir.je_header_id
  JOIN gl_je_lines gjl
    ON gjl.je_header_id = gir.je_header_id
   AND gjl.je_line_num = gir.je_line_num
 WHERE gir.gl_sl_link_id = :p_gl_sl_link_id
   AND gir.gl_sl_link_table = :p_gl_sl_link_table;
```

## 状态与排查

- **事件未处理**：查 `XLA_EVENTS.EVENT_STATUS_CODE/PROCESS_STATUS_CODE`、交易是否已完成业务前置和 Create Accounting 日志。
- **无法派生账户**：从错误消息定位 JLT/ADR/Mapping Set，检查 Source 为 NULL、值范围、规则优先级和 CCID 有效性。
- **SLA 已有但 GL 没有**：查 `GL_TRANSFER_STATUS_CODE`、Transfer to GL/Journal Import 请求、接口错误和期间状态。
- **GL 不能 Drilldown**：检查 Transfer 汇总级别、`GL_IMPORT_REFERENCES`、`GL_SL_LINK_ID/TABLE` 和日记账来源配置。
- **Draft 与 Final 不一致**：检查两次之间交易、汇率、规则或映射是否变更。

## 官方参考

- [Oracle Subledger Accounting Implementation Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48771/)
