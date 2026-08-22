# 端到端财务流程

本目录不复制各模块的设置或表结构，而是维护跨模块业务状态、业务键、接口点、会计事件、关账依赖和对账责任。每一条 E2E 链必须定义来源系统业务键、EBS 主键、批次号、重试/补偿策略和最终财务签字口径。

## 专题导航

- [采购到付款（P2P）](procure-to-pay.md)
- [订单到收款（O2C）](order-to-cash.md)
- [库存、WIP、成本到 GL](inventory-wip-cost-gl.md)
- [项目、费用与资产资本化](projects-assets.md)
- [Record to Report 关账编排](record-to-report-close.md)
- [跨模块表链](tables.md)
- [端到端接口编排案例](interfaces.md)

## 标准追溯方法

```text
来源业务键 / 外部相关号
  → EBS 接口批次与接口行
  → 业务单据主键
  → SLA TRANSACTION_ENTITY / EVENT / AE_HEADER
  → GL_IMPORT_REFERENCES / GL_JE_LINES
  → 银行、报表、对账和签字证据
```

## 设计原则

- 采用补偿和状态推进，而非试图跨 EBS、银行和外部系统执行分布式回滚。
- 以可重放、幂等、可审计的批次为最小运维单元；每次重跑均需先判定原批次的成功/部分成功/失败状态。
- 月结控制按依赖而非组织习惯排期：业务模块完成、子账会计完成、GL 过账完成后才允许形成最终报表和余额签字。

## 官方依据

- [Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
- [Oracle Projects Documentation](https://docs.oracle.com/cd/E26401_01/nav/projects.htm)
