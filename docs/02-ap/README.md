# Oracle Payables（AP）

本目录覆盖 P2P 链路中的供应商主数据、采购/收货匹配、应付发票、付款、子账会计、对账与关账。付款执行由 Oracle Payments（IBY）承担；银行对账由 Cash Management（CE）承担，因此必须以端到端相关号和控制批次追溯，而不是仅依赖单张 AP 发票状态。

## 专题导航

- [流程与控制点](process.md)
- [供应商与地点](suppliers.md)
- [发票、验证、匹配与税务](invoices.md)
- [付款与核销](payments.md)
- [Oracle Payments 与 Internet Expenses](payments-iby-expenses.md)
- [预付款、员工报销、借贷项和保留款](special-transactions.md)
- [会计、月结与报表](accounting-close-reports.md)
- [表结构](tables.md)
- [标准 Open Interface 实现](interfaces.md)
- [接口排错](interfaces-troubleshooting.md)

## 必须形成闭环的控制

| 控制目标 | EBS 关键点 | 验证证据 |
| --- | --- | --- |
| 防重复付款 | Supplier Site、Invoice Number、Duplicate Invoice 控制、付款状态 | 重复发票例外清单、付款批次复核 |
| 采购合规 | 2/3/4-way Match、Tolerance、Hold、Receipt Accrual | PO/Receipt/Invoice 三单匹配差异及 Hold 处理记录 |
| 付款安全 | Payment Method、Payment Process Profile、职责分离、银行回执 | PPR 审批、支付文件、ACK/拒绝和作废/重发记录 |
| 财务完整性 | Validation、Create Accounting、Transfer/Post、AP Trial Balance | AP 负债、付款、XLA 和 GL 的期间对账 |

## 相关产品边界

- 采购、收货和应计归属 Procurement/Receiving；本目录说明 AP 消耗这些结果的方式。
- 税务确定归属 E-Business Tax；本目录仅说明发票场景的使用和诊断入口。
- iExpenses、Corporate Card、iSupplier 和 Payments 是可选/已安装产品依赖项，实施前核对许可证、责任和补丁级别。

## 官方依据

- [Oracle Payables Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
- [Oracle Payments Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
