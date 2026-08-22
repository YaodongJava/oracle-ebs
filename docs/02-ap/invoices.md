# AP 发票录入、验证、匹配与税务

## 生命周期

```text
Enter/Import → Lines/Distributions/Tax → Match → Validate
→ Holds Released → Approve → Account → Pay/Cancel
```

常见类型有 Standard、Credit Memo、Debit Memo、Prepayment、Expense Report、Mixed。发票金额应与行/税/分配一致；PO Match 可按 PO/Receipt 及 2-way/3-way/4-way 控制数量、价格、收货和检验容差。Validation 会创建/释放系统 Hold，Approval 与 Validation 是两个不同状态。

## 关键配置

- Payables Options、Financial Options、Invoice Tolerance、Distribution Set、Payment Terms、Invoice Approval/AME。
- PO Match Option、Invoice Match Option、Receipt Routing、Accrual Method、Price/Quantity/Exchange Rate Tolerance。
- EBTax Regime/Tax/Status/Rate/Rule、Tax Classification、Recovery Rate 和 Inclusive/Exclusive Tax。
- Sequential Numbering、GL Date Basis、Future Period、Exchange Rate Type、Liability Account。

## SQL

```sql
SELECT aia.invoice_id, aia.invoice_num, aia.org_id,
       aia.invoice_type_lookup_code, aia.invoice_amount,
       aia.invoice_currency_code, aia.invoice_date, aia.gl_date,
       aia.payment_status_flag, aia.wfapproval_status,
       ap_invoices_pkg.get_approval_status(
         aia.invoice_id, aia.invoice_amount,
         aia.payment_status_flag, aia.invoice_type_lookup_code) validation_status
  FROM ap_invoices_all aia
 WHERE aia.invoice_id = :p_invoice_id;

SELECT ail.line_number, ail.line_type_lookup_code,
       ail.amount, ail.accounting_date, ail.discarded_flag,
       ail.po_header_id, ail.po_line_id, ail.rcv_transaction_id
  FROM ap_invoice_lines_all ail
 WHERE ail.invoice_id = :p_invoice_id
 ORDER BY ail.line_number;

SELECT aih.hold_lookup_code, aih.hold_reason,
       aih.hold_date, aih.release_lookup_code,
       aih.release_reason, aih.release_date
  FROM ap_holds_all aih
 WHERE aih.invoice_id = :p_invoice_id
 ORDER BY aih.hold_date;
```

## 排查

- 金额不平：比较 Header/Lines/Distributions/Tax，查舍入、已丢弃行和反冲分配。
- Matching Hold：跟踪 `PO_DISTRIBUTION_ID/RCV_TRANSACTION_ID`，对比 PO、Receipt、Correction/Return 和容差。
- Tax Hold/税不对：先查交易日期、法人/OU、供应商地点、税分类和 Place of Supply，再查 ZX 明细。
- 无法 Cancel：检查已付款、已核销预付款、已会计和 PO/Receipt 关联，使用标准取消流程。

## 关联

- [发票接口](interfaces-troubleshooting.md)
- [特殊交易](special-transactions.md)
- [P2P](../08-e2e/procure-to-pay.md)
