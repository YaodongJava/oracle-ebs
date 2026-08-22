# Credit to Cash

## 范围与目标
覆盖 TCA 客户、订单/发运边界、应收交易、信用、催收、自助、扣款/争议、Loans 和 C2C 内控。

## 运行与实施控制
以 Party/Account/Site 分层治理客户；定义交易来源/类型、AutoAccounting、收款方法、信用策略、催收和坏账审批；以交易、收款、会计和 GL 完成对账。

## 核心数据对象
HZ_PARTIES、HZ_CUST_ACCOUNTS、RA_CUSTOMER_TRX_ALL、RA_CUSTOMER_TRX_LINES_ALL、AR_PAYMENT_SCHEDULES_ALL、AR_CASH_RECEIPTS_ALL、AR_RECEIVABLE_APPLICATIONS_ALL、XLA_AE_HEADERS。对象、列、状态和 API 签名须在目标实例 eTRM、Integration Repository 与数据字典复核。

## 与既有知识的关系
本目标目录新增详细入口；已有专题保留在 [03-ar/README](../03-ar/README.md) 并逐步迁移链接，不复制历史内容。

## 官方依据
[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
