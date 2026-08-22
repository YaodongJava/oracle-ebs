# Oracle Receivables 业务流程（O2C 子账视角）

## 主流程

```text
Customer/Account/Site → Order → Ship → AutoInvoice/Manual Transaction
→ Complete → Revenue/Tax → Create Accounting
→ Receipt/Lockbox → Apply/Unapply/Adjust/Refund
→ Transfer/Post GL → Reconcile → Close
```

AR 发票头/行为 `RA_CUSTOMER_TRX_ALL/RA_CUSTOMER_TRX_LINES_ALL`，会计分配为 `RA_CUST_TRX_LINE_GL_DIST_ALL`，未收分期为 `AR_PAYMENT_SCHEDULES_ALL`，收款为 `AR_CASH_RECEIPTS_ALL`，核销历史为 `AR_RECEIVABLE_APPLICATIONS_ALL`。客户主数据使用 TCA，AR 交易以 OU 隔离。

## 配置主线

1. 完成 Ledger/OU/MOAC、AR System Options、Receivables Activities、AutoAccounting。
2. 定义 Transaction Source/Type、Payment Terms、Memo Lines、Salesperson、Document Sequence。
3. 定义 Receipt Class/Method、Remittance Bank、Lockbox、AutoCash Rule Set、Application Rule Set。
4. 配置 TCA Customer、Account/Site Use、Profile/Credit、Tax、Revenue Scheduling/SLA。
5. 测试手工/AutoInvoice、Credit Memo、Receipt/Application、Chargeback、Adjustment、Refund、Unidentified/Unapplied 及外币。

## SQL

```sql
SELECT rcta.customer_trx_id, rcta.trx_number, rcta.org_id,
       rcta.cust_trx_type_id, rcta.bill_to_customer_id,
       rcta.trx_date, rcta.invoice_currency_code,
       rcta.complete_flag, rcta.previous_customer_trx_id
  FROM ra_customer_trx_all rcta
 WHERE rcta.org_id = :p_org_id
   AND rcta.trx_number = :p_trx_number;

SELECT aps.payment_schedule_id, aps.class, aps.status,
       aps.due_date, aps.amount_due_original,
       aps.amount_due_remaining, aps.amount_applied,
       aps.amount_credited, aps.amount_adjusted
  FROM ar_payment_schedules_all aps
 WHERE aps.customer_trx_id = :p_customer_trx_id;
```

## 排查

- 交易不能 Complete：查 Line/Tax/Freight、GL Distribution、AutoAccounting、Payment Terms、Salesperson、Currency/Rate。
- 不能会计：查 Complete Flag、GL Date、Period、Revenue Contingency/Scheduling、XLA Event 和 CCID。
- 收款不能核销：查 Customer/Account/Site、Currency、Receipt Status、Open Balance、Application Date/GL Date。
- AR/GL 不平：按 Transaction/Receipt/Adjustment 分类，对比 SLA Transfer/Post、GL 手工分录和截止日。

## 关联

- [AR/TCA 常用表结构与字段含义](tables.md)
- [O2C 端到端](../08-e2e/order-to-cash.md)
- [AR 会计结账](accounting-close-reports.md)
