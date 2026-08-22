# Procure to Pay

## 范围与目标
覆盖供应商、采购/收货、应付发票、Payments、iExpenses、供应商协同和 P2P 内控，强调 PO/Receipt/Invoice/Payment/CE/GL 的闭环。

## 运行与实施控制
确认供应商与地点、采购匹配、容差/Hold、税务、付款方式、银行账户、审批和会计期间；每批接口必须有业务唯一键、控制总额和重跑策略。

## 核心数据对象
PO_HEADERS_ALL、PO_LINES_ALL、RCV_TRANSACTIONS、AP_INVOICES_ALL、AP_INVOICE_LINES_ALL、AP_INVOICE_DISTRIBUTIONS_ALL、AP_PAYMENT_SCHEDULES_ALL、AP_CHECKS_ALL、XLA_AE_HEADERS。对象、列、状态和 API 签名须在目标实例 eTRM、Integration Repository 与数据字典复核。

## 与既有知识的关系
本目标目录新增详细入口；已有专题保留在 [02-ap/README](../02-ap/README.md) 并逐步迁移链接，不复制历史内容。

## 官方依据
[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
