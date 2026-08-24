# 按角色阅读与练习路径

本页把知识库内容转换为可执行的学习路径。所有角色都应先阅读 [顾问学习手册](consultant-handbook.md)、[范围与版本](scope-and-version.md) 和 [生产安全边界](safety-and-production-boundaries.md)，再进入专项。

## 共同基础：先建立端到端认知

| 阶段 | 阅读内容 | 动手练习 | 验收标准 |
| --- | --- | --- | --- |
| 1. 企业结构 | [企业组织](../01-common/organization.md)、[COA](../01-common/coa.md)、[期间币种](../01-common/calendar-currency-period.md) | 画出一个实例的 Ledger、Legal Entity、OU、Inventory Org 关系 | 能解释组织、核算和权限边界 |
| 2. 会计主线 | [SLA](../01-common/sla.md)、[GL 流程](../04-gl/process.md) | 追溯一笔子账交易至 SLA 和 GL | 能区分业务状态、会计状态、传输和过账状态 |
| 3. 端到端流程 | [P2P](../08-e2e/procure-to-pay.md)、[O2C](../08-e2e/order-to-cash.md)、[R2R](../08-e2e/record-to-report-close.md) | 为一个流程标注单据、状态、主键、分录和控制点 | 能说明上游失败如何影响关账 |
| 4. 安全与运维 | [安全](../01-common/security.md)、[技术运维](../09-technical/operations.md) | 用职责、组织、账簿、期间复现一次“查不到数据”问题 | 能先验证上下文，而不是先改数据 |

## 财务功能顾问路径

### 阶段 1：公共财务基础

建议顺序：

1. 企业结构、Legal Entity Configurator（法律实体配置器）、Accounting Setup Manager（会计设置管理器）。
2. Ledger（账簿）、Ledger Set（账簿集）、Secondary Ledger（辅助账簿）、Reporting Currency（报告币种）。
3. Chart of Accounts（科目表）、Key Flexfield（关键弹性域）、会计日历、币种和汇率。
4. MOAC（多组织访问控制）、Data Access Set（数据访问集）、职责和审批。
5. TCA（贸易社区架构）、集中银行账户、EBTax（电子商务税）。
6. SLA（子账会计）、单据序列、报表和审计基础。

成果：企业结构图、COA 设计说明、职责/数据权限矩阵、共享主数据所有权矩阵和会计设计原则。

### 阶段 2：选择主修业务域

| 主修方向 | 推荐阅读顺序 | 必做练习 |
| --- | --- | --- |
| R2R/GL | [GL](../04-gl/README.md) → [SLA/FAH/AGIS](../04-gl/sla-fah-agis.md) → [重估合并](../04-gl/consolidation-revaluation.md) → [关账](../04-gl/close-reports.md) | 手工/接口日记账、过账、冲销、外币重估和期末对账 |
| P2P/AP | [AP](../02-ap/README.md) → [供应商](../02-ap/suppliers.md) → [发票](../02-ap/invoices.md) → [付款](../02-ap/payments.md) → [P2P](../08-e2e/procure-to-pay.md) | PO 匹配发票、Hold、预付款、付款、作废和 AP-GL 对账 |
| C2C/AR | [AR](../03-ar/README.md) → [客户信用](../03-ar/customers-credit.md) → [交易](../03-ar/transactions.md) → [收款](../03-ar/receipts.md) → [O2C](../08-e2e/order-to-cash.md) | AutoInvoice、收款核销/撤销、贷项通知单、账龄和 AR-GL 对账 |
| Assets | [FA](../05-fa/README.md) → [设置](../05-fa/setup.md) → [资产交易](../05-fa/asset-transactions.md) → [折旧](../05-fa/depreciation-accounting.md) | 新增、调整、转移、折旧、报废/恢复和 FA-GL 对账 |
| Cash/Tax | [CE/Tax](../07-ce-tax/README.md) → [现金管理](../07-ce-tax/cash-management.md) → [EBTax](../07-ce-tax/ebtax.md) | 银行对账单、自动对账、税务确定、税额会计和申报对账 |
| Costing | [成本](../06-cost/README.md) → [成本方法](../06-cost/costing-methods.md) → [会计流](../06-cost/accounting-flow.md) | 采购入库、库存移动、WIP 完工、成本处理和库存/WIP-GL 对账 |

### 阶段 3：实施能力

按 [实施与运维生命周期](../11-implementation-operations/README.md) 学习以下交付：

- 现状评估、Fit-to-Standard/Fit-Gap（标准适配/差异分析）和范围边界。
- 配置工作簿、会计事件矩阵、主数据模板和报表清单。
- CRP（会议室演练）、SIT（系统集成测试）、UAT（用户验收测试）和财务回归。
- 余额/未结交易迁移、切换对账、Go/No-Go 和 Hypercare（上线强化支持）。
- 月结运行手册、控制证据、问题升级和知识转移。

功能顾问完成路径的标准：能从一个业务需求推导配置、角色、会计、接口、报表、测试、关账和内控影响；能与技术顾问共同定义可验证的接口与排错证据。

## 财务技术顾问路径

### 阶段 1：理解业务数据而不只记表名

1. 阅读 [模块数据字典](../README.md#模块数据字典) 与至少一个端到端流程。
2. 选择 AP 或 AR 的一笔交易，记录业务主键、`ORG_ID`、Ledger、期间和币种。
3. 追溯业务分配、XLA Event、SLA 分录、GL Import Reference、Journal 和 Balance。
4. 分别观察正常、冲销、拒绝和跨期场景；整理状态字典。

验收标准：能够解释每次连接的业务意义，而不是只给出“能查到数据”的 SQL。

### 阶段 2：EBS 技术栈

| 专题 | 入口 | 实践成果 |
| --- | --- | --- |
| 架构与请求链路 | [技术总览](../10-technical/README.md)、[R12.2 技术运维](../09-technical/README.md) | 一张浏览器到数据库/并发/外部系统的调用链图 |
| 数据模型与 PL/SQL | [数据模型](../09-technical/data-model.md)、[表索引](../09-technical/tables.md) | 只读诊断 SQL、对象依赖与权限说明 |
| 接口与 API | [集成](../09-technical/integration.md)、[接口实现手册](../09-technical/interfaces.md) | 接口契约、状态机、幂等、重试、补偿和对账设计 |
| Concurrent | [并发程序](../09-technical/concurrent-programs.md) | 程序定义、请求提交、日志和队列排错手册 |
| Workflow/AME/OAF | [工作流治理](../09-technical/workflow-ame-oaf-governance.md) | 一条审批链的业务键、活动、通知和错误定位 |
| 定制与迁移 | [定制](../09-technical/customization.md) | CEMLI 工件、依赖、迁移和回退清单 |
| ADOP/EBR | [发布治理](../09-technical/adop-ebr-release.md) | 一次非生产发布记录和 Cutover 冒烟测试 |
| 运维与性能 | [运维](../09-technical/operations.md) | 从请求号到 SQL/日志/业务结果的证据包 |

### 阶段 3：模块接口主修

选择一个接口完成全链路实作：

- AP Invoice Open Interface（AP 发票开放接口）。
- AutoInvoice（AR 自动开票）。
- GL Interface / Journal Import（总账接口/日记账导入）。
- FA Mass Additions（固定资产成批增加）。
- Material Transactions Interface（物料事务接口）。
- Bank Statement Interface（银行对账单接口）。

每个实作必须包含：来源控制总额、暂存表、字段映射、业务验证、标准入口、并发程序、错误分类、幂等重放、业务主键回写、SLA/GL 检查、监控告警和数据保留。

技术顾问完成路径的标准：实现不直接 DML 业务基表；能证明重复调用不产生重复业务；能从外部消息关联到 EBS 业务单据、并发请求、会计和回执；发布符合 R12.2 ADOP/EBR 边界。

## 集成顾问路径

1. 先理解目标业务状态机和最终会计结果。
2. 从 Open Interface、Public API、Integration Repository/ISG、Business Event 中选择受支持入口。
3. 定义业务唯一键、批次/相关号、顺序性、幂等、重试上限和人工补偿。
4. 设计认证、授权、TLS、证书/密钥、敏感数据脱敏和审计。
5. 建立“来源 → 中间件 → EBS 接口 → EBS 业务 → SLA/GL → 回执”的数量与金额对账。
6. 以正常、重复、乱序、超时、部分成功、跨期和下游不可用场景完成 SIT。

推荐入口：[通用接口实现](../01-common/interfaces.md)、[跨模块接口](../08-e2e/interfaces.md)、[技术接口治理](../09-technical/interfaces.md)。

## 实施经理/方案负责人路径

重点不是记忆产品表，而是管理边界、依赖与证据：

- 用 [财务产品地图](financials-product-map.md) 确定产品、许可证、负责人和相邻系统。
- 用 [实施生命周期](reading-paths-by-lifecycle.md) 规划阶段门、交付物和签字。
- 评审企业结构、COA、会计、数据迁移、接口、报表、安全和关账方案之间的依赖。
- 管理 CEMLI（Configurations, Extensions, Modifications, Localizations, Integrations，配置、扩展、修改、本地化和集成）清单与标准能力替代计划。
- 在切换前确认控制总额、不可逆点、业务补偿、性能容量和支持移交。

## 测试人员路径

测试脚本不能只写页面点击步骤。每个用例应包含：前置配置、角色/组织、输入数据、预期业务状态、预期会计、预期报表、接口/请求证据、反向交易和清理方式。

优先场景：正常、取消/冲销、拒绝、重复、跨期、外币、多组织、税差异、舍入、批量、并发冲突、权限拒绝和关账期间。

## 生产运维人员路径

推荐顺序：[生产安全边界](safety-and-production-boundaries.md) → [技术运维](../09-technical/operations.md) → [接口排错](../09-technical/interfaces.md) → 各模块 `interfaces-troubleshooting.md`/关账文档。

运维人员应能：

- 从业务单据、接口批次或请求号定位日志和状态。
- 区分可重试技术失败、需修正业务拒绝和需 Oracle Support 的产品缺陷。
- 在恢复前判断步骤是否可能重复付款、重复开票或重复记账。
- 使用标准页面、公开 API、Open Interface、标准并发程序或批准的 Support 方案修复。
- 为事件、问题和变更保存前后对账、审批和审计证据。

## 建议的 12 周共同学习计划

| 周 | 主题 | 交付 |
| --- | --- | --- |
| 1 | 产品地图、企业结构、职责 | 实例结构图与术语卡片 |
| 2 | COA、日历、币种、期间 | 核算基础配置说明 |
| 3 | SLA 与 GL | 子账到 GL 追溯记录 |
| 4 | P2P/AP/IBY | 发票到付款流程与分录 |
| 5 | O2C/AR/CE | 开票到银行对账流程与分录 |
| 6 | FA/Projects | 资本化、折旧和资产对账 |
| 7 | INV/WIP/Cost | 成本处理状态和库存/WIP 对账 |
| 8 | EBTax、公司间和报表 | 税务/公司间案例与报告解释 |
| 9 | 数据模型、SQL、Concurrent | 只读诊断与请求排错 |
| 10 | 接口、Workflow、AME | 一份可重放接口设计与审批排错 |
| 11 | ADOP/EBR、性能、安全 | 非生产发布和冒烟测试证据 |
| 12 | 迁移、切换、关账、支持 | 端到端演练与交付评审 |

## 学习纪律

- 所有表、列、API 签名、状态值和并发程序参数均以目标实例验证为准。
- SQL 是诊断证据，不是绕过产品校验的数据修复工具。
- 示例会计分录是理解工具，不代表项目实际账户和规则。
- 可选产品、国家本地化和数据库诊断包必须确认许可证与安装范围。
- 每次结论都应能回到业务事实、配置依据、运行日志或官方资料。
