# 库存、WIP 与成本（INV / WIP / CST）

本目录覆盖收货、库存事务、WIP、成本计算和销售成本向 SLA/GL 的会计链。库存组织与成本方法属于高风险基础配置，任何成本重算、期间关闭或接口重传均应先界定组织、期间、物料和事务范围。

## 专题导航

- [收货、库存、WIP 与销售成本会计流](accounting-flow.md)
- [成本组织、成本类型与成本组设置](setup.md)
- [标准、平均与周期成本](costing-methods.md)
- [物料、资源与间接费](cost-elements.md)
- [成本结转、关期与报表](period-close-reports.md)
- [高级成本控制与差异](advanced-costing-controls.md)
- [表结构](tables.md)
- [Transaction Open Interface 实现](interfaces.md)
- [处理器和接口排错](interfaces-troubleshooting.md)

## 必须控制的业务事件

- 收货应计、发票价格/汇率差异、库存接收与 AP 负债须按采购、收货和发票三条链对账。
- 每笔物料事务必须可追溯到事务类型、来源类型、成本组织、成本期间和会计事件；库存余额不能仅用应用页面的当前数量替代会计分析。
- 标准成本更新、平均成本调整、WIP 完工/关闭和 COGS Recognition 需要在关期清单中设定顺序、冻结窗口和异常报告。
- 接口使用 `MTL_TRANSACTIONS_INTERFACE` 等标准入口，须设业务唯一键、批次控制、Lot/Serial 校验和失败行隔离。

## 官方依据

- [Oracle Supply Chain Management Documentation](https://docs.oracle.com/cd/E26401_01/nav/scm.htm)
