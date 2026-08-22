# AP 会计、过账、月结与常用报表

## 会计链路

AP 发票、付款、作废、预付核销和兑兑损益通过 SLA 建立会计。常见经济分录（以实际 SLA 规则为准）：

```text
Invoice:  Dr Expense/Asset/Tax   Cr Liability
Payment:  Dr Liability           Cr Cash/Cash Clearing
Clearing: Dr Cash Clearing       Cr Cash
```

## 月结顺序

1. 冻结当期录入，确认 AP 期间范围和汇率完整。
2. 处理 Open Interface、报销、未验证发票、Holds、未审批和未完成 PPR。
3. 运行 Create Accounting（Final），处理 SLA 错误，转入 GL 并 Journal Import/Post。
4. 运行 Invoice Register、Payment Register、AP Trial Balance/Aging、Unaccounted Transactions 等报表。
5. 对账 AP Liability 子账与 GL，单独分析未转 GL、未过账和手工 GL 分录。
6. 关闭 AP 期间，再完成 GL 月结。

## SQL

```sql
-- 未过账 AP 分配
SELECT aid.org_id, aid.accounting_date, aid.posted_flag,
       aid.match_status_flag, COUNT(*) row_count, SUM(aid.amount) amount
  FROM ap_invoice_distributions_all aid
 WHERE aid.accounting_date BETWEEN :p_start_date AND :p_end_date
   AND NVL(aid.posted_flag, 'N') <> 'Y'
 GROUP BY aid.org_id, aid.accounting_date,
          aid.posted_flag, aid.match_status_flag
 ORDER BY aid.org_id, aid.accounting_date;

-- 发票支付余额
SELECT aia.org_id, aia.invoice_id, aia.invoice_num,
       aia.invoice_amount, aia.amount_paid,
       SUM(apsa.amount_remaining) amount_remaining
  FROM ap_invoices_all aia
  JOIN ap_payment_schedules_all apsa ON apsa.invoice_id = aia.invoice_id
 WHERE aia.org_id = :p_org_id
   AND aia.cancelled_date IS NULL
 GROUP BY aia.org_id, aia.invoice_id, aia.invoice_num,
          aia.invoice_amount, aia.amount_paid;

-- SLA 未转 GL
SELECT xah.application_id, xah.ledger_id,
       xah.gl_transfer_status_code,
       COUNT(*) header_count
  FROM xla_ae_headers xah
 WHERE xah.application_id = 200
   AND xah.accounting_date BETWEEN :p_start_date AND :p_end_date
 GROUP BY xah.application_id, xah.ledger_id,
          xah.gl_transfer_status_code;
```

## 对账差异排查

- 确保 AP Trial Balance 和 GL 使用同一 Ledger、OU/法人、截止日、币种和责任账户范围。
- 区分未会计、已会计未转 GL、已转未 Import、已 Import 未 Post 四个断点。
- 查 GL 中对 AP Source 的手工调整、其他 Source 过入 AP Liability 账户、反冲期间错位。
- 报表差异优先根据标准 AP Trial Balance 日志和 Oracle Support 建议排查，不直接汇总 `AP_INVOICES_ALL` 代替会计余额。

## 关联

- [SLA](../01-common/sla.md)
- [GL 月结](../04-gl/close-reports.md)
