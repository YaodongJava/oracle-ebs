# 财务公共基础

本目录是所有财务与供应链子账的共同前提。先完成企业结构、账簿、会计科目、期间、MOAC 和职责权限的设计，再配置 AP、AR、GL、FA 或成本模块；后续改变这些基础对象通常需要跨模块影响评估和完整回归。

## 阅读与实施顺序

1. [企业结构与多组织](organization.md)：Business Group、Legal Entity、Ledger、OU、Inventory Organization 与 MOAC。
2. [会计科目](coa.md)：COA、段限定符、值集、交叉验证和安全规则。
3. [日历、币种与期间](calendar-currency-period.md)：会计日历、汇率与期间状态。
4. [职责与数据安全](security.md)：Responsibility、Data Access Set、MO: Security Profile 和请求组。
5. [SLA](sla.md)：所有子账会计事件进入 GL 前的权威会计规则入口。
6. [附件、DFF 与序列](attachments-dff.md)：合规凭证、扩展字段和编号控制。
7. [表结构](tables.md) 与 [接口基础](interfaces.md)：只读诊断、会话上下文和集成治理。

## 关键设计决策

| 决策 | 应在蓝图阶段确认 | 典型风险 |
| --- | --- | --- |
| Ledger 边界 | 币种、会计日历、COA、会计方法、法定/管理报告 | 将仅需 OU 隔离的场景错误拆为多个 Ledger，增加合并和对账成本 |
| Legal Entity 边界 | 法定责任、注册、税务和银行账户所有权 | 把经营组织当成法人，造成税务、公司间和付款主体混乱 |
| OU/MOAC | 交易处理责任、共享服务访问范围、默认 OU | 使用全局安全配置掩盖职责隔离，或自定义 SQL 漏加 `ORG_ID` |
| COA | Balancing、Cost Center、Natural Account 等限定符和治理流程 | 上线后改段结构、改限定符或删除值，导致历史数据和报表不可比 |
| SLA | 法规/管理会计差异、辅助参考、过账粒度、审计追溯 | 直接修改子账或 GL 数据替代规则设计，破坏 Drilldown 和审计链 |

## R12.2 适用边界

- 页面配置、Profile、职责和标准设置优先通过应用界面或 FNDLOAD 迁移；不可直接更新 FND、HR、XLA、GL 等业务基表。
- 只读 SQL 必须在目标实例用 `ALL_TAB_COLUMNS` 和 eTRM 复核。多组织对象优先限定 `ORG_ID`，账簿对象限定 `LEDGER_ID`。
- Legal Entity Configurator、Accounting Setup Manager、MOAC 和 Data Access Set 的设计应保留工作簿、审批记录和回归证据。

## 官方依据

- [Oracle Financials Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48783/toc.htm)
- [Oracle E-Business Suite Security Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22952/toc.htm)
