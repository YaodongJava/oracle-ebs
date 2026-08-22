# AR 收款、Lockbox、自动核销与退款

## 生命周期

```text
Receipt Enter/Lockbox → Confirm → Remit → Clear
             └→ Identify Customer → Apply / Unapply / On-account
                                      └→ Refund / Reverse
```

Receipt Class 定义 Confirmation/Remittance/Clearing 步骤，Receipt Method 关联账户和活动，AutoCash Rule Set 定义自动核销顺序。Receipt History 记录确认、托收、清算和反冲状态；Application 表中同一收款可有 Applied/Unapplied/On-account 等多行历史。

## 配置

- Receipt Class/Method、Remittance Bank Account、Receivables Activities、AutoCash Rule Set。
- Lockbox、Transmission Format、Data Record Mapping、Receipt Batch Source、AutoAssociate、Post QuickCash。
- Cross Currency Rate/Account、Unidentified/Unapplied/On-account、Bank Charges、Refund Activity。
- IBY 退款、CE 银行对账、SLA Cash/Clearing/Remittance 账户。

## SQL

```sql
SELECT acr.cash_receipt_id, acr.receipt_number, acr.org_id,
       acr.receipt_date, acr.gl_date, acr.currency_code,
       acr.amount, acr.status, acr.type,
       acr.pay_from_customer, acr.customer_site_use_id,
       acr.receipt_method_id, acr.reversal_date
  FROM ar_cash_receipts_all acr
 WHERE acr.cash_receipt_id = :p_cash_receipt_id;

SELECT ara.receivable_application_id, ara.status,
       ara.display, ara.apply_date, ara.gl_date,
       ara.amount_applied, ara.amount_applied_from,
       ara.applied_customer_trx_id, ara.applied_payment_schedule_id,
       ara.code_combination_id
  FROM ar_receivable_applications_all ara
 WHERE ara.cash_receipt_id = :p_cash_receipt_id
 ORDER BY ara.receivable_application_id;

SELECT cash_receipt_history_id, status, current_record_flag,
       trx_date, gl_date, amount, acctd_amount,
       first_posted_record_flag
  FROM ar_cash_receipt_history_all
 WHERE cash_receipt_id = :p_cash_receipt_id
 ORDER BY cash_receipt_history_id;
```

## 排查

- Lockbox 拒绝：查 Transmission Format/记录类型、金额控制总数、收款方法/银行、Customer/Invoice 引用和重复项。
- AutoCash 不核销：查 Rule Set 顺序、Customer Profile、折扣/容差、到期余额、参考匹配和币种。
- Receipt 状态不对：以 `AR_CASH_RECEIPT_HISTORY_ALL` 的 Current Record 和日期链路判断，不只看 Header Status。
- 无法 Reverse：检查后续 Refund/Chargeback、已核销交易、关闭期间和反冲日期。

## 关联

- [Cash Management](../07-ce-tax/cash-management.md)
- [AR 结账](accounting-close-reports.md)
