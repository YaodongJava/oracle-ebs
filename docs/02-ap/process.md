# Oracle Payables 业务流程（P2P 子账视角）

## 流程与控制点

```text
Supplier/Site → Requisition → PO → Receipt → Invoice/Match
 → Validation → Approval → Create Accounting → Payment
 → Clearing/Reconciliation → Transfer/Post to GL → Close
```

AP 以 OU 隔离交易，发票头为 `AP_INVOICES_ALL`，行为 `AP_INVOICE_LINES_ALL`，分配为 `AP_INVOICE_DISTRIBUTIONS_ALL`。PO 匹配将发票与 `PO_*`/`RCV_*` 单据关联；Validation 检查金额、税、期间、匹配和挂起；Approval 决定业务授权；SLA 创建会计；Oracle Payments（IBY）生成支付指令和文件。

## 实施清单

1. 完成 Ledger、OU、Legal Entity、COA、期间和 MOAC。
2. 配置 Financial Options、Payables Options、Invoice Tolerances、Distribution Sets、Payment Terms、Lookups。
3. 配置供应商/地点、银行账户、Payment Process Profile、付款方法与格式。
4. 配置 EBTax、Withholding Tax、预付款、审批、文档序列和 SLA。
5. 测试 PO/Receipt Match、非 PO 发票、贷项、预付款、外币、付款、作废、对账和月结。

## 快速跟踪 SQL

```sql
SELECT aia.invoice_id, aia.invoice_num, aia.invoice_type_lookup_code,
       aia.org_id, aia.vendor_id, aia.vendor_site_id,
       aia.invoice_date, aia.gl_date, aia.invoice_currency_code,
       aia.invoice_amount, aia.amount_paid, aia.payment_status_flag,
       aia.wfapproval_status, aia.cancelled_date
  FROM ap_invoices_all aia
 WHERE aia.org_id = :p_org_id
   AND aia.invoice_num = :p_invoice_num;

SELECT aid.invoice_distribution_id, aid.invoice_line_number,
       aid.line_type_lookup_code, aid.amount, aid.dist_code_combination_id,
       aid.accounting_date, aid.posted_flag, aid.match_status_flag,
       aid.po_distribution_id, aid.rcv_transaction_id
  FROM ap_invoice_distributions_all aid
 WHERE aid.invoice_id = :p_invoice_id
 ORDER BY aid.invoice_line_number, aid.distribution_line_number;
```

## 排查顺序

- 先确认 OU、单据 ID、当前状态和业务日期，再查错误表/请求日志。
- 不能 Validation：查 Holds、期间、分配总额、税行、匹配容差和兑换率。
- 不能审批：查 AME/Workflow 规则、审批人、职位层级、通知和代理。
- 不能会计：查 XLA Event、无效 CCID、关闭期间、未解决 Hold 和 Create Accounting 日志。
- 不能付款：查 Payment Hold、到期日、银行账户权限、付款方法/PPP、供应商地点和 IBY 错误。

## 关联文档

- [AP 常用表结构与字段含义](tables.md)
- [P2P 端到端](../08-e2e/procure-to-pay.md)
- [AP 月结](accounting-close-reports.md)
- [SLA](../01-common/sla.md)
