# GL 期间开关、月结、年结与报表

## 关账依赖

```text
Operational Freeze
→ AP/AR/CE/FA/INV/WIP/Cost/Projects Close & Reconcile
→ SLA Final + Transfer → GL Import/Post
→ Intercompany/Allocation/Revaluation/Translation
→ Trial Balance & Financial Statements
→ GL Close → Year-end Carry Forward
```

## 月结清单

1. 发布结账日历和截止时间，确认本期/下期输入规则。
2. 所有子账完成业务处理、库存/接口、会计、转 GL 和对账。
3. 处理 GL Interface、Suspense、未审批/未过账 Journal、Intercompany 不平。
4. 运行 Allocation/Accrual/Revaluation/Translation/Elimination，过账并复核。
5. 运行 Trial Balance、Account Analysis、FSG/BI 财务报表，保留参数和请求 ID。
6. 关闭 GL 期间。需重开时走授权与审计流程。

## 年结

- 确认 Natural Account Type，收入/费用结转 Retained Earnings，资产/负债/权益结转期初。
- 完成最终重估/折算、法人税务调整、抵销与审计分录。
- 开放新年期间前验证 Calendar 和期间，保存年末 Trial Balance/财务报表快照。

## SQL

```sql
SELECT fa.application_short_name, gps.period_name,
       gps.closing_status, gps.start_date, gps.end_date
  FROM gl_period_statuses gps
  JOIN fnd_application fa ON fa.application_id = gps.application_id
 WHERE gps.set_of_books_id = :p_ledger_id
   AND gps.period_name = :p_period_name
 ORDER BY fa.application_short_name;

SELECT gjh.je_source, gjh.je_category, gjh.status,
       COUNT(*) journal_count
  FROM gl_je_headers gjh
 WHERE gjh.ledger_id = :p_ledger_id
   AND gjh.period_name = :p_period_name
 GROUP BY gjh.je_source, gjh.je_category, gjh.status
 ORDER BY gjh.je_source, gjh.je_category, gjh.status;

SELECT status, user_je_source_name, COUNT(*) row_count
  FROM gl_interface
 WHERE ledger_id = :p_ledger_id
 GROUP BY status, user_je_source_name;
```

## 排查

- 无法关期：读取关期页面/报表指出的未完成项，定位子账、SLA、Interface 或 Journal 断点。
- Trial Balance 不平：分析 Ledger/Currency/Actual Flag/Translated Flag，检查 Suspense、Intercompany、异常 Journal 和数据完整性。
- 报表数字变动：比较两次请求之间的过账/反冲/重开期间记录，固定报表参数和数据截止时间。
- 期间误关：不更新 Period Status；评估报表/披露影响后使用标准 Reopen。

## 关联

- [Calendar/Period](../01-common/calendar-currency-period.md)
- [AP Close](../02-ap/accounting-close-reports.md)
- [AR Close](../03-ar/accounting-close-reports.md)
