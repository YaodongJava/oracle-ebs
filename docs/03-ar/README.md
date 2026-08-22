# Oracle Receivables（AR / Credit to Cash）

本目录覆盖客户主数据、应收交易、收款与核销、信用/催收、AutoInvoice/Lockbox、子账会计和关账。订单管理、Shipping 与税务并不属于 AR 的独立账簿边界，但会决定交易来源、开票时点和收入/税务结果。

## 专题导航

- [O2C 子账流程](process.md)
- [客户、地点与信用](customers-credit.md)
- [交易、贷项通知单与调整](transactions.md)
- [收款、Lockbox 与退款](receipts.md)
- [催收、账龄与核销](collections-aging.md)
- [信用、催收、争议与客户自助](credit-collections-ireceivables.md)
- [会计、月结与报表](accounting-close-reports.md)
- [表结构](tables.md)
- [AutoInvoice/Lockbox 实现](interfaces.md)
- [接口排错](interfaces-troubleshooting.md)

## 核心业务链

```text
TCA Party / Account / Site
  → Transaction Source / Type / Line
  → Complete → Create Accounting → Transfer/Post
  → Payment Schedule（未结余额）
  → Receipt → Application / Unapplication / Reversal
  → CE 对账 → Aging、AR Trial Balance、GL 对账
```

## 关键控制

- 客户、客户账户、地点和付款地点必须分层治理；不要仅以显示名称作为唯一业务键。
- AutoInvoice 必须具备来源系统业务键、接口批次、幂等策略与拒绝行回写；重传前先确认原交易是否已成功创建。
- 收款应用、核销、退款、调整、坏账和催收策略都可能影响账龄和会计，需保留批准与对账证据。
- 交易、收款、SLA 与 GL 的断点应按 `CUSTOMER_TRX_ID`、`CASH_RECEIPT_ID`、`EVENT_ID` 和 `GL_SL_LINK_ID` 分层定位。

## 官方依据

- [Oracle Receivables Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
- [Oracle Advanced Collections Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
