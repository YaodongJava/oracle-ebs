# AP 付款、付款批次、银行账户与核销

## 流程

```text
Due Installment → Payment Process Request
→ Select/Build/Validate → Payment Instruction
→ Format/Transmit/Confirm → AP Payment
→ Cash Clearing/Reconciliation → Accounting
```

R12 付款由 AP 与 Oracle Payments（IBY）共同完成。`AP_CHECKS_ALL` 保存 AP 付款，`AP_INVOICE_PAYMENTS_ALL` 关联发票与付款，`AP_PAYMENT_SCHEDULES_ALL` 保存分期应付；IBY 保存付款服务请求、指令、文件与传输状态。

## 配置

- Internal Bank/Branch/Account、OU/Legal Entity Account Use、Cash/Clearing Account。
- Payment Method、Payment Process Profile（PPP）、Format、Transmission Configuration、Payment System。
- Payment Terms、Pay Group、Priority、Currency、Pay Alone、Separate Remittance Advice。
- 付款文件、编号、签名权限、正向付款/止付和银行对账控制。

## SQL

```sql
SELECT ac.check_id, ac.check_number, ac.org_id,
       ac.vendor_id, ac.vendor_site_id, ac.bank_account_name,
       ac.currency_code, ac.amount, ac.check_date,
       ac.status_lookup_code, ac.void_date, ac.cleared_date
  FROM ap_checks_all ac
 WHERE ac.check_id = :p_check_id;

SELECT aipa.invoice_payment_id, aipa.invoice_id, aipa.check_id,
       aipa.amount, aipa.accounting_date, aipa.posted_flag,
       aipa.reversal_flag, aipa.reversal_inv_pmt_id
  FROM ap_invoice_payments_all aipa
 WHERE aipa.check_id = :p_check_id
 ORDER BY aipa.invoice_payment_id;

SELECT apsa.invoice_id, apsa.payment_num, apsa.due_date,
       apsa.gross_amount, apsa.amount_remaining,
       apsa.payment_status_flag, apsa.hold_flag
  FROM ap_payment_schedules_all apsa
 WHERE apsa.invoice_id = :p_invoice_id
 ORDER BY apsa.payment_num;
```

## 排查

- PPR 选不到发票：查 Validation/Approval、Payment Hold、Due Date、Pay Group/Priority、Currency、Payment Method、Supplier Site 和 OU。
- Build/Validation 失败：查 IBY 请求日志、收款人银行账户、PPP、内部银行账户用途和格式。
- 文件未生成/传输：查 BI Publisher 模板、Output Post Processor、Transmission Configuration、文件路径/证书和外部系统回执。
- 付款无法作废：检查是否已清算/对账、会计、传输或存在后续反冲，按标准 Void/Stop 流程处理。
- AP 与银行差异：按 Check ID/Payment Reference 对比 AP、IBY、CE Statement 和 SLA，区分 Issued、Cleared、Voided。

## 关联

- [现金管理](../07-ce-tax/cash-management.md)
- [AP 会计与结账](accounting-close-reports.md)
