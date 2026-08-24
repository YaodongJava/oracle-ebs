# 按实施生命周期阅读路径

本页将知识库映射到 Oracle EBS 财务实施、扩展、升级和重大变更的阶段。详细交付方法见 [实施与运维生命周期](../11-implementation-operations/README.md)。

## 生命周期总览

```text
Assessment → Blueprint → Build → Test → Cutover → Hypercare → BAU
    ↑             每个阶段都维护需求追踪、风险、决策和证据             ↓
    └──────────────── 持续改进与下一轮变更 ────────────────────────┘
```

## 1. Assessment（评估）

### 先读

- [范围、版本与适用性](scope-and-version.md)
- [财务产品地图与边界](financials-product-map.md)
- [评估、范围与许可证](../11-implementation-operations/assessment-scope-license/README.md)

### 回答

- 当前 EBS/AD-TXK/数据库/中间件版本和支持状态是什么？
- 哪些产品已安装、已配置并有许可？哪些只是数据对象存在？
- 企业组织、Ledger、COA、期间、币种、交易量和月结窗口如何？
- 现有 CEMLI、接口、报表、数据质量、性能和控制风险是什么？
- 项目范围、排除项、依赖、假设、成功指标和负责人是什么？

### 退出证据

版本/产品/许可证清单、现状流程、CEMLI 清单、数据量和质量评估、风险登记、范围与高阶计划签字。

## 2. Blueprint（解决方案蓝图）

### 先读

- [顾问学习手册](consultant-handbook.md)
- [财务公共基础](../01-common/README.md)
- [端到端流程](../08-e2e/README.md)
- [解决方案蓝图](../11-implementation-operations/solution-blueprint/README.md)

### 设计顺序

1. 企业结构、法律实体、Ledger、OU、库存组织和数据访问。
2. COA、日历、币种、汇率、SLA、公司间和报告结构。
3. P2P、C2C、R2R、A2R、Cost、Cash/Tax 的正常和例外流程。
4. 主数据、审批、职责分离、银行/税务和审计控制。
5. 接口、报表、数据迁移、CEMLI、性能和支持模型。

### 退出证据

蓝图、L1～L3 流程、会计事件矩阵、角色/权限矩阵、接口与报表目录、数据迁移策略、决策日志和业务/财务/技术签字。

## 3. Build（配置与构建）

### 先读

- [配置顺序与工作簿](../11-implementation-operations/setup-sequence-and-workbooks/README.md)
- [技术架构与开发](../10-technical/README.md)
- 各产品的 `interfaces.md` 和表/流程文档

### 关键活动

- 按依赖顺序配置，维护配置工作簿和环境差异。
- 对 CEMLI 编写功能/技术设计、代码评审、单元测试和部署/回退工件。
- 接口实现业务键、状态、幂等、重试、补偿、对账、监控和安全。
- 所有 R12.2 数据库/文件工件满足 EBR 和 ADOP 要求。
- 迁移模板包含来源、清洗、映射、验证和控制总额。

### 退出证据

版本化配置和工件、单元测试、代码/设计评审、对象清单、迁移包、运行手册初稿和需求追踪更新。

## 4. Test（测试）

### 先读

- [测试策略](../11-implementation-operations/testing-strategy/README.md)
- [按角色路径中的测试要求](reading-paths-by-role.md#测试人员路径)
- 各模块关账、接口排错和端到端文档

### 测试层次

| 类型 | 核心验收 |
| --- | --- |
| CRP | 目标流程、组织、会计和职责设计可行 |
| SIT | 跨模块/跨系统状态、异常、重试、性能和对账闭环 |
| UAT | 业务用户确认操作、控制、会计、报表和关账 |
| Regression | 补丁/变更未破坏关键现有流程 |
| Performance/Security | 峰值窗口、并发、最小权限、敏感数据和审计达标 |

每个财务用例都要检查业务状态、数量、金额/币种、税、SLA、GL、报表、反向交易和审计证据。

### 退出证据

需求到测试的追踪、结果和日志、缺陷决定、性能容量报告、安全评审、业务/财务/技术验收签字。

## 5. Cutover（切换）

### 先读

- [数据迁移与转换](../11-implementation-operations/data-migration-and-conversion/README.md)
- [切换与回退](../11-implementation-operations/cutover-and-rollback/README.md)
- [生产安全边界](safety-and-production-boundaries.md)

### 关键控制

- 多轮 Mock Conversion 已证明质量、顺序和窗口。
- Freeze、Delta、负责人、依赖、控制总额和不可逆点明确。
- 技术回退与业务补偿分别设计并经过演练。
- 余额、未结 AP/AR、库存、资产、项目、银行和 GL 对账口径获批准。
- Go/No-Go 依据实际证据，不以“计划完成百分比”代替质量判断。

### 退出证据

执行完成的 Cutover Runbook、请求/日志、前后对账、问题与决定、Go-Live 签字和 Hypercare 交接。

## 6. Hypercare（上线强化支持）

### 先读

- [Hypercare 与支持移交](../11-implementation-operations/hypercare-and-support-transition/README.md)
- [监控与诊断](../11-implementation-operations/monitoring-and-diagnostics/README.md)
- [事件、问题与变更](../11-implementation-operations/incident-problem-change/README.md)

### 日常节奏

- 监控接口、并发、Workflow、会计、银行/税务回执和关键服务。
- 按日对账数量、金额、子账/GL 和未结异常。
- 对缺陷分级，区分恢复、根因、永久修复和临时规避。
- 更新已知错误、操作手册、培训和支持升级路径。

### 退出证据

稳定性 KPI 达标、重大缺陷关闭、无未解释重大对账差异、BAU 团队独立运行、知识与责任正式移交。

## 7. BAU Operations（常态运维）

### 先读

- [期间关账运维](../11-implementation-operations/period-close-operations/README.md)
- [补丁与升级](../11-implementation-operations/patching-and-upgrade/README.md)
- [克隆与刷新](../11-implementation-operations/cloning-and-refresh/README.md)
- [备份恢复与灾备](../11-implementation-operations/backup-recovery-dr/README.md)
- [归档清理与容量](../11-implementation-operations/archive-purge-capacity/README.md)

### 持续控制

- 日/周/月批处理与关账清单、对账和签字。
- 接口、并发、Workflow、证书、容量和数据增长监控。
- Incident/Problem/Change、RCA、补丁演练和财务回归。
- 克隆隔离、数据脱敏、备份恢复和 DR 演练。
- CEMLI、配置、文档和支持知识随变更同步更新。

## 各阶段共同禁止项

- 未确认版本和许可证便承诺产品能力。
- 用直接 DML 业务基表代替标准流程或受支持接口。
- 只测试页面成功，不验证 SLA、GL、报表、对账和反向交易。
- 只写技术回退，不考虑已发付款/发票/会计的业务补偿。
- 上线后用长期手工调整掩盖流程、配置或接口设计缺陷。
