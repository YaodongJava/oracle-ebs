# 端到端流程常用表与跨模块关联

## 业务说明

端到端排查不应只靠单据号。同一业务单据可拆成多个 Line、Shipment、Distribution、Receipt Transaction、Invoice Distribution、SLA Event 和 GL Line。应使用每个模块保留的源 ID，同时检查 OU/Inventory Organization/Ledger 边界。

## P2P 表链

| 表 | 中文名 | 关键关联/业务粒度 |
| --- | --- | --- |
| `PO_REQUISITION_HEADERS_ALL` | 采购申请头 | `REQUISITION_HEADER_ID`, `SEGMENT1` 申请号 |
| `PO_REQUISITION_LINES_ALL` | 采购申请行 | `REQUISITION_LINE_ID`, Item/Quantity/Destination Org |
| `PO_REQ_DISTRIBUTIONS_ALL` | 采购申请分配 | `DISTRIBUTION_ID`, `CODE_COMBINATION_ID`, Project/Task |
| `PO_HEADERS_ALL` | 采购订单头 | `PO_HEADER_ID`, `SEGMENT1`, `ORG_ID`, `AUTHORIZATION_STATUS` |
| `PO_LINES_ALL` | 采购订单行 | `PO_LINE_ID`, `LINE_NUM`, Item/Category/Unit Price |
| `PO_LINE_LOCATIONS_ALL` | PO 发运/价格行 | `LINE_LOCATION_ID`, `SHIPMENT_NUM`, Ship-to/Need-by/Quantity |
| `PO_DISTRIBUTIONS_ALL` | PO 分配 | `PO_DISTRIBUTION_ID`, Charge/Accrual/Variance Account, Destination |
| `RCV_SHIPMENT_HEADERS` | 收货批/送货头 | `SHIPMENT_HEADER_ID`, `RECEIPT_NUM` |
| `RCV_SHIPMENT_LINES` | 收货送货行 | `SHIPMENT_LINE_ID`, PO Line/Item/Quantity |
| `RCV_TRANSACTIONS` | 收货交易 | `TRANSACTION_ID`, `PARENT_TRANSACTION_ID`, `PO_DISTRIBUTION_ID` |
| `AP_INVOICE_DISTRIBUTIONS_ALL` | AP 发票分配 | `PO_DISTRIBUTION_ID`, `RCV_TRANSACTION_ID`, Amount/Accounting Date |
| `AP_INVOICE_PAYMENTS_ALL` | AP 发票付款核销 | `INVOICE_ID`, `CHECK_ID`, `AMOUNT` |

### PO 常用状态

`PO_HEADERS_ALL.AUTHORIZATION_STATUS` 常见业务含义包括 Incomplete、In Process、Pre-Approved、Approved、Rejected、Requires Reapproval。同时检查 `CANCEL_FLAG`、`CLOSED_CODE`、`FROZEN_FLAG`、`USER_HOLD_FLAG`：

| 字段/值 | 业务含义 |
| --- | --- |
| `CANCEL_FLAG='Y'` | 单据/行/发运已取消，剩余可收/可开票数量受影响 |
| `CLOSED_CODE='OPEN'` | 业务上开放，仍需结合数量和取消标志 |
| `CLOSED_CODE='CLOSED FOR RECEIVING'` | 已关闭收货 |
| `CLOSED_CODE='CLOSED FOR INVOICE'` | 已关闭开票 |
| `CLOSED_CODE='FINALLY CLOSED'` | 最终关闭，通常不允许新增收货/开票 |

`RCV_TRANSACTIONS.TRANSACTION_TYPE` 常见 `RECEIVE`、`DELIVER`、`ACCEPT`、`REJECT`、`RETURN TO RECEIVING`、`RETURN TO VENDOR`、`CORRECT`。收货是事件链，应通过 `PARENT_TRANSACTION_ID` 跟踪原收货、交付、更正与退货，不只汇总所有正数行。

## O2C 表链

| 表 | 中文名 | 关键关联/业务粒度 |
| --- | --- | --- |
| `OE_ORDER_HEADERS_ALL` | 销售订单头 | `HEADER_ID`, `ORDER_NUMBER`, `FLOW_STATUS_CODE`, `ORG_ID` |
| `OE_ORDER_LINES_ALL` | 销售订单行 | `LINE_ID`, `HEADER_ID`, `LINE_NUMBER`, `FLOW_STATUS_CODE` |
| `WSH_DELIVERY_DETAILS` | 发运交付明细 | `DELIVERY_DETAIL_ID`, `SOURCE_LINE_ID`, `RELEASED_STATUS` |
| `WSH_DELIVERY_ASSIGNMENTS` | 交付明细分组 | Detail 与 Delivery 关联 |
| `WSH_NEW_DELIVERIES` | 交货/发运 | `DELIVERY_ID`, `STATUS_CODE`, Initial Pick-up/Ultimate Drop-off |
| `MTL_RESERVATIONS` | 库存保留 | Demand Source+Item+Org+Supply |
| `MTL_MATERIAL_TRANSACTIONS` | 销售出库交易 | OM Line 通过 Source Line/Transaction Source 关联 |
| `RA_INTERFACE_LINES_ALL` | AutoInvoice 接口 | OM Line ID 通过 Transaction Flexfield Attributes 传递 |
| `RA_CUSTOMER_TRX_LINES_ALL` | AR 发票行 | 保留 Interface Line Context/Attributes 用于 Drilldown |
| `AR_RECEIVABLE_APPLICATIONS_ALL` | AR 收款核销 | Receipt 到 Invoice/Payment Schedule 关联 |

### OM/Shipping 常用状态

- `OE_ORDER_HEADERS_ALL.FLOW_STATUS_CODE` 常见 Entered、Booked、Closed、Cancelled 等头状态。
- `OE_ORDER_LINES_ALL.FLOW_STATUS_CODE` 更细，可包括 Entered、Awaiting Shipping、Picked、Shipped、Interfaced、Closed、Cancelled。实际 Workflow Activity Status 应在 Workflow Status Monitor/WF 表中验证。
- `WSH_DELIVERY_DETAILS.RELEASED_STATUS` 是单字母 Shipping Lookup，常见业务含义有 Ready to Release、Released to Warehouse、Staged/Pick Confirmed、Shipped、Backordered、Cancelled。必须关联 Shipping Lookup，不建议在定制中只写不完整 `DECODE`。

## Projects → Assets 表链

| 表 | 中文名 | 关键关联 |
| --- | --- | --- |
| `PA_PROJECTS_ALL` | 项目 | `PROJECT_ID`, `SEGMENT1`, `ORG_ID` |
| `PA_TASKS` | 项目任务 | `TASK_ID`, `PROJECT_ID`, `TOP_TASK_ID` |
| `PA_EXPENDITURE_ITEMS_ALL` | 项目支出项 | `EXPENDITURE_ITEM_ID`, `PROJECT_ID`, `TASK_ID`, Cost Distribution Flags |
| `PA_PROJECT_ASSETS_ALL` | 项目资产 | `PROJECT_ASSET_ID`, `PROJECT_ID`, FA Book/Category/Asset 关联 |
| `PA_PROJECT_ASSET_LINES_ALL` | 项目资产行 | 归集到资产的项目成本 |
| `FA_MASS_ADDITIONS` | FA 批量增加 | `PROJECT_ID`, `TASK_ID`, `POSTING_STATUS` |

## 跨模块跟踪原则

1. 首先记录源单据的内部 ID 和组织，再查下游外键/接口 Attribute。
2. 一对多链路应分层汇总，避免把 PO→Receipt→Invoice 直接多对多 Join 后重复计数/金额。
3. 反冲/更正/退货不会删除原交易；应沿 Parent/Reversal ID 链计算净额。
4. 业务完成、子账会计、SLA Final、Transfer GL、Journal Import、Post 是独立状态。

## 官方参考

- [Oracle E-Business Suite R12.2 Documentation Library](https://docs.oracle.com/cd/E26401_01/index.htm)
- [Oracle E-Business Suite eTRM User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/)
