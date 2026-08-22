# AR 交易类型、发票、贷项通知单与调整

## 原理

Transaction Source 控制编号、批次、自动发票和参考字段；Transaction Type 控制类别、Open Receivable、Post to GL、Natural Application、Accounting 和 Credit Memo 关系；AutoAccounting 从 Transaction Type、Memo Line、Salesperson、Standard Line、Tax 等来源派生账户。

发票完成后建立 Payment Schedule。Credit Memo 可对原交易的行/税/运费或余额贷记；Adjustment 直接调整应收、费用、运费或税，需审批限额和 Receivables Activity。

## 配置

1. 定义 Transaction Source/Type、AutoAccounting、Payment Terms、Memo Lines、Salesperson。
2. 配置 EBTax、Revenue Scheduling/Contingency、Invoicing/Accounting Rules、SLA。
3. 定义 Adjustment Activity、Approval Limit、Credit Memo Reason/Method、Document Sequence。
4. 对 AutoInvoice 测试唯一性、Grouping Rule、Line Ordering、Reference 和 OM 退货/贷项链路。

## SQL

```sql
SELECT rcta.customer_trx_id, rcta.trx_number, rcta.org_id,
       rctta.name trx_type, rbsa.name source,
       rcta.trx_date, rcta.complete_flag,
       rcta.bill_to_customer_id, rcta.invoice_currency_code
  FROM ra_customer_trx_all rcta
  JOIN ra_cust_trx_types_all rctta
    ON rctta.cust_trx_type_id = rcta.cust_trx_type_id
   AND rctta.org_id = rcta.org_id
  JOIN ra_batch_sources_all rbsa
    ON rbsa.batch_source_id = rcta.batch_source_id
   AND rbsa.org_id = rcta.org_id
 WHERE rcta.customer_trx_id = :p_customer_trx_id;

SELECT customer_trx_line_id, line_number, line_type,
       inventory_item_id, quantity_invoiced, unit_selling_price,
       extended_amount, tax_line_id, link_to_cust_trx_line_id,
       interface_line_context
  FROM ra_customer_trx_lines_all
 WHERE customer_trx_id = :p_customer_trx_id
 ORDER BY line_number;
```

## 排查

- AutoAccounting 失败：按 Receivable/Revenue/Tax/Freight/AutoInvoice Clearing 账户类型检查来源、CCID 和必需设置。
- 不能 Complete：检查行、GL Distribution、Tax、Terms、Salesperson、Exchange Rate 和原交易引用。
- Credit Memo 金额不允许：查原单剩余行/税/运费/应收、Natural Application 和已有 CM/Adjustment。
- Adjustment 待批准：查 Approval Limit、Adjustment Type/Activity、Reason、Amount 和审批人职责。

## 关联

- [AutoInvoice](interfaces-troubleshooting.md)
- [AR 会计](accounting-close-reports.md)
