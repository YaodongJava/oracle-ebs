# Oracle EBS 财务实施与运维生命周期

本目录将 Assessment（评估）、Blueprint（蓝图）、Build（构建）、Test（测试）、Cutover（切换）、Hypercare（上线强化支持）和 BAU Operations（常态运维）串成可审计的交付闭环。方法适用于新实施、扩展上线、升级、重大补丁和财务流程重构；具体产品配置与技术实现仍应链接对应模块正文。

## 生命周期与阶段门

| 阶段 | 关键问题 | 最低交付物 | 退出条件 |
| --- | --- | --- | --- |
| 1. Assessment | 为什么做、范围多大、现状风险是什么？ | 现状/版本/产品/许可证/CEMLI/数据量评估，范围和高阶计划 | 范围、假设、依赖、预算和治理获批准 |
| 2. Blueprint | 目标流程、组织、会计和控制如何设计？ | 企业结构、流程、COA、SLA、税务、权限、接口、报表、迁移蓝图 | 关键设计决策与差异正式签字 |
| 3. Build | 如何可重复地配置、开发和迁移？ | 配置工作簿、技术设计、代码、迁移工件、单元测试 | 工件版本化且通过评审/单测 |
| 4. Test | 业务、会计、性能和安全是否可接受？ | CRP、SIT、UAT、回归、性能、安全证据与缺陷决定 | 无阻断缺陷；业务、财务、技术共同签字 |
| 5. Cutover | 如何在受控窗口迁移并判断上线？ | 冻结、增量、Runbook、控制总额、回退/补偿、Go/No-Go | 迁移与对账完成，关键流程可用 |
| 6. Hypercare | 如何稳定运行并收敛缺陷？ | 每日监控/对账、缺陷看板、RCA、知识库和支持排班 | KPI 达标、重大问题关闭、支持团队接收 |
| 7. BAU | 如何持续关账、修补、审计和优化？ | 运行手册、月结、监控、变更、补丁、灾备和归档制度 | 持续运行，无一次性退出点 |

## 专题导航

| 专题 | 中文说明 | 核心产出 |
| --- | --- | --- |
| [assessment-scope-license](assessment-scope-license/README.md) | 评估、范围与许可证 | 产品/版本/CEMLI/数据量清单、范围边界和风险 |
| [solution-blueprint](solution-blueprint/README.md) | 解决方案蓝图 | 企业结构、流程、会计、数据、集成、报表和控制设计 |
| [setup-sequence-and-workbooks](setup-sequence-and-workbooks/README.md) | 配置顺序与工作簿 | 依赖矩阵、配置值、迁移方式、验证和回退 |
| [data-migration-and-conversion](data-migration-and-conversion/README.md) | 数据迁移与转换 | Profile/Cleanse/Map/Load/Reconcile、Mock 和切换批次 |
| [integration-delivery](integration-delivery/README.md) | 集成交付 | 契约、幂等、状态机、安全、容量、监控和对账 |
| [testing-strategy](testing-strategy/README.md) | 测试策略 | CRP/SIT/UAT/回归/性能/安全、需求追踪和会计断言 |
| [cutover-and-rollback](cutover-and-rollback/README.md) | 切换与回退 | Freeze、Delta、Runbook、不可逆点、Go/No-Go 和补偿 |
| [hypercare-and-support-transition](hypercare-and-support-transition/README.md) | 上线支持与移交 | 每日对账、缺陷分级、知识转移、SLA 和接收标准 |
| [period-close-operations](period-close-operations/README.md) | 期间关账运维 | 关账日历、依赖、程序、对账、签字和重开控制 |
| [monitoring-and-diagnostics](monitoring-and-diagnostics/README.md) | 监控与诊断 | 接口/并发/Workflow/会计/服务指标和证据包 |
| [incident-problem-change](incident-problem-change/README.md) | 事件、问题与变更 | 恢复、RCA、已知错误、变更审批和前后验证 |
| [patching-and-upgrade](patching-and-upgrade/README.md) | 补丁与升级 | RUP/AD-TXK/CPU/one-off、冲突、演练、回归和发布 |
| [cloning-and-refresh](cloning-and-refresh/README.md) | 克隆与刷新 | Rapid Clone、环境差异、脱敏、接口隔离和验证 |
| [backup-recovery-dr](backup-recovery-dr/README.md) | 备份、恢复与灾备 | RPO/RTO、RMAN、Data Guard、应用恢复和演练证据 |
| [archive-purge-capacity](archive-purge-capacity/README.md) | 归档、清理与容量 | 保留政策、标准 Purge、增长预测、性能和审计 |

## 贯穿全生命周期的工作流

### 需求到证据

```text
业务需求
  → 方案决策/配置或 CEMLI
  → 设计与风险
  → 构建工件
  → 测试用例与结果
  → 切换步骤与验证
  → 运行控制与监控
```

Requirements Traceability Matrix（RTM，需求追踪矩阵）必须能从需求追溯到配置、开发、测试和上线证据，也能从一个生产工件反向找到批准需求和所有者。

### 数量与金额对账

凡涉及主数据、余额、未结交易、接口、会计或期间，至少按组织、账簿、期间、币种和业务分类验证：

```text
来源控制总额 = 成功 + 拒绝 + 在途
期初 + 增加 - 减少 ± 调整 = 期末
子账余额 + 已批准时间性差异 = GL 控制账户
```

对账结果必须记录数据截止时间、查询/报表版本、差异原因、责任人和解决期限。

### 决策与例外

架构、会计、权限、数据保留、接口补偿和切换不可逆点等重大决策使用 Decision Log（决策日志）记录：背景、选项、权衡、批准人、日期、影响和重新评估条件。任何绕过标准控制的临时方案必须有到期日和退出计划。

## 主要交付物模板

### Solution Blueprint（解决方案蓝图）

- 业务目标、范围、排除项、假设和成功指标。
- 企业结构、COA、日历、币种、Ledger 和数据访问。
- L1～L3 流程、异常/冲销、职责分离和审批。
- 主数据、税务、银行、会计事件/SLA 和报表。
- 接口、数据迁移、CEMLI、容量、安全和审计。
- 关账、运维、支持模型、风险、决策和开放事项。

### Configuration Workbook（配置工作簿）

- 对象、业务目的、导航责任、前置依赖和配置所有者。
- 配置代码/名称、中文含义、默认值依据和环境差异。
- 会计、权限、接口、报表和关账影响。
- 迁移方式、目标环境、验证证据、停用/回退方式。

### Technical Design（技术设计）

- 业务契约、数据模型、状态机、字段映射和错误目录。
- EBS 上下文、受支持入口、事务、幂等、重试与补偿。
- 安全、敏感数据、性能容量、日志、指标、告警和保留。
- 对象/文件清单、依赖、ADOP/EBR、安装/卸载和回退。
- 单元、SIT、性能、安全、会计和对账断言。

### Cutover Runbook（切换运行手册）

每一步应包含：编号、负责人、计划/实际时间、前置条件、命令/导航/程序参数、输入输出、控制总额、成功标准、失败动作、重试/回退边界、证据位置和批准人。

## 测试覆盖矩阵

| 维度 | 最低场景 |
| --- | --- |
| 业务 | 正常、取消、冲销、退回、拒绝、部分成功 |
| 时间 | 当前期、未来期、关闭期、跨期、月末/年末 |
| 组织 | 单 OU、多 OU、跨法人、数据权限拒绝 |
| 金额 | 本位币、外币、汇率缺失/变化、税、舍入、零/负数边界 |
| 集成 | 重复、乱序、超时、重放、外部不可用、回执拒绝 |
| 批量 | 代表性峰值、并发、批次拆分、恢复时间和关账窗口 |
| 会计 | Draft/Final、SLA 错误、未传 GL、导入失败、未过账、冲销 |
| 安全 | 最小权限、SoD、敏感数据、证书/密钥、审计追踪 |

## 上线准备检查

1. 生产版本、补丁、许可证、容量和拓扑已确认。
2. 配置基线、CEMLI、接口端点、证书和调度均完成生产核对。
3. 迁移 Mock 达到质量和窗口要求，最终控制总额已批准。
4. Sev1/Sev2 缺陷关闭；剩余缺陷有业务接受和规避方案。
5. 回退与业务补偿在演练中可执行，不可逆点已标记。
6. 运维监控、告警、批处理、关账和升级路径已就绪。
7. 业务、财务、技术、安全和支持负责人完成 Go/No-Go 签字。

## Hypercare 退出标准

- 关键接口、并发请求和 Workflow 按窗口稳定运行。
- AP/AR/FA/CE/CST/SLA/GL 等关键对账无未解释重大差异。
- 缺陷数量和严重度达到接收阈值，Workaround 有有效期和所有者。
- 操作手册、已知错误、监控阈值、支持排班和供应商升级路径已移交。
- BAU 团队独立完成至少一个代表性日结/月结或批处理周期。

## 生产运行原则

- 标准页面、公开 API、Open Interface、标准并发程序和 Oracle Support 批准方案优先。
- 不用直接 DML 基表、无审批脚本或长期手工 GL 调整掩盖根因。
- 所有变更具备影响分析、测试、回退、窗口、批准和前后验证。
- 克隆环境在连接外部系统前完成端点隔离、计划任务控制和敏感数据脱敏。
- 备份“成功”不等于可恢复；按批准 RPO/RTO 定期演练数据库和应用全链路恢复。

## 相关入口

- [顾问学习手册](../00-guide/consultant-handbook.md)
- [按生命周期阅读路径](../00-guide/reading-paths-by-lifecycle.md)
- [生产安全与支持边界](../00-guide/safety-and-production-boundaries.md)
- [技术架构与开发](../10-technical/README.md)
- [端到端流程](../09-end-to-end/README.md)
