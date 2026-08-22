# AR 催收、账龄、坏账准备与核销

## 原理

Aging 以 `AR_PAYMENT_SCHEDULES_ALL` 的 Due Date、Amount Due Remaining 和截止日计算，必须正确纳入发票、Debit Memo、Credit Memo、Chargeback 及收款。当前余额不等于历史截止日余额；历史 Aging 需考虑截止日后的核销/调整。

Collections 可使用 Dunning Letters/Statements 或 Advanced Collections 的 Scoring、Strategy、Work Item、Promise to Pay、Dispute。坏账准备可通过 SLA/GL 调整流程实现；核销通常使用 Adjustment/Receivables Activity，需保留审批与税务证据。

## 配置

- Aging Buckets、Statement Cycle、Dunning/Collections Profile、Collector、Customer Profile Class。
- Adjustment Activity、Approval Limit、Reason、Write-off Account/SLA。
- Advanced Collections 的 Scoring Engine、Strategy Template、Work Items、Territory/Collector 分配。
- 信用损失政策需与会计准则、账龄、历史回收率和管理审批一致。

## SQL

```sql
-- 当前开放应收简表，历史 Aging 请使用标准报表
SELECT aps.org_id, aps.customer_id, aps.customer_site_use_id,
       aps.class, aps.trx_number, aps.due_date,
       TRUNC(:p_as_of_date) - TRUNC(aps.due_date) days_past_due,
       aps.invoice_currency_code, aps.amount_due_original,
       aps.amount_due_remaining
  FROM ar_payment_schedules_all aps
 WHERE aps.status = 'OP'
   AND aps.gl_date <= :p_as_of_date
   AND aps.org_id = :p_org_id
 ORDER BY aps.customer_id, aps.due_date;

SELECT customer_id, SUM(amount_due_remaining) open_amount
  FROM ar_payment_schedules_all
 WHERE org_id = :p_org_id
   AND status = 'OP'
 GROUP BY customer_id
 ORDER BY open_amount DESC;
```

## 排查

- Aging 与交易查询不一致：检查 As-of Date、GL Date、Currency、Open/Closed、截止日后 Application/Adjustment 和报表参数。
- Statement 漏单：查 Customer/Site Profile、Statement Flag/Cycle、Minimum Amount、Site Use 和交易完成状态。
- Collector 不正确：查 Profile Class/Account/Site 覆盖、Collector 有效期和 Collections 分配同步。
- Write-off 不能审批：查限额、Activity、Reason、Amount/Percentage、Period 和职责权限。

## 关联

- [Customer/Credit](customers-credit.md)
- [AR 会计](accounting-close-reports.md)
