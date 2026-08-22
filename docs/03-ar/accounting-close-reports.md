# AR 会计、过账、月结与报表

## 会计流

```text
Transaction/Receipt/Adjustment Event
→ Create Accounting → XLA → Transfer to GL
→ Journal Import → Post → Reconcile
```

典型分录（以实际 AutoAccounting/SLA 为准）：Invoice 借 Receivable/贷 Revenue+Tax；Receipt 借 Cash/Remittance/未核销、贷 Receivable；Clear 在 Cash Clearing 与 Cash 间转换；Adjustment 在 Receivable 与 Adjustment Account 间转换。

## 月结

1. 完成 AutoInvoice/Lockbox，处理未完成交易、未识别/未核销收款、待批调整。
2. 检查发票/收款 GL Date、交易税、外币和未处理接口。
3. 运行 Create Accounting Final、Transfer to GL、Journal Import/Post。
4. 运行 Aging、Transaction/Receipt Register、Journal Entries Report、Unaccounted Transactions、AR to GL Reconciliation。
5. 对账 Receivable、Revenue、Tax、Unapplied/Unidentified、Cash/Clearing 账户，再关闭 AR 期间。

## SQL

```sql
-- 未会计/未转 GL 的 AR SLA 头
SELECT xah.ledger_id, xah.accounting_entry_status_code,
       xah.gl_transfer_status_code, COUNT(*) cnt
  FROM xla_ae_headers xah
 WHERE xah.application_id = 222
   AND xah.accounting_date BETWEEN :p_start_date AND :p_end_date
 GROUP BY xah.ledger_id, xah.accounting_entry_status_code,
          xah.gl_transfer_status_code;

-- 交易 GL 分配
SELECT customer_trx_id, account_class, latest_rec_flag,
       code_combination_id, gl_date, amount, acctd_amount,
       account_set_flag
  FROM ra_cust_trx_line_gl_dist_all
 WHERE customer_trx_id = :p_customer_trx_id
 ORDER BY cust_trx_line_gl_dist_id;

-- 当前开放余额汇总，不替代历史 Trial Balance
SELECT org_id, class, invoice_currency_code,
       SUM(amount_due_remaining) amount_remaining
  FROM ar_payment_schedules_all
 WHERE status = 'OP'
   AND org_id = :p_org_id
 GROUP BY org_id, class, invoice_currency_code;
```

## 差异排查

- 统一 Ledger/OU、As-of Date、Currency、Account Range 和 Posted 参数。
- 分别检查未会计、未转 GL、未 Import、未 Post、GL 手工调整。
- 使用 AR to GL Reconciliation 标准报表区分 Receivables/Revenue/Tax 差异，不用当前 `AMOUNT_DUE_REMAINING` 替代历史快照。
- 收款差异应同时跟踪 Receipt History、Application、CE Reconciliation 和 SLA。

## 关联

- [SLA](../01-common/sla.md)
- [GL 结账](../04-gl/close-reports.md)
