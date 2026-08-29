# Oracle E-Business Suite R12.2.x 财务业务与技术知识库

本知识库面向 Oracle EBS 财务功能顾问、技术顾问、集成顾问、实施经理、测试和生产运维人员。资料按知识域整理为“一个模块一个文件”，避免同一模块在多个目录和页面之间来回查找。

> 默认适用于 Oracle E-Business Suite R12.2.x。产品能力、对象、字段、API、状态和处理行为必须结合目标实例的产品安装、许可证、本地化、EBS/AD-TXK/数据库补丁及客户定制验证。

## 快速开始

- [文档总导航](docs/README.md)
- [导航、学习路径与顾问手册](docs/00-guide.md)
- [财务公共基础](docs/01-foundation.md)
- [记录到报告（R2R）](docs/02-record-to-report.md)
- [采购到付款（P2P）](docs/03-procure-to-pay.md)
- [信用到收款（C2C）](docs/04-credit-to-cash.md)
- [技术架构、开发与集成](docs/10-technical.md)
- [中英文术语与统一参考](docs/90-reference.md)
- [模块数据字典与名词解释](docs/data-dictionary.md)

## 模块文件

| 编号 | 模块文件 | 主要内容 |
| --- | --- | --- |
| 00 | [导航、学习与知识库治理](docs/00-guide.md) | 顾问学习手册、产品地图、角色/生命周期路径、版本、规范、安全和官方资料 |
| 01 | [财务公共基础](docs/01-foundation.md) | 企业结构、法人、Ledger、COA、期间、MOAC、TCA、银行、安全和共享设置 |
| 02 | [记录到报告](docs/02-record-to-report.md) | GL、SLA、FAH、AGIS、预算、重估、合并、财务报告和关账 |
| 03 | [采购到付款](docs/03-procure-to-pay.md) | 供应商、采购、接收、AP、IBY、费用报销、接口、会计和对账 |
| 04 | [信用到收款](docs/04-credit-to-cash.md) | 客户/TCA、AR、收款、信用、催收、争议、接口和会计 |
| 05 | [资产、项目与资本化](docs/05-assets-projects.md) | FA、Projects、项目成本/开票、项目转资产和相关扩展产品 |
| 06 | [现金、资金与税务](docs/06-cash-tax.md) | CE、银行接口/对账、Cash Position、Treasury 和 E-Business Tax |
| 07 | [供应链财务与成本](docs/07-cost-accounting.md) | PO/RCV、INV、WIP、CST、OPM、LCM、COGS 与 SLA/GL |
| 08 | [报表、关账与治理](docs/08-reporting-governance.md) | FSG、BI Publisher、Web ADI、ECC、内控、审计和本地化 |
| 09 | [端到端业务流程](docs/09-end-to-end.md) | R2R、P2P、O2C、资产、项目、成本、税务、公司间和外部子账 |
| 10 | [技术架构、开发与集成](docs/10-technical.md) | R12.2 架构/文件系统/中间件、数据库、接口、Concurrent、Workflow、OAF/Forms、报表、Java、ADOP/EBR、安全、高可用和排错 |
| 11 | [实施与运维生命周期](docs/11-implementation-operations.md) | 评估、蓝图、配置、迁移、测试、切换、Hypercare、补丁、克隆和灾备 |
| 90 | [统一参考资料](docs/90-reference.md) | 术语、表、API、并发程序、Profile、Lookup、报表、状态和错误索引 |
| 99 | [历史归档](docs/99-archive.md) | 已迁移、已废弃或仅供历史追溯的资料 |

## 推荐阅读顺序

```text
产品地图与企业结构
  → 主修业务模块
  → SLA 与 GL
  → 端到端流程、报表、对账和关账
  → 数据模型、接口、并发与排错
  → 实施、测试、切换和生产运维
```

功能顾问可从 [顾问学习手册](docs/00-guide.md#src-docs-00-guide-consultant-handbook) 开始；技术顾问可在掌握业务主键和会计链后进入 [技术模块](docs/10-technical.md)。

## 内容组织说明

- 每个模块仅维护一个 Markdown 文件，并按“学习目标 → 业务/技术全景 → 配置与控制 → 数据、会计和接口 → 页面/UML 实操 → 排错与练习 → 专题详解”组织。
- 已合并的高价值专题保留 `<!-- source: 原路径 -->` 标记；已删除的重复模板保留不可见兼容锚点，避免旧链接失效。
- 通用概念只在所属模块维护；跨模块内容通过链接引用。
- SQL 默认为只读诊断样例，使用绑定变量，并限制组织、账簿、期间、日期或主键范围。
- 写入应使用标准页面、公开 API、Open Interface、标准并发程序或经批准的 Oracle Support 方案；不得直接 DML EBS 业务基表。

## 官方资料

- [Oracle E-Business Suite Release 12.2 Documentation Library](https://docs.oracle.com/cd/E26401_01/index.htm)
- [Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
- [Oracle E-Business Suite Technology Documentation](https://docs.oracle.com/cd/E26401_01/nav/technology.htm)
