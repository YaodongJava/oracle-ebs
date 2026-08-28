# Oracle EBS R12.2.x 文档总导航

本目录采用“一个模块一个文件”的结构。每个模块先提供学习目标、业务全景、配置与控制、技术与会计追溯、排错和练习，再进入保留的专题详解；核心模块还提供 Mermaid UML 图、标准页面操作剧本和资深顾问评审清单。专题中的来源标记可追溯到整理前的文件路径。

## 模块导航

| 模块 | 文件 | 内容重点 |
| --- | --- | --- |
| 学习与治理 | [00-guide.md](00-guide.md) | 顾问手册、产品地图、阅读路径、版本、安全和官方资料 |
| 财务公共基础 | [01-foundation.md](01-foundation.md) | 组织、Ledger、COA、期间、MOAC、TCA、银行和安全 |
| R2R | [02-record-to-report.md](02-record-to-report.md) | GL、SLA、FAH、AGIS、预算、合并、报告和关账 |
| P2P | [03-procure-to-pay.md](03-procure-to-pay.md) | Supplier、PO、Receiving、AP、IBY、费用和对账 |
| C2C | [04-credit-to-cash.md](04-credit-to-cash.md) | Customer/TCA、AR、Receipt、Credit、Collections 和对账 |
| 资产与项目 | [05-assets-projects.md](05-assets-projects.md) | FA、Projects、资本化、折旧、开票及项目转资产 |
| 现金与税务 | [06-cash-tax.md](06-cash-tax.md) | CE、银行对账、Cash/Treasury 和 E-Business Tax |
| 成本会计 | [07-cost-accounting.md](07-cost-accounting.md) | Receiving、Inventory、WIP、Costing、LCM 和 COGS |
| 报表与治理 | [08-reporting-governance.md](08-reporting-governance.md) | FSG、BI Publisher、ECC、内控、审计和本地化 |
| 端到端流程 | [09-end-to-end.md](09-end-to-end.md) | 跨模块状态、主键、会计、接口、对账和关账依赖 |
| 技术模块 | [10-technical.md](10-technical.md) | 架构、数据、接口、并发、Workflow、OAF、Forms 和 ADOP |
| 实施与运维 | [11-implementation-operations.md](11-implementation-operations.md) | 蓝图、迁移、测试、切换、运行、补丁、克隆和灾备 |
| 统一参考 | [90-reference.md](90-reference.md) | 中英术语、表、接口、程序、Profile、Lookup、报表和错误 |
| 历史归档 | [99-archive.md](99-archive.md) | 历史追溯和废弃资料 |

## 按角色进入

| 角色 | 建议起点 | 后续路径 |
| --- | --- | --- |
| 财务功能顾问 | [综合顾问学习手册](00-guide.md#src-docs-00-guide-consultant-handbook) | 公共基础 → 主修模块 → SLA/GL → E2E → 报表/关账 |
| R2R 顾问 | [记录到报告](02-record-to-report.md) | Ledger/SLA → Journal/Posting → Revaluation/Consolidation → Close |
| P2P 顾问 | [采购到付款](03-procure-to-pay.md) | Supplier → PO/Receipt → AP Invoice → IBY/CE → SLA/GL |
| C2C 顾问 | [信用到收款](04-credit-to-cash.md) | TCA → AR Transaction → Receipt → Collections → SLA/GL |
| 资产/项目顾问 | [资产与项目](05-assets-projects.md) | 来源成本 → 资本化/资产 → 折旧/开票 → 对账 |
| 成本顾问 | [供应链财务与成本](07-cost-accounting.md) | PO/RCV → INV/WIP/CST → SLA/GL → 期间关闭 |
| 技术/集成顾问 | [技术模块](10-technical.md) | 数据模型 → 标准接口/API → 并发/Workflow → ADOP/运维 |
| 实施经理/运维 | [实施与运维](11-implementation-operations.md) | Assessment → Blueprint → Test → Cutover → Hypercare/BAU |

## 使用边界

- 表、列、API 签名、状态、并发程序参数和菜单以目标实例验证为准。
- 示例会计分录用于理解业务，实际结果以项目 SLA 配置和测试为准。
- 可选产品、本地化和数据库诊断能力需确认许可证及安装范围。
- 生产写入和数据修复必须遵守 [生产安全边界](00-guide.md#src-docs-00-guide-safety-and-production-boundaries)。
