# 订单到收款（O2C）端到端

## 业务与数据链

```text
Customer/TCA → OE_ORDER_HEADERS/LINES
→ WSH_DELIVERY_DETAILS/ASSIGNMENTS/DELIVERIES
→ MTL_MATERIAL_TRANSACTIONS（Sales Order Issue）
→ RA_INTERFACE_LINES → RA_CUSTOMER_TRX/LINES
→ AR_PAYMENT_SCHEDULES → AR_CASH_RECEIPTS/APPLICATIONS
→ Revenue + COGS Recognition → XLA → GL
```

OM Line Workflow 控制 Book、Schedule、Pick、Ship、Invoice Interface、Close。Shipping 生成 Inventory Sales Order Issue；Invoice Interface 将 OM 行推到 AutoInvoice；AR 发票/收款进入 SLA。R12 COGS 可先记录 Deferred COGS，再按 AR Revenue Recognition 比例转为 COGS。

## SQL

```sql
SELECT ooh.order_number, ool.line_id, ool.line_number,
       ool.flow_status_code, ool.ordered_item,
       ool.ordered_quantity, ool.shipped_quantity,
       ool.invoiced_quantity, ool.org_id,
       wdd.delivery_detail_id, wdd.released_status,
       wdd.shipped_quantity wsh_shipped_quantity,
       ril.interface_line_id, ril.request_id,
       rctl.customer_trx_id, rcta.trx_number
  FROM oe_order_headers_all ooh
  JOIN oe_order_lines_all ool ON ool.header_id = ooh.header_id
  LEFT JOIN wsh_delivery_details wdd
    ON wdd.source_code = 'OE'
   AND wdd.source_line_id = ool.line_id
  LEFT JOIN ra_interface_lines_all ril
    ON ril.interface_line_context = 'ORDER ENTRY'
   AND ril.interface_line_attribute6 = TO_CHAR(ool.line_id)
  LEFT JOIN ra_customer_trx_lines_all rctl
    ON rctl.interface_line_context = 'ORDER ENTRY'
   AND rctl.interface_line_attribute6 = TO_CHAR(ool.line_id)
  LEFT JOIN ra_customer_trx_all rcta
    ON rcta.customer_trx_id = rctl.customer_trx_id
 WHERE ooh.header_id = :p_order_header_id
 ORDER BY ool.line_number, wdd.delivery_detail_id;
```

> `ORDER ENTRY` 上下文的 Attribute 映射可受实施/补丁影响，先查实际 AutoInvoice Line 的 Context/Attributes 再固化查询。

## 排错

- Order Line 卡住：查 `FLOW_STATUS_CODE`、Workflow Status Monitor、Holds、Credit Check、Scheduling/ATP、Inventory Reservation。
- Pick/Ship 失败：查 Release Status、On-hand/Reservation、Subinventory/Locator、Lot/Serial、Shipping Parameters 和 Delivery Assignment。
- Interface Trip Stop/Inventory 卡住：查 Shipping/Inventory 并发日志、Pending Material Transaction、Period、COGS Account。
- AutoInvoice 拒绝：查 Interface Error、Transaction Source/Type、Customer Bill-to、Tax、UOM、Currency、Grouping Rule。
- 收入已确认但 COGS 未转：跟踪 OM Line、Material Transaction、AR Revenue Schedule、COGS Recognition 请求与 SLA。

## 关联

- [AR Process](../03-ar/process.md)
- [Cost Accounting](../06-cost/accounting-flow.md)
