# 采购到付款（P2P）端到端

## 业务与数据链

```text
Requisition
→ PO_HEADERS/PO_LINES/PO_LINE_LOCATIONS/PO_DISTRIBUTIONS
→ RCV_SHIPMENT_HEADERS/LINES → RCV_TRANSACTIONS
→ AP_INVOICES/LINES/DISTRIBUTIONS
→ AP_INVOICE_PAYMENTS → AP_CHECKS + IBY
→ XLA → GL
```

### 关键关联

- Requisition Distribution → PO Distribution：自动创建采购单时保留来源分配。
- PO Distribution → Receipt：`RCV_TRANSACTIONS.PO_DISTRIBUTION_ID`。
- Receipt → AP Distribution：`AP_INVOICE_DISTRIBUTIONS_ALL.RCV_TRANSACTION_ID/PO_DISTRIBUTION_ID`。
- Invoice → Payment：`AP_INVOICE_PAYMENTS_ALL.INVOICE_ID/CHECK_ID`。
- Subledger → GL：XLA Entity/Event/AE Header/Line → `GL_IMPORT_REFERENCES.GL_SL_LINK_ID/TABLE`。

## 典型会计

```text
Receipt:       Dr Receiving Inspection   Cr AP Accrual
Deliver:       Dr Inventory/Expense      Cr Receiving Inspection
AP Invoice:   Dr AP Accrual/Expense/Tax Cr Liability
Payment:      Dr Liability              Cr Cash/Clearing
```

Expense Destination 可在 Receipt/Period End 应计；Inventory Destination 通常在收货交付时进入库存会计。具体分录以 Accrual Option、Destination、SLA 和税设置为准。

## 跟踪 SQL

```sql
SELECT pha.segment1 po_number, pla.line_num, plla.shipment_num,
       pda.po_distribution_id, pda.org_id,
       pda.quantity_ordered, pda.quantity_delivered,
       pda.quantity_billed, pda.quantity_cancelled,
       rt.transaction_id rcv_transaction_id,
       rt.transaction_type rcv_type, rt.quantity rcv_quantity,
       aid.invoice_id, aid.invoice_distribution_id,
       aid.amount invoice_dist_amount, aid.match_status_flag
  FROM po_headers_all pha
  JOIN po_lines_all pla ON pla.po_header_id = pha.po_header_id
  JOIN po_line_locations_all plla ON plla.po_line_id = pla.po_line_id
  JOIN po_distributions_all pda
    ON pda.line_location_id = plla.line_location_id
  LEFT JOIN rcv_transactions rt
    ON rt.po_distribution_id = pda.po_distribution_id
  LEFT JOIN ap_invoice_distributions_all aid
    ON aid.po_distribution_id = pda.po_distribution_id
 WHERE pha.po_header_id = :p_po_header_id
 ORDER BY pla.line_num, plla.shipment_num,
          pda.distribution_num, rt.transaction_id;
```

## 排错与对账

- **PO/Receipt**：查 Approval/Authorization Status、Shipment/Distribution Open Quantity、Receiving Control/Tolerance、Return/Correction 链。
- **Receipt/AP**：对比 Ordered/Received/Delivered/Billed 数量，注意 UOM Conversion、Price Correction、Exchange Rate 和税。
- **Accrual Reconciliation**：按 PO Distribution 对比 Receipt Accrual、AP Invoice、Return/Correction、Write-off，使用同一截止日。
- **重复付款**：核查 Supplier+Invoice Number+OU、PPR 状态、IBY Instruction、Check Status 和银行回执。
- 排错时优先使用 `PO_DISTRIBUTION_ID` 和 `RCV_TRANSACTION_ID`，不仅用单据号做字符串关联。

## 关联

- [端到端常用表与跨模块关联](tables.md)
- [AP Process](../02-ap/process.md)
- [Cost Accounting](../06-cost/accounting-flow.md)
