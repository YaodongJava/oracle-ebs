# AP 预付款、员工报销、借贷项与保留款

## 场景

- **Prepayment**：Temporary 预付款可在 Settlement Date 后核销标准发票；Permanent 不用于核销。
- **Credit/Debit Memo**：减少供应商余额，可匹配 PO/原发票，应保留原始单据引用。
- **Employee Expense**：来自 Internet Expenses/报销接口，员工作为 Party/Supplier 的主数据同步是关键。
- **Retainage**：在复杂 PO/工程采购中按进度保留部分款项，释放时生成对应单据和会计。
- **Withholding Tax**：根据税组/税码和计算时点生成预扣税分配或发票。

## 控制与会计

1. 预付款的可用日、币种、供应商/地点、OU 必须与核销单据兼容。
2. 核销和取消核销是独立业务事件，需重新 Create Accounting。
3. 员工报销要检查 HR 员工有效期、默认费用账户、成本中心、银行信息和报销审批。
4. 贷项核销需区分供应商余额冲抵与实际银行退款。

## SQL

```sql
-- 预付款及可用情况
SELECT invoice_id, invoice_num, org_id, vendor_id, vendor_site_id,
       invoice_amount, amount_paid, invoice_currency_code,
       earliest_settlement_date, payment_status_flag, cancelled_date
  FROM ap_invoices_all
 WHERE invoice_type_lookup_code = 'PREPAYMENT'
   AND vendor_id = :p_vendor_id
   AND org_id = :p_org_id;

-- 核销分配线索（列含义以当前补丁级别为准）
SELECT invoice_distribution_id, invoice_id, line_type_lookup_code,
       amount, prepay_distribution_id, parent_reversal_id,
       reversal_flag, accounting_date
  FROM ap_invoice_distributions_all
 WHERE invoice_id IN (:p_prepay_invoice_id, :p_standard_invoice_id)
 ORDER BY accounting_date, invoice_distribution_id;
```

## 排查

- 预付款不可选：查是否已付款/验证，Earliest Settlement Date、剩余可用额、币种、Supplier/Site/OU。
- 取消核销失败：查后续会计/付款/期间状态，确保从最后一笔链路逆序反冲。
- 报销导入失败：查 `AP_EXPENSE_REPORT_HEADERS_ALL/LINES_ALL`、员工 Supplier 生成、GL Account、Currency/Rate 和接口拒绝原因。
- 预扣税不对：查 Supplier/Site 税组、发票日期、计算基础、阈值、会计时点和已作废单据。

## 关联

- [AP 发票](invoices.md)
- [AP 付款](payments.md)
