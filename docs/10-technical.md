# 技术架构、开发与集成

> R12.2 架构、数据模型、接口、并发、Workflow、OAF、Forms、ADOP 与运维。本文件由原目录中的 29 份资料合并而成；各章节保留原来源标记，便于审计与后续去重。

## 本模块章节导航

- [Oracle EBS R12.2.x 技术架构、开发与集成](#src-docs-10-technical-readme)（原 `docs/10-technical/README.md`）
- [R12.2 技术专题：adop-and-ebr](#src-docs-10-technical-adop-and-ebr-readme)（原 `docs/10-technical/adop-and-ebr/README.md`）
- [R12.2 技术专题：architecture](#src-docs-10-technical-architecture-readme)（原 `docs/10-technical/architecture/README.md`）
- [R12.2 技术专题：audit-and-compliance](#src-docs-10-technical-audit-and-compliance-readme)（原 `docs/10-technical/audit-and-compliance/README.md`）
- [R12.2 技术专题：concurrent-processing](#src-docs-10-technical-concurrent-processing-readme)（原 `docs/10-technical/concurrent-processing/README.md`）
- [R12.2 技术专题：configuration-migration](#src-docs-10-technical-configuration-migration-readme)（原 `docs/10-technical/configuration-migration/README.md`）
- [R12.2 技术专题：data-model](#src-docs-10-technical-data-model-readme)（原 `docs/10-technical/data-model/README.md`）
- [R12.2 技术专题：database-development](#src-docs-10-technical-database-development-readme)（原 `docs/10-technical/database-development/README.md`）
- [R12.2 技术专题：database-platform](#src-docs-10-technical-database-platform-readme)（原 `docs/10-technical/database-platform/README.md`）
- [R12.2 技术专题：forms](#src-docs-10-technical-forms-readme)（原 `docs/10-technical/forms/README.md`）
- [R12.2 技术专题：integration](#src-docs-10-technical-integration-readme)（原 `docs/10-technical/integration/README.md`）
- [R12.2 技术专题：middleware](#src-docs-10-technical-middleware-readme)（原 `docs/10-technical/middleware/README.md`）
- [R12.2 技术专题：oaf](#src-docs-10-technical-oaf-readme)（原 `docs/10-technical/oaf/README.md`）
- [R12.2 技术专题：performance](#src-docs-10-technical-performance-readme)（原 `docs/10-technical/performance/README.md`）
- [R12.2 技术专题：release-engineering](#src-docs-10-technical-release-engineering-readme)（原 `docs/10-technical/release-engineering/README.md`）
- [R12.2 技术专题：reporting-development](#src-docs-10-technical-reporting-development-readme)（原 `docs/10-technical/reporting-development/README.md`）
- [R12.2 技术专题：security](#src-docs-10-technical-security-readme)（原 `docs/10-technical/security/README.md`）
- [R12.2 技术专题：testing-and-automation](#src-docs-10-technical-testing-and-automation-readme)（原 `docs/10-technical/testing-and-automation/README.md`）
- [R12.2 技术专题：workflow-and-ame](#src-docs-10-technical-workflow-and-ame-readme)（原 `docs/10-technical/workflow-and-ame/README.md`）
- [EBS R12.2 技术、集成与运维](#src-docs-09-technical-readme)（原 `docs/09-technical/README.md`）
- [R12.2 Online Patching、EBR 与发布治理](#src-docs-09-technical-adop-ebr-release)（原 `docs/09-technical/adop-ebr-release.md`）
- [Concurrent Program、请求集与日志排错](#src-docs-09-technical-concurrent-programs)（原 `docs/09-technical/concurrent-programs.md`）
- [PL/SQL、Forms、Personalization 与 OAF 定制](#src-docs-09-technical-customization)（原 `docs/09-technical/customization.md`）
- [EBS R12.2 数据模型与常用表](#src-docs-09-technical-data-model)（原 `docs/09-technical/data-model.md`）
- [开放接口、API、报表与数据迁移](#src-docs-09-technical-integration)（原 `docs/09-technical/integration.md`）
- [Oracle EBS 技术接口实现手册](#src-docs-09-technical-interfaces)（原 `docs/09-technical/interfaces.md`）
- [性能调优、权限审计与 R12.2 生产运维](#src-docs-09-technical-operations)（原 `docs/09-technical/operations.md`）
- [FND、Concurrent、Workflow 与运维常用表结构](#src-docs-09-technical-tables)（原 `docs/09-technical/tables.md`）
- [Workflow、AME、OAF/Forms 与配置迁移治理](#src-docs-09-technical-workflow-ame-oaf-governance)（原 `docs/09-technical/workflow-ame-oaf-governance.md`）

---

<!-- source: docs/10-technical/README.md -->
<a id="src-docs-10-technical-readme"></a>
## Oracle EBS R12.2.x 技术架构、开发与集成


本目录面向财务技术顾问、集成顾问、应用 DBA、测试和生产运维人员。它维护跨产品的技术原则；AP、AR、GL、FA、CE、CST 等具体接口表、业务规则和会计验证仍以对应产品目录为权威入口。

<a id="src-docs-10-technical-readme--技术认知地图"></a>
### 技术认知地图

```text
Browser / Forms Client / External System
          │
Load Balancer / OHS / TLS
          │
WebLogic ─ OAF / Forms / ISG / Workflow
          │
Concurrent Processing / OPP / File Integration
          │
Oracle Database ─ APPS / Product Schemas / EBR
          │
Business Tables → SLA/XLA → GL
```

<a id="src-docs-10-technical-readme--专题导航"></a>
### 专题导航

| 专题 | 中文说明 | 应掌握内容 | 现有详文 |
| --- | --- | --- | --- |
| [architecture](#src-docs-10-technical-architecture-readme) | 总体架构 | 请求链路、节点、OHS、WebLogic、Forms、Concurrent、数据库 | [旧版技术总览](#src-docs-09-technical-readme) |
| [data-model](#src-docs-10-technical-data-model-readme) | 数据模型 | APPS/产品 Schema、同义词、EBR View、WHO、共享模型 | [数据模型详文](#src-docs-09-technical-data-model) |
| [database-development](#src-docs-10-technical-database-development-readme) | 数据库开发 | PL/SQL、Context、FND API、事务、锁、批处理和日志 | [定制开发](#src-docs-09-technical-customization) |
| [integration](#src-docs-10-technical-integration-readme) | 系统集成 | Open Interface、Public API、ISG、事件、文件、幂等和对账 | [技术接口手册](#src-docs-09-technical-interfaces) |
| [concurrent-processing](#src-docs-10-technical-concurrent-processing-readme) | 并发处理 | Program、Request Set、Manager、队列、OPP 和诊断 | [并发程序详文](#src-docs-09-technical-concurrent-programs) |
| [workflow-and-ame](#src-docs-10-technical-workflow-and-ame-readme) | 工作流与审批 | Item/Activity、Notification、Mailer、AME Rule、重试和清理 | [Workflow/AME 治理](#src-docs-09-technical-workflow-ame-oaf-governance) |
| [oaf](#src-docs-10-technical-oaf-readme) | Oracle 应用框架 | MVC、EO/VO/AM/CO、Personalization、Extension 和调试 | [定制开发](#src-docs-09-technical-customization) |
| [forms](#src-docs-10-technical-forms-readme) | Oracle Forms | Template、PLL、Personalization、编译、部署和跟踪 | [定制开发](#src-docs-09-technical-customization) |
| [reporting-development](#src-docs-10-technical-reporting-development-readme) | 报表开发 | BI Publisher、eText、Bursting、FSG、Web ADI、OPP | [报表接口](02-record-to-report.md#src-docs-04-gl-reporting-interfaces) |
| [configuration-migration](#src-docs-10-technical-configuration-migration-readme) | 配置迁移 | FNDLOAD、WFLOAD、XDOLoader、依赖、环境差异和验证 | [迁移治理](#src-docs-09-technical-workflow-ame-oaf-governance) |
| [adop-and-ebr](#src-docs-10-technical-adop-and-ebr-readme) | 在线补丁与 EBR | Run/Patch、Edition、ADOP 周期、同步、恢复和边界 | [ADOP/EBR 详文](#src-docs-09-technical-adop-ebr-release) |
| [database-platform](#src-docs-10-technical-database-platform-readme) | 数据库平台 | Schema、存储、统计、锁、备份、RAC/Data Guard 边界 | [运维详文](#src-docs-09-technical-operations) |
| [middleware](#src-docs-10-technical-middleware-readme) | 中间件 | OHS、WebLogic、Node Manager、JVM、TLS、Wallet 和日志 | [运维详文](#src-docs-09-technical-operations) |
| [performance](#src-docs-10-technical-performance-readme) | 性能 | 请求、SQL、锁、执行计划、JVM、OPP、容量和许可证 | [运维详文](#src-docs-09-technical-operations) |
| [security](#src-docs-10-technical-security-readme) | 技术安全 | 用户/职责、RBAC、MOAC、TLS、SSO、DMZ、最小权限 | [财务安全](01-foundation.md#src-docs-01-common-security) |
| [audit-and-compliance](#src-docs-10-technical-audit-and-compliance-readme) | 审计与合规 | AuditTrail、Sign-On、WHO、SoD、保留和脱敏 | [生产安全边界](00-guide.md#src-docs-00-guide-safety-and-production-boundaries) |
| [testing-and-automation](#src-docs-10-technical-testing-and-automation-readme) | 测试与自动化 | 接口契约、会计断言、回归、性能、安全和测试数据 | [角色学习路径](00-guide.md#src-docs-00-guide-reading-paths-by-role) |
| [release-engineering](#src-docs-10-technical-release-engineering-readme) | 发布工程 | Git、评审、依赖、Patch Driver、晋级、回退和证据 | [ADOP/EBR 详文](#src-docs-09-technical-adop-ebr-release) |

<a id="src-docs-10-technical-readme--技术顾问的最低能力"></a>
### 技术顾问的最低能力

<a id="src-docs-10-technical-readme--业务与数据"></a>
#### 业务与数据

- 能说明一笔 AP/AR/FA/INV 交易的业务状态、组织、账簿、期间和币种。
- 能从业务主键追溯 `XLA_TRANSACTION_ENTITIES`、`XLA_EVENTS`、`XLA_AE_HEADERS/LINES`，并在保留导入引用时追溯 GL Journal。
- 能解释 `_ALL`、`_B`、`_TL`、`_VL`、Interface 和临时对象的常见含义，同时用目标实例验证而非仅凭命名猜测。
- 能用只读、绑定变量和范围受限的 SQL 收集证据，不把直接 DML 基表当作修复。

<a id="src-docs-10-technical-readme--集成与批处理"></a>
#### 集成与批处理

- 能按业务量、实时性和支持边界选择 Open Interface、Public API、ISG、Business Event、Concurrent 或文件接口。
- 能设计业务唯一键、批次、相关号、幂等、重试上限、人工补偿和数量/金额对账。
- 能区分 HTTP/API/文件/请求“技术成功”和 EBS 业务对象、会计、银行/税务回执“业务成功”。
- 能从请求号定位参数、日志、父子请求、Manager、数据库会话和最终业务结果。

<a id="src-docs-10-technical-readme--r122-发布与运行"></a>
#### R12.2 发布与运行

- 理解 Run/Patch File System、Run/Patch Edition、`fs_ne`、AutoConfig 和 Context File。
- 能说明 `adop` Prepare、Apply、Finalize、Cutover、Cleanup 的目的和验证点。
- 自定义数据库对象、文件和配置迁移具有 EBR 合规说明、版本、依赖、安装/卸载和回退步骤。
- 性能和安全诊断遵守 AWR/ASH/SQL Monitor 等许可证边界及敏感数据最小暴露原则。

<a id="src-docs-10-technical-readme--标准排障框架"></a>
### 标准排障框架

```text
1. 明确实例/版本/时间/用户/职责/组织/账簿/期间
2. 确认业务主键、当前状态、预期状态和最近变更
3. 定位接口批次、并发请求、Workflow Item 或 Web 请求
4. 读取首个有意义错误并确认可否安全重试
5. 验证业务结果、SLA、GL、报表和外部回执
6. 使用标准流程/API/接口或批准的 Support 方案恢复
7. 保存前后数量金额对账、日志和审批证据
```

<a id="src-docs-10-technical-readme--设计评审清单"></a>
### 设计评审清单

| 维度 | 必答问题 |
| --- | --- |
| 支持性 | 是否使用 Oracle 标准页面、公开 API、Open Interface 或受支持扩展点？ |
| 上下文 | 用户、职责、应用、OU、Ledger、NLS 和会计日期如何设置/验证？ |
| 事务 | 谁 Commit/Rollback？部分成功如何处理？是否有补偿？ |
| 幂等 | 重复、乱序、超时重放是否会重复付款/开票/记账？ |
| 对账 | 来源、接口、业务、SLA、GL、外部回执如何按数量和金额闭环？ |
| 安全 | 认证、授权、TLS、证书/密钥、敏感数据和审计如何控制？ |
| 性能 | 峰值量、批次大小、并发度、索引、锁和窗口是否验证？ |
| 可观测性 | 业务键、相关号、请求号、状态、错误分类、指标和告警是否齐全？ |
| 发布 | 工件是否版本化、EBR 合规并纳入 ADOP？环境差异如何注入？ |
| 恢复 | 技术回退、业务冲销、重试上限和人工处置是否明确？ |

<a id="src-docs-10-technical-readme--安全边界"></a>
### 安全边界

1. 禁止直接 DML Oracle EBS 业务、会计、FND 或 Workflow 运行时基表作为常规集成/修复。
2. 禁止覆盖 Oracle 标准文件或在 APPS Schema 中无治理地创建对象。
3. 禁止在未确认上下文、事务和公开性时调用来源不明的 Package。
4. 禁止在生产执行无范围限制的大表查询、Debug 或 Trace。
5. 任何数据修复都应优先使用标准业务反向流程；确需 Support 数据修复时保留 SR、批准、备份、演练和前后对账。

<a id="src-docs-10-technical-readme--官方资料"></a>
### 官方资料

- [Oracle E-Business Suite Technology Documentation](https://docs.oracle.com/cd/E26401_01/nav/technology.htm)
- [Oracle E-Business Suite Concepts](https://docs.oracle.com/cd/E26401_01/doc.122/e22949/toc.htm)
- [Oracle E-Business Suite Maintenance Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22954/toc.htm)
- [Oracle Integrated SOA Gateway Developer's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e20927/toc.htm)
- [Oracle E-Business Suite Electronic Technical Reference Manual User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/)


<!-- source: docs/10-technical/adop-and-ebr/README.md -->
<a id="src-docs-10-technical-adop-and-ebr-readme"></a>
## R12.2 技术专题：adop-and-ebr


<a id="src-docs-10-technical-adop-and-ebr-readme--目标"></a>
### 目标
本专题规范 adop-and-ebr 的设计、部署、运行和故障定位，所有实现均须符合 EBR、ADOP、最小权限和可观测性要求。

<a id="src-docs-10-technical-adop-and-ebr-readme--实施要点"></a>
### 实施要点

明确对象所有者、依赖、配置来源、环境差异、日志、监控、回退和审批。接口/并发程序必须具备业务键、幂等、状态、错误分类和对账；页面和数据库定制优先使用受支持扩展点。

<a id="src-docs-10-technical-adop-and-ebr-readme--r122-边界"></a>
### R12.2 边界

禁止覆盖 Oracle 标准文件或直接修改标准业务对象。发布纳入 ADOP 生命周期，使用版本化工件、校验清单和回归测试；性能诊断先限定请求、会话、SQL 或时间窗口，许可证能力单独确认。

<a id="src-docs-10-technical-adop-and-ebr-readme--诊断"></a>
### 诊断

使用数据字典、并发日志、应用日志、会话信息和对象有效性进行只读诊断；保存请求号、时间、节点、Context 和关联业务键。


<!-- source: docs/10-technical/architecture/README.md -->
<a id="src-docs-10-technical-architecture-readme"></a>
## R12.2 技术专题：architecture


<a id="src-docs-10-technical-architecture-readme--目标"></a>
### 目标
本专题规范 architecture 的设计、部署、运行和故障定位，所有实现均须符合 EBR、ADOP、最小权限和可观测性要求。

<a id="src-docs-10-technical-architecture-readme--实施要点"></a>
### 实施要点

明确对象所有者、依赖、配置来源、环境差异、日志、监控、回退和审批。接口/并发程序必须具备业务键、幂等、状态、错误分类和对账；页面和数据库定制优先使用受支持扩展点。

<a id="src-docs-10-technical-architecture-readme--r122-边界"></a>
### R12.2 边界

禁止覆盖 Oracle 标准文件或直接修改标准业务对象。发布纳入 ADOP 生命周期，使用版本化工件、校验清单和回归测试；性能诊断先限定请求、会话、SQL 或时间窗口，许可证能力单独确认。

<a id="src-docs-10-technical-architecture-readme--诊断"></a>
### 诊断

使用数据字典、并发日志、应用日志、会话信息和对象有效性进行只读诊断；保存请求号、时间、节点、Context 和关联业务键。


<!-- source: docs/10-technical/audit-and-compliance/README.md -->
<a id="src-docs-10-technical-audit-and-compliance-readme"></a>
## R12.2 技术专题：audit-and-compliance


<a id="src-docs-10-technical-audit-and-compliance-readme--目标"></a>
### 目标
本专题规范 audit-and-compliance 的设计、部署、运行和故障定位，所有实现均须符合 EBR、ADOP、最小权限和可观测性要求。

<a id="src-docs-10-technical-audit-and-compliance-readme--实施要点"></a>
### 实施要点

明确对象所有者、依赖、配置来源、环境差异、日志、监控、回退和审批。接口/并发程序必须具备业务键、幂等、状态、错误分类和对账；页面和数据库定制优先使用受支持扩展点。

<a id="src-docs-10-technical-audit-and-compliance-readme--r122-边界"></a>
### R12.2 边界

禁止覆盖 Oracle 标准文件或直接修改标准业务对象。发布纳入 ADOP 生命周期，使用版本化工件、校验清单和回归测试；性能诊断先限定请求、会话、SQL 或时间窗口，许可证能力单独确认。

<a id="src-docs-10-technical-audit-and-compliance-readme--诊断"></a>
### 诊断

使用数据字典、并发日志、应用日志、会话信息和对象有效性进行只读诊断；保存请求号、时间、节点、Context 和关联业务键。


<!-- source: docs/10-technical/concurrent-processing/README.md -->
<a id="src-docs-10-technical-concurrent-processing-readme"></a>
## R12.2 技术专题：concurrent-processing


<a id="src-docs-10-technical-concurrent-processing-readme--目标"></a>
### 目标
本专题规范 concurrent-processing 的设计、部署、运行和故障定位，所有实现均须符合 EBR、ADOP、最小权限和可观测性要求。

<a id="src-docs-10-technical-concurrent-processing-readme--实施要点"></a>
### 实施要点

明确对象所有者、依赖、配置来源、环境差异、日志、监控、回退和审批。接口/并发程序必须具备业务键、幂等、状态、错误分类和对账；页面和数据库定制优先使用受支持扩展点。

<a id="src-docs-10-technical-concurrent-processing-readme--r122-边界"></a>
### R12.2 边界

禁止覆盖 Oracle 标准文件或直接修改标准业务对象。发布纳入 ADOP 生命周期，使用版本化工件、校验清单和回归测试；性能诊断先限定请求、会话、SQL 或时间窗口，许可证能力单独确认。

<a id="src-docs-10-technical-concurrent-processing-readme--诊断"></a>
### 诊断

使用数据字典、并发日志、应用日志、会话信息和对象有效性进行只读诊断；保存请求号、时间、节点、Context 和关联业务键。


<!-- source: docs/10-technical/configuration-migration/README.md -->
<a id="src-docs-10-technical-configuration-migration-readme"></a>
## R12.2 技术专题：configuration-migration


<a id="src-docs-10-technical-configuration-migration-readme--目标"></a>
### 目标
本专题规范 configuration-migration 的设计、部署、运行和故障定位，所有实现均须符合 EBR、ADOP、最小权限和可观测性要求。

<a id="src-docs-10-technical-configuration-migration-readme--实施要点"></a>
### 实施要点

明确对象所有者、依赖、配置来源、环境差异、日志、监控、回退和审批。接口/并发程序必须具备业务键、幂等、状态、错误分类和对账；页面和数据库定制优先使用受支持扩展点。

<a id="src-docs-10-technical-configuration-migration-readme--r122-边界"></a>
### R12.2 边界

禁止覆盖 Oracle 标准文件或直接修改标准业务对象。发布纳入 ADOP 生命周期，使用版本化工件、校验清单和回归测试；性能诊断先限定请求、会话、SQL 或时间窗口，许可证能力单独确认。

<a id="src-docs-10-technical-configuration-migration-readme--诊断"></a>
### 诊断

使用数据字典、并发日志、应用日志、会话信息和对象有效性进行只读诊断；保存请求号、时间、节点、Context 和关联业务键。


<!-- source: docs/10-technical/data-model/README.md -->
<a id="src-docs-10-technical-data-model-readme"></a>
## R12.2 技术专题：data-model


<a id="src-docs-10-technical-data-model-readme--目标"></a>
### 目标
本专题规范 data-model 的设计、部署、运行和故障定位，所有实现均须符合 EBR、ADOP、最小权限和可观测性要求。

<a id="src-docs-10-technical-data-model-readme--实施要点"></a>
### 实施要点

明确对象所有者、依赖、配置来源、环境差异、日志、监控、回退和审批。接口/并发程序必须具备业务键、幂等、状态、错误分类和对账；页面和数据库定制优先使用受支持扩展点。

<a id="src-docs-10-technical-data-model-readme--r122-边界"></a>
### R12.2 边界

禁止覆盖 Oracle 标准文件或直接修改标准业务对象。发布纳入 ADOP 生命周期，使用版本化工件、校验清单和回归测试；性能诊断先限定请求、会话、SQL 或时间窗口，许可证能力单独确认。

<a id="src-docs-10-technical-data-model-readme--诊断"></a>
### 诊断

使用数据字典、并发日志、应用日志、会话信息和对象有效性进行只读诊断；保存请求号、时间、节点、Context 和关联业务键。


<!-- source: docs/10-technical/database-development/README.md -->
<a id="src-docs-10-technical-database-development-readme"></a>
## R12.2 技术专题：database-development


<a id="src-docs-10-technical-database-development-readme--目标"></a>
### 目标
本专题规范 database-development 的设计、部署、运行和故障定位，所有实现均须符合 EBR、ADOP、最小权限和可观测性要求。

<a id="src-docs-10-technical-database-development-readme--实施要点"></a>
### 实施要点

明确对象所有者、依赖、配置来源、环境差异、日志、监控、回退和审批。接口/并发程序必须具备业务键、幂等、状态、错误分类和对账；页面和数据库定制优先使用受支持扩展点。

<a id="src-docs-10-technical-database-development-readme--r122-边界"></a>
### R12.2 边界

禁止覆盖 Oracle 标准文件或直接修改标准业务对象。发布纳入 ADOP 生命周期，使用版本化工件、校验清单和回归测试；性能诊断先限定请求、会话、SQL 或时间窗口，许可证能力单独确认。

<a id="src-docs-10-technical-database-development-readme--诊断"></a>
### 诊断

使用数据字典、并发日志、应用日志、会话信息和对象有效性进行只读诊断；保存请求号、时间、节点、Context 和关联业务键。


<!-- source: docs/10-technical/database-platform/README.md -->
<a id="src-docs-10-technical-database-platform-readme"></a>
## R12.2 技术专题：database-platform


<a id="src-docs-10-technical-database-platform-readme--目标"></a>
### 目标
本专题规范 database-platform 的设计、部署、运行和故障定位，所有实现均须符合 EBR、ADOP、最小权限和可观测性要求。

<a id="src-docs-10-technical-database-platform-readme--实施要点"></a>
### 实施要点

明确对象所有者、依赖、配置来源、环境差异、日志、监控、回退和审批。接口/并发程序必须具备业务键、幂等、状态、错误分类和对账；页面和数据库定制优先使用受支持扩展点。

<a id="src-docs-10-technical-database-platform-readme--r122-边界"></a>
### R12.2 边界

禁止覆盖 Oracle 标准文件或直接修改标准业务对象。发布纳入 ADOP 生命周期，使用版本化工件、校验清单和回归测试；性能诊断先限定请求、会话、SQL 或时间窗口，许可证能力单独确认。

<a id="src-docs-10-technical-database-platform-readme--诊断"></a>
### 诊断

使用数据字典、并发日志、应用日志、会话信息和对象有效性进行只读诊断；保存请求号、时间、节点、Context 和关联业务键。


<!-- source: docs/10-technical/forms/README.md -->
<a id="src-docs-10-technical-forms-readme"></a>
## R12.2 技术专题：forms


<a id="src-docs-10-technical-forms-readme--目标"></a>
### 目标
本专题规范 forms 的设计、部署、运行和故障定位，所有实现均须符合 EBR、ADOP、最小权限和可观测性要求。

<a id="src-docs-10-technical-forms-readme--实施要点"></a>
### 实施要点

明确对象所有者、依赖、配置来源、环境差异、日志、监控、回退和审批。接口/并发程序必须具备业务键、幂等、状态、错误分类和对账；页面和数据库定制优先使用受支持扩展点。

<a id="src-docs-10-technical-forms-readme--r122-边界"></a>
### R12.2 边界

禁止覆盖 Oracle 标准文件或直接修改标准业务对象。发布纳入 ADOP 生命周期，使用版本化工件、校验清单和回归测试；性能诊断先限定请求、会话、SQL 或时间窗口，许可证能力单独确认。

<a id="src-docs-10-technical-forms-readme--诊断"></a>
### 诊断

使用数据字典、并发日志、应用日志、会话信息和对象有效性进行只读诊断；保存请求号、时间、节点、Context 和关联业务键。


<!-- source: docs/10-technical/integration/README.md -->
<a id="src-docs-10-technical-integration-readme"></a>
## R12.2 技术专题：integration


<a id="src-docs-10-technical-integration-readme--目标"></a>
### 目标
本专题规范 integration 的设计、部署、运行和故障定位，所有实现均须符合 EBR、ADOP、最小权限和可观测性要求。

<a id="src-docs-10-technical-integration-readme--实施要点"></a>
### 实施要点

明确对象所有者、依赖、配置来源、环境差异、日志、监控、回退和审批。接口/并发程序必须具备业务键、幂等、状态、错误分类和对账；页面和数据库定制优先使用受支持扩展点。

<a id="src-docs-10-technical-integration-readme--r122-边界"></a>
### R12.2 边界

禁止覆盖 Oracle 标准文件或直接修改标准业务对象。发布纳入 ADOP 生命周期，使用版本化工件、校验清单和回归测试；性能诊断先限定请求、会话、SQL 或时间窗口，许可证能力单独确认。

<a id="src-docs-10-technical-integration-readme--诊断"></a>
### 诊断

使用数据字典、并发日志、应用日志、会话信息和对象有效性进行只读诊断；保存请求号、时间、节点、Context 和关联业务键。


<!-- source: docs/10-technical/middleware/README.md -->
<a id="src-docs-10-technical-middleware-readme"></a>
## R12.2 技术专题：middleware


<a id="src-docs-10-technical-middleware-readme--目标"></a>
### 目标
本专题规范 middleware 的设计、部署、运行和故障定位，所有实现均须符合 EBR、ADOP、最小权限和可观测性要求。

<a id="src-docs-10-technical-middleware-readme--实施要点"></a>
### 实施要点

明确对象所有者、依赖、配置来源、环境差异、日志、监控、回退和审批。接口/并发程序必须具备业务键、幂等、状态、错误分类和对账；页面和数据库定制优先使用受支持扩展点。

<a id="src-docs-10-technical-middleware-readme--r122-边界"></a>
### R12.2 边界

禁止覆盖 Oracle 标准文件或直接修改标准业务对象。发布纳入 ADOP 生命周期，使用版本化工件、校验清单和回归测试；性能诊断先限定请求、会话、SQL 或时间窗口，许可证能力单独确认。

<a id="src-docs-10-technical-middleware-readme--诊断"></a>
### 诊断

使用数据字典、并发日志、应用日志、会话信息和对象有效性进行只读诊断；保存请求号、时间、节点、Context 和关联业务键。


<!-- source: docs/10-technical/oaf/README.md -->
<a id="src-docs-10-technical-oaf-readme"></a>
## R12.2 技术专题：oaf


<a id="src-docs-10-technical-oaf-readme--目标"></a>
### 目标
本专题规范 oaf 的设计、部署、运行和故障定位，所有实现均须符合 EBR、ADOP、最小权限和可观测性要求。

<a id="src-docs-10-technical-oaf-readme--实施要点"></a>
### 实施要点

明确对象所有者、依赖、配置来源、环境差异、日志、监控、回退和审批。接口/并发程序必须具备业务键、幂等、状态、错误分类和对账；页面和数据库定制优先使用受支持扩展点。

<a id="src-docs-10-technical-oaf-readme--r122-边界"></a>
### R12.2 边界

禁止覆盖 Oracle 标准文件或直接修改标准业务对象。发布纳入 ADOP 生命周期，使用版本化工件、校验清单和回归测试；性能诊断先限定请求、会话、SQL 或时间窗口，许可证能力单独确认。

<a id="src-docs-10-technical-oaf-readme--诊断"></a>
### 诊断

使用数据字典、并发日志、应用日志、会话信息和对象有效性进行只读诊断；保存请求号、时间、节点、Context 和关联业务键。


<!-- source: docs/10-technical/performance/README.md -->
<a id="src-docs-10-technical-performance-readme"></a>
## R12.2 技术专题：performance


<a id="src-docs-10-technical-performance-readme--目标"></a>
### 目标
本专题规范 performance 的设计、部署、运行和故障定位，所有实现均须符合 EBR、ADOP、最小权限和可观测性要求。

<a id="src-docs-10-technical-performance-readme--实施要点"></a>
### 实施要点

明确对象所有者、依赖、配置来源、环境差异、日志、监控、回退和审批。接口/并发程序必须具备业务键、幂等、状态、错误分类和对账；页面和数据库定制优先使用受支持扩展点。

<a id="src-docs-10-technical-performance-readme--r122-边界"></a>
### R12.2 边界

禁止覆盖 Oracle 标准文件或直接修改标准业务对象。发布纳入 ADOP 生命周期，使用版本化工件、校验清单和回归测试；性能诊断先限定请求、会话、SQL 或时间窗口，许可证能力单独确认。

<a id="src-docs-10-technical-performance-readme--诊断"></a>
### 诊断

使用数据字典、并发日志、应用日志、会话信息和对象有效性进行只读诊断；保存请求号、时间、节点、Context 和关联业务键。


<!-- source: docs/10-technical/release-engineering/README.md -->
<a id="src-docs-10-technical-release-engineering-readme"></a>
## R12.2 技术专题：release-engineering


<a id="src-docs-10-technical-release-engineering-readme--目标"></a>
### 目标
本专题规范 release-engineering 的设计、部署、运行和故障定位，所有实现均须符合 EBR、ADOP、最小权限和可观测性要求。

<a id="src-docs-10-technical-release-engineering-readme--实施要点"></a>
### 实施要点

明确对象所有者、依赖、配置来源、环境差异、日志、监控、回退和审批。接口/并发程序必须具备业务键、幂等、状态、错误分类和对账；页面和数据库定制优先使用受支持扩展点。

<a id="src-docs-10-technical-release-engineering-readme--r122-边界"></a>
### R12.2 边界

禁止覆盖 Oracle 标准文件或直接修改标准业务对象。发布纳入 ADOP 生命周期，使用版本化工件、校验清单和回归测试；性能诊断先限定请求、会话、SQL 或时间窗口，许可证能力单独确认。

<a id="src-docs-10-technical-release-engineering-readme--诊断"></a>
### 诊断

使用数据字典、并发日志、应用日志、会话信息和对象有效性进行只读诊断；保存请求号、时间、节点、Context 和关联业务键。


<!-- source: docs/10-technical/reporting-development/README.md -->
<a id="src-docs-10-technical-reporting-development-readme"></a>
## R12.2 技术专题：reporting-development


<a id="src-docs-10-technical-reporting-development-readme--目标"></a>
### 目标
本专题规范 reporting-development 的设计、部署、运行和故障定位，所有实现均须符合 EBR、ADOP、最小权限和可观测性要求。

<a id="src-docs-10-technical-reporting-development-readme--实施要点"></a>
### 实施要点

明确对象所有者、依赖、配置来源、环境差异、日志、监控、回退和审批。接口/并发程序必须具备业务键、幂等、状态、错误分类和对账；页面和数据库定制优先使用受支持扩展点。

<a id="src-docs-10-technical-reporting-development-readme--r122-边界"></a>
### R12.2 边界

禁止覆盖 Oracle 标准文件或直接修改标准业务对象。发布纳入 ADOP 生命周期，使用版本化工件、校验清单和回归测试；性能诊断先限定请求、会话、SQL 或时间窗口，许可证能力单独确认。

<a id="src-docs-10-technical-reporting-development-readme--诊断"></a>
### 诊断

使用数据字典、并发日志、应用日志、会话信息和对象有效性进行只读诊断；保存请求号、时间、节点、Context 和关联业务键。


<!-- source: docs/10-technical/security/README.md -->
<a id="src-docs-10-technical-security-readme"></a>
## R12.2 技术专题：security


<a id="src-docs-10-technical-security-readme--目标"></a>
### 目标
本专题规范 security 的设计、部署、运行和故障定位，所有实现均须符合 EBR、ADOP、最小权限和可观测性要求。

<a id="src-docs-10-technical-security-readme--实施要点"></a>
### 实施要点

明确对象所有者、依赖、配置来源、环境差异、日志、监控、回退和审批。接口/并发程序必须具备业务键、幂等、状态、错误分类和对账；页面和数据库定制优先使用受支持扩展点。

<a id="src-docs-10-technical-security-readme--r122-边界"></a>
### R12.2 边界

禁止覆盖 Oracle 标准文件或直接修改标准业务对象。发布纳入 ADOP 生命周期，使用版本化工件、校验清单和回归测试；性能诊断先限定请求、会话、SQL 或时间窗口，许可证能力单独确认。

<a id="src-docs-10-technical-security-readme--诊断"></a>
### 诊断

使用数据字典、并发日志、应用日志、会话信息和对象有效性进行只读诊断；保存请求号、时间、节点、Context 和关联业务键。


<!-- source: docs/10-technical/testing-and-automation/README.md -->
<a id="src-docs-10-technical-testing-and-automation-readme"></a>
## R12.2 技术专题：testing-and-automation


<a id="src-docs-10-technical-testing-and-automation-readme--目标"></a>
### 目标
本专题规范 testing-and-automation 的设计、部署、运行和故障定位，所有实现均须符合 EBR、ADOP、最小权限和可观测性要求。

<a id="src-docs-10-technical-testing-and-automation-readme--实施要点"></a>
### 实施要点

明确对象所有者、依赖、配置来源、环境差异、日志、监控、回退和审批。接口/并发程序必须具备业务键、幂等、状态、错误分类和对账；页面和数据库定制优先使用受支持扩展点。

<a id="src-docs-10-technical-testing-and-automation-readme--r122-边界"></a>
### R12.2 边界

禁止覆盖 Oracle 标准文件或直接修改标准业务对象。发布纳入 ADOP 生命周期，使用版本化工件、校验清单和回归测试；性能诊断先限定请求、会话、SQL 或时间窗口，许可证能力单独确认。

<a id="src-docs-10-technical-testing-and-automation-readme--诊断"></a>
### 诊断

使用数据字典、并发日志、应用日志、会话信息和对象有效性进行只读诊断；保存请求号、时间、节点、Context 和关联业务键。


<!-- source: docs/10-technical/workflow-and-ame/README.md -->
<a id="src-docs-10-technical-workflow-and-ame-readme"></a>
## R12.2 技术专题：workflow-and-ame


<a id="src-docs-10-technical-workflow-and-ame-readme--目标"></a>
### 目标
本专题规范 workflow-and-ame 的设计、部署、运行和故障定位，所有实现均须符合 EBR、ADOP、最小权限和可观测性要求。

<a id="src-docs-10-technical-workflow-and-ame-readme--实施要点"></a>
### 实施要点

明确对象所有者、依赖、配置来源、环境差异、日志、监控、回退和审批。接口/并发程序必须具备业务键、幂等、状态、错误分类和对账；页面和数据库定制优先使用受支持扩展点。

<a id="src-docs-10-technical-workflow-and-ame-readme--r122-边界"></a>
### R12.2 边界

禁止覆盖 Oracle 标准文件或直接修改标准业务对象。发布纳入 ADOP 生命周期，使用版本化工件、校验清单和回归测试；性能诊断先限定请求、会话、SQL 或时间窗口，许可证能力单独确认。

<a id="src-docs-10-technical-workflow-and-ame-readme--诊断"></a>
### 诊断

使用数据字典、并发日志、应用日志、会话信息和对象有效性进行只读诊断；保存请求号、时间、节点、Context 和关联业务键。


<!-- source: docs/09-technical/README.md -->
<a id="src-docs-09-technical-readme"></a>
## EBS R12.2 技术、集成与运维


本目录覆盖 R12.2 技术开发和生产运行的公共规范。业务模块接口文档描述产品入口；本目录定义接口选型、并发程序、PL/SQL、Workflow、OAF/Forms、迁移、EBR/ADOP、性能、安全和可观测性的共性边界。

<a id="src-docs-09-technical-readme--专题导航"></a>
### 专题导航

- [开放接口、API、报表与迁移](#src-docs-09-technical-integration)
- [技术接口实现手册](#src-docs-09-technical-interfaces)
- [数据模型与元数据](#src-docs-09-technical-data-model)
- [Concurrent Program](#src-docs-09-technical-concurrent-programs)
- [PL/SQL、Forms、Personalization 与 OAF 定制](#src-docs-09-technical-customization)
- [性能、权限审计与生产运维](#src-docs-09-technical-operations)
- [R12.2 ADOP、EBR 与发布治理](#src-docs-09-technical-adop-ebr-release)
- [Workflow、AME、OAF/Forms 与迁移治理](#src-docs-09-technical-workflow-ame-oaf-governance)
- [FND、Concurrent、Workflow 表](#src-docs-09-technical-tables)

<a id="src-docs-09-technical-readme--r122-不可省略的边界"></a>
### R12.2 不可省略的边界

1. 定制对象和部署必须遵循 Edition-Based Redefinition 与 Online Patching（ADOP）约束；不得以覆盖 Oracle 标准文件或直接修改标准对象作为常规交付方式。
2. 支持写入的路径依次为标准页面、公开 API、Open Interface、Integration Repository/ISG 和客户自定义对象；禁止直接 DML EBS 业务基表修复数据。
3. 接口应具备幂等键、状态机、错误分类、审计相关号、最小权限、监控、重试上限和人工补偿入口。
4. 性能问题先以并发请求、日志、SQL 执行计划和应用上下文定位；AWR、ASH、SQL Monitor 等能力须确认数据库许可证。

<a id="src-docs-09-technical-readme--官方依据"></a>
### 官方依据

- [Oracle E-Business Suite Technology Documentation](https://docs.oracle.com/cd/E26401_01/nav/technology.htm)
- [Oracle E-Business Suite Maintenance Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22954/toc.htm)


<!-- source: docs/09-technical/adop-ebr-release.md -->
<a id="src-docs-09-technical-adop-ebr-release"></a>
## R12.2 Online Patching、EBR 与发布治理


<a id="src-docs-09-technical-adop-ebr-release--核心原则"></a>
### 核心原则

R12.2 使用 Online Patching（ADOP）与 Edition-Based Redefinition（EBR）降低在线补丁对业务的影响，但并不意味着自定义对象可以任意创建、修改或直接在生产库修复。每次交付均应识别对象类型、Edition 属性、依赖、部署方式、回滚策略和验证证据。

<a id="src-docs-09-technical-adop-ebr-release--adop-生命周期"></a>
### ADOP 生命周期

```text
Prepare → Apply → Finalize → Cutover → Cleanup
                         ↘ 必要时按 Oracle 受支持流程 Abort/Recover
```

| 阶段 | 目标 | 发布控制 |
| --- | --- | --- |
| Prepare | 建立 Patch 文件系统与同步前提 | 检查环境健康、磁盘、服务、已有会话和未完成周期 |
| Apply | 应用 Oracle/自定义补丁 | 记录 patch 日志、失败对象、依赖、重跑范围 |
| Finalize | 将耗时工作前置 | 评估业务窗口、并发请求、OPP/Workflow/接口影响 |
| Cutover | 切换运行文件系统/Edition | 设置冻结、变更窗口、健康检查、业务签字与回退判定 |
| Cleanup | 清理旧版定义 | 归档日志、更新基线、完成回归与配置对比 |

<a id="src-docs-09-technical-adop-ebr-release--自定义交付清单"></a>
### 自定义交付清单

1. 源码、数据库对象、Forms/OAF、报表、FND 注册、Workflow、Profile/Lookup 分别建立受版本控制的迁移工件。
2. 自定义数据库对象使用独立 schema、最小授权和受支持 synonym/grant；不得直接修改 Oracle 标准对象。
3. 在开发、测试、预生产完成 ADOP 演练，验证在线/切换窗口、并发程序、接口、会计和核心报表。
4. 发布包应具有唯一版本、依赖清单、校验 SQL、回滚/补偿方案、日志位置和责任人。

<a id="src-docs-09-technical-adop-ebr-release--只读诊断-sql"></a>
### 只读诊断 SQL

```sql
-- 检查自定义对象的有效性；不要在生产直接编译 Oracle 标准对象作为常规修复。
select owner,
       object_type,
       object_name,
       status,
       last_ddl_time
  from all_objects
 where owner = upper(:p_custom_schema)
   and status <> 'VALID'
 order by object_type, object_name;

-- 用数据字典复核对象/列，再决定补丁脚本的兼容性。
select owner, table_name, column_name, data_type, data_length
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-09-technical-adop-ebr-release--常见问题"></a>
### 常见问题

- 只在 Run 文件系统修改文件：下一个 ADOP 周期可能被覆盖，且无法形成可部署基线。
- 直接在生产补数据或编译：可能绕过 Edition、审计和回退设计；应改用受支持 API、补丁或 Oracle Support 方案。
- Cutover 后接口/并发异常：按发布清单检查服务、context、custom library/报表、注册定义、日志与依赖，而不是盲目重跑整批业务。

<a id="src-docs-09-technical-adop-ebr-release--官方参考"></a>
### 官方参考

- [Oracle E-Business Suite Maintenance Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22954/toc.htm)
- [Oracle E-Business Suite Technology Documentation](https://docs.oracle.com/cd/E26401_01/nav/technology.htm)


<!-- source: docs/09-technical/concurrent-programs.md -->
<a id="src-docs-09-technical-concurrent-programs"></a>
## Concurrent Program、请求集与日志排错


<a id="src-docs-09-technical-concurrent-programs--架构"></a>
### 架构

```text
Submit Request → FND_CONCURRENT_REQUESTS(Pending)
→ ICM/Service Manager → Target Manager
→ Worker Process → Executable
→ Log/Output → OPP（XML/PDF/Excel 后处理）
→ Completed Normal/Warning/Error
```

Concurrent Program 关联 Executable、Parameters/Value Sets、Incompatibility、Request Group、Output Format、Printer/Style。Request Set 用 Stage 和 Link 组合多程序，需考虑失败分支、参数默认和重启。

R12.2 在 Online Patching 时使用 `ADZDPATCH` 协调不兼容程序；不要为了让 adop 继续而随意终止 ICM/ADZDPATCH。

<a id="src-docs-09-technical-concurrent-programs--sql"></a>
### SQL

```sql
SELECT r.request_id, r.parent_request_id,
       cp.user_concurrent_program_name,
       r.phase_code, r.status_code, r.hold_flag,
       r.requested_start_date, r.actual_start_date,
       r.actual_completion_date, r.argument_text,
       r.concurrent_process_id, r.oracle_process_id,
       r.logfile_name, r.outfile_name
  FROM fnd_concurrent_requests r
  JOIN fnd_concurrent_programs_vl cp
    ON cp.concurrent_program_id = r.concurrent_program_id
   AND cp.application_id = r.program_application_id
 WHERE r.request_id = :p_request_id;

SELECT q.concurrent_queue_name, q.user_concurrent_queue_name,
       q.node_name, q.running_processes, q.max_processes,
       q.enabled_flag, q.control_code
  FROM fnd_concurrent_queues_vl q
 ORDER BY q.user_concurrent_queue_name;

SELECT request_id, phase_code, status_code,
       requested_start_date, actual_start_date,
       actual_completion_date
  FROM fnd_concurrent_requests
 WHERE phase_code IN ('P','R')
 ORDER BY requested_start_date;
```

<a id="src-docs-09-technical-concurrent-programs--排错"></a>
### 排错

- Pending/Standby：查 Requested Start、Hold、Manager Specialization、Incompatibility、Parent Request、Manager Processes、Node/Work Shift。
- Running 过久：查日志最后进度、DB Session/Wait/Blocking、参数数据量和子请求；先评估 Cancel/Terminate 业务后果。
- Completed Error：从日志第一个有意义的 ORA-/APP-/Exception 开始，不只看最后的通用错误。
- OPP 超时/无输出：查 Data Engine 输出大小、Template/Locale、OPP Service/Thread、Heap、Temporary Directory、Font 和巨大 XML。
- Manager 异常：通过 OAM/标准脚本管理，保留 ICM/Manager/Service Manager 日志，不直接改 FND Queue/Request 状态。

<a id="src-docs-09-technical-concurrent-programs--关联"></a>
### 关联

- [Operations](#src-docs-09-technical-operations)
- [Integration](#src-docs-09-technical-integration)


<!-- source: docs/09-technical/customization.md -->
<a id="src-docs-09-technical-customization"></a>
## PL/SQL、Forms、Personalization 与 OAF 定制


<a id="src-docs-09-technical-customization--r122-定制原则"></a>
### R12.2 定制原则

- 优先级：标准设置 > Personalization/Extension > Published API/Open Interface > 经评审的定制；禁止修改 Oracle seeded 源码和基表。
- 自定义对象使用客户前缀/Schema，通过 APPS Synonym/Grant 接入，所有 DDL 满足 Edition-Based Redefinition（EBR）。
- 数据库对象变更通过 `adop` Online Patching 周期发布，开发环境执行一致性/在线补丁检查。
- Forms Personalization 用于可见性、默认、验证和菜单动作；CUSTOM.pll 仅在 Personalization 无法实现时使用。
- OAF 使用 Personalization 或 Controller Extension，不修改 seeded XML/Java；扩展需评估升级/补丁兼容。

<a id="src-docs-09-technical-customization--plsql-标准"></a>
### PL/SQL 标准

1. 公开 API 前初始化 FND/MOAC 上下文，传入 `p_api_version/p_init_msg_list/p_commit`。
2. 尊重 API 交易边界，默认由调用者 Commit/Rollback，不在底层工具函数隐式提交。
3. 读取 `x_return_status/x_msg_count/x_msg_data` 及 Message Stack，日志记录业务键而非敏感数据。
4. SQL 使用 Bind、明确组织/账簿条件、批量处理和可重启设计，避免 Row-by-row 大批量处理。

<a id="src-docs-09-technical-customization--诊断-sql"></a>
### 诊断 SQL

```sql
SELECT owner, object_name, object_type, status, edition_name
  FROM all_objects
 WHERE object_name = UPPER(:p_object_name)
 ORDER BY owner, object_type;

SELECT owner, name, type, line, position, text
  FROM all_errors
 WHERE name = UPPER(:p_object_name)
 ORDER BY owner, type, sequence;

SELECT lookup_type, lookup_code, meaning, enabled_flag,
       start_date_active, end_date_active
  FROM fnd_lookup_values_vl
 WHERE lookup_type = :p_lookup_type
 ORDER BY lookup_code;
```

<a id="src-docs-09-technical-customization--排错"></a>
### 排错

- 补丁后定制失效：检查 Invalid Objects、API Signature/View Columns 变更、Synonym/Grant、OAF Extension 兼容和 adop 日志。
- Personalization 不生效：查 Function/Form/Page Context、Level/Sequence/Condition、字段名、缓存，使用 Diagnostics 验证实际项。
- API 返回 Error：打印完整 Message Stack、输入 ID/OU/User/Responsibility、前置状态，不只记 `SQLERRM`。
- ORA-04061/4068：可能是 Package 重编译导致会话状态失效，重建会话并检查发布是否遵循 Online Patching。

<a id="src-docs-09-technical-customization--关联"></a>
### 关联

- [Data Model](#src-docs-09-technical-data-model)
- [Operations](#src-docs-09-technical-operations)


<!-- source: docs/09-technical/data-model.md -->
<a id="src-docs-09-technical-data-model"></a>
## EBS R12.2 数据模型与常用表


<a id="src-docs-09-technical-data-model--建模约定"></a>
### 建模约定

- `_ALL` 通常包含 OU 级数据并带 `ORG_ID`；`_B` 为基表，`_TL` 为翻译，`_VL` 常为基表+当前语言视图，`_F` 常为 DateTrack，`_INTERFACE` 为接口，`_TEMP/_GT` 可为临时数据。
- WHO 列：`CREATION_DATE/CREATED_BY/LAST_UPDATE_DATE/LAST_UPDATED_BY/LAST_UPDATE_LOGIN`。
- ID 应作为关联键，业务编号用于显示；多 OU 关联还要核对 `ORG_ID`。
- 历史表通常使用 Effective/Ineffective Date 或 Current Flag，不能在无日期条件时当作当前值。
- APPS 通过 Synonym/View/Package 访问产品 Schema。R12.2 定制对象必须遵循 Editioning/Online Patching 标准。

<a id="src-docs-09-technical-data-model--模块速查"></a>
### 模块速查

| 模块 | 主要对象 |
| --- | --- |
| FND | `FND_USER`, `FND_RESPONSIBILITY_VL`, `FND_CONCURRENT_REQUESTS`, `FND_PROFILE_OPTION_VALUES` |
| HR/ORG | `HR_ALL_ORGANIZATION_UNITS`, `HR_OPERATING_UNITS`, `ORG_ORGANIZATION_DEFINITIONS` |
| TCA | `HZ_PARTIES`, `HZ_CUST_ACCOUNTS`, `HZ_PARTY_SITES`, `HZ_LOCATIONS` |
| GL | `GL_LEDGERS`, `GL_CODE_COMBINATIONS`, `GL_JE_*`, `GL_BALANCES`, `GL_INTERFACE` |
| XLA | `XLA_TRANSACTION_ENTITIES`, `XLA_EVENTS`, `XLA_AE_HEADERS`, `XLA_AE_LINES` |
| AP | `AP_SUPPLIERS`, `AP_INVOICES_ALL`, `AP_INVOICE_DISTRIBUTIONS_ALL`, `AP_CHECKS_ALL` |
| AR | `RA_CUSTOMER_TRX_ALL`, `AR_PAYMENT_SCHEDULES_ALL`, `AR_CASH_RECEIPTS_ALL` |
| PO/RCV | `PO_HEADERS_ALL`, `PO_DISTRIBUTIONS_ALL`, `RCV_TRANSACTIONS` |
| OM/WSH | `OE_ORDER_HEADERS_ALL`, `OE_ORDER_LINES_ALL`, `WSH_DELIVERY_DETAILS` |
| INV/CST/WIP | `MTL_SYSTEM_ITEMS_B`, `MTL_MATERIAL_TRANSACTIONS`, `CST_ITEM_COSTS`, `WIP_ENTITIES` |
| FA | `FA_ADDITIONS_B`, `FA_BOOKS`, `FA_DISTRIBUTION_HISTORY`, `FA_DEPRN_SUMMARY` |
| CE/Tax | `CE_BANK_ACCOUNTS`, `CE_STATEMENT_*`, `ZX_LINES` |

<a id="src-docs-09-technical-data-model--元数据-sql"></a>
### 元数据 SQL

```sql
SELECT owner, object_name, object_type, status
  FROM all_objects
 WHERE object_name = UPPER(:p_object_name)
 ORDER BY owner, object_type;

SELECT owner, table_name, column_id, column_name,
       data_type, data_length, nullable
  FROM all_tab_columns
 WHERE table_name = UPPER(:p_object_name)
 ORDER BY owner, column_id;

SELECT owner, synonym_name, table_owner, table_name, db_link
  FROM all_synonyms
 WHERE synonym_name = UPPER(:p_object_name);

SELECT acc.owner, acc.constraint_name, acc.table_name,
       acc.column_name, ac.constraint_type,
       ac.r_owner, ac.r_constraint_name
  FROM all_cons_columns acc
  JOIN all_constraints ac
    ON ac.owner = acc.owner
   AND ac.constraint_name = acc.constraint_name
 WHERE acc.table_name = UPPER(:p_table_name)
 ORDER BY acc.constraint_name, acc.position;
```

<a id="src-docs-09-technical-data-model--原则与排错"></a>
### 原则与排错

- 先用 eTRM/`ALL_*` 确认对象、列、同义词和约束，不凭记忆跨版本写 SQL。
- 查询必须包含组织/账簿、主键或日期范围，对大表先看执行计划。
- 不直接 DML 基表；使用标准 Form/OAF、Open Interface 或 Published API，数据修复走 Oracle Support。
- 不使用 `SELECT *`、隐式日期转换和字符串拼 SQL；使用显式列、绑定变量和 ANSI Join。

<a id="src-docs-09-technical-data-model--关联"></a>
### 关联

- [FND、Concurrent、Workflow 与运维常用表](#src-docs-09-technical-tables)
- [Integration](#src-docs-09-technical-integration)
- [Operations](#src-docs-09-technical-operations)


<!-- source: docs/09-technical/integration.md -->
<a id="src-docs-09-technical-integration"></a>
## 开放接口、API、报表与数据迁移


> Concurrent Worker、标准 API 模板、ISG REST、重试与可观测性代码见 [技术接口实现手册](#src-docs-09-technical-interfaces)。

<a id="src-docs-09-technical-integration--选型原则"></a>
### 选型原则

| 方式 | 适用场景 | 控制要点 |
| --- | --- | --- |
| Open Interface + Import | 高吞吐异步批量 | Group/Source、拒绝表、幂等、可重启 |
| Published PL/SQL API | 同步单笔/小批量 | FND/MOAC Context、Message Stack、Commit 边界 |
| Business Event/Workflow | 事件驱动 | Event Key、Subscription Phase、重试/错误队列 |
| Integrated SOA Gateway | SOAP/REST 暴露 EBS 接口 | Authentication、Grant、MOAC、限流/审计 |
| XML Gateway/EDI | B2B 标准消息 | Trading Partner、Map、Transaction Type、ACK |
| BI Publisher/Concurrent | 报表/文件 | Data Definition、Template、Locale、OPP/Output |

<a id="src-docs-09-technical-integration--接口分层"></a>
### 接口分层

```text
Source → Landing（原始不可变）
→ Staging（标准化/验证/幂等）
→ EBS Interface/API（标准业务验证）
→ Base Transaction → Accounting
→ Reconciliation/Acknowledgement/Archive
```

每条数据保存 Source System、External Key、Batch/Correlation ID、ORG_ID/Ledger、Payload Hash、Status、Retry Count、Request ID、EBS Transaction ID、Error Code/Message。技术重试必须幂等，业务驳回需修正后重提，不应无限自动重试。

<a id="src-docs-09-technical-integration--sql"></a>
### SQL

```sql
-- 并发请求跟踪
SELECT request_id, parent_request_id, phase_code, status_code,
       argument_text, actual_start_date, actual_completion_date,
       logfile_name, outfile_name
  FROM fnd_concurrent_requests
 WHERE request_id = :p_request_id;

-- Workflow 事件/活动跟踪；生产查询必须限定 Item Type/Key
SELECT item_type, item_key, begin_date, end_date,
       root_activity, root_activity_version
  FROM wf_items
 WHERE item_type = :p_item_type
   AND item_key = :p_item_key;

SELECT item_type, item_key, process_activity,
       activity_status, activity_result_code,
       begin_date, end_date, error_name, error_message
  FROM wf_item_activity_statuses
 WHERE item_type = :p_item_type
   AND item_key = :p_item_key
 ORDER BY begin_date;
```

<a id="src-docs-09-technical-integration--迁移清单"></a>
### 迁移清单

1. 数据 Profile/清洗/映射，确定历史深度和未结业务边界。
2. 按主数据→期初余额→未结交易→历史的依赖顺序导入。
3. 多轮 Mock，每轮保存输入、拒绝、数量/金额对账、性能和时间。
4. Cutover 冻结、Delta、业务签字和回退标准必须事先定义。

<a id="src-docs-09-technical-integration--排错"></a>
### 排错

- 先定位 Landing/Staging/EBS Interface/Import/Base/Accounting 断点，再查对应状态。
- 重复：检查 External Key/Hash、超时后重试、EBS 已成功但 ACK 失败的情况。
- 部分成功：对每条保存 EBS ID/Status，仅重试失败项，不重放整批成功数据。
- 性能：使用批次、Bind/Array Processing、合理 Commit Size、并发限流，避免过度 API 单行循环。

<a id="src-docs-09-technical-integration--关联"></a>
### 关联

- [AP Interface](03-procure-to-pay.md#src-docs-02-ap-interfaces-troubleshooting)
- [AR Interface](04-credit-to-cash.md#src-docs-03-ar-interfaces-troubleshooting)
- [Concurrent Processing](#src-docs-09-technical-concurrent-programs)


<!-- source: docs/09-technical/interfaces.md -->
<a id="src-docs-09-technical-interfaces"></a>
## Oracle EBS 技术接口实现手册


<a id="src-docs-09-technical-interfaces--1-接口方式选型"></a>
### 1. 接口方式选型

| 方式 | 适合场景 | 调用/处理特点 | 主要风险控制 |
| --- | --- | --- | --- |
| Open Interface + Import | 高吞吐异步批量 | 先写接口表，再跑标准 Import | 幂等、Group ID、错误表、对账 |
| Published PL/SQL API | 同步单笔/小批量 | 完整业务验证和 Message Stack | FND/MOAC Context、Commit 语义 |
| ISG REST/SOAP | 对外暴露 EBS 标准能力 | Integration Repository 部署与授权 | HTTPS、Grant、限流、WADL/WSDL 版本 |
| Concurrent Program REST | 长任务/报表/批处理 | REST 仅 POST，异步返回 Request ID | 参数位置、轮询、超时、并发队列 |
| Business Event | EBS 业务事件通知 | 本地/Deferred Subscription | Event Key 幂等、Error Queue |
| Service Invocation Framework | EBS 调外部 SOAP/REST | Workflow BES + Java Deferred Agent | 证书、凭据、回调、Invocation Monitor |
| XML Gateway/EDI | B2B 标准报文 | Trading Partner、Map、ACK | 版本、字符集、签名、重复报文 |

Oracle R12.2 官方说明：PL/SQL API、Concurrent Program、Business Service Object 可暴露为 SOAP/REST；Inbound Open Interface REST 支持 POST/GET/PUT/DELETE；Concurrent Program REST 只支持 POST。自定义 Open Interface 表/视图不能直接作为 Integration Repository 的 Custom Interface 类型发布，应使用自定义 PL/SQL API 或标准接口暂存层。

<a id="src-docs-09-technical-interfaces--2-自定义-concurrent-program-worker"></a>
### 2. 自定义 Concurrent Program Worker

<a id="src-docs-09-technical-interfaces--21-package-specification"></a>
#### 2.1 Package Specification

```sql
CREATE OR REPLACE PACKAGE xxint_worker_pkg AUTHID DEFINER AS
  PROCEDURE main(
    errbuf           OUT VARCHAR2,
    retcode          OUT NUMBER,
    p_interface_code IN  VARCHAR2,
    p_batch_size     IN  NUMBER DEFAULT 500
  );
END xxint_worker_pkg;
/
```

<a id="src-docs-09-technical-interfaces--22-package-body"></a>
#### 2.2 Package Body

```sql
CREATE OR REPLACE PACKAGE BODY xxint_worker_pkg AS
  PROCEDURE main(
    errbuf           OUT VARCHAR2,
    retcode          OUT NUMBER,
    p_interface_code IN  VARCHAR2,
    p_batch_size     IN  NUMBER DEFAULT 500
  ) IS
    CURSOR c_message IS
      SELECT message_id
        FROM xxint_messages
       WHERE interface_code = p_interface_code
         AND status_code IN ('VALIDATED', 'RETRY')
         AND NVL(next_retry_date, SYSDATE) <= SYSDATE
       ORDER BY message_id
       FOR UPDATE SKIP LOCKED;

    l_ebs_transaction_id NUMBER;
    l_request_id         NUMBER;
    l_success_count      PLS_INTEGER := 0;
    l_error_count        PLS_INTEGER := 0;
    l_processed_count    PLS_INTEGER := 0;
  BEGIN
    errbuf := NULL;
    retcode := 0;

    fnd_file.put_line(fnd_file.log,
      'Start interface=' || p_interface_code ||
      ', batch_size=' || p_batch_size);

    FOR r IN c_message LOOP
      EXIT WHEN l_processed_count >= p_batch_size;
      l_processed_count := l_processed_count + 1;
      SAVEPOINT one_message;
      BEGIN
        UPDATE xxint_messages
           SET status_code = 'SUBMITTED',
               last_update_date = SYSDATE,
               last_updated_by = fnd_global.user_id
         WHERE message_id = r.message_id;

        -- Router 内部调用 AP/AR/GL/FA/INV 的标准 API 或 Open Interface。
        xxint_router.process_message(
          p_message_id         => r.message_id,
          x_ebs_transaction_id => l_ebs_transaction_id,
          x_request_id         => l_request_id
        );

        UPDATE xxint_messages
           SET status_code = CASE
                               WHEN l_request_id IS NULL THEN 'SUCCESS'
                               ELSE 'SUBMITTED'
                             END,
               ebs_transaction_id = l_ebs_transaction_id,
               request_id = l_request_id,
               error_code = NULL,
               error_message = NULL,
               last_update_date = SYSDATE,
               last_updated_by = fnd_global.user_id
         WHERE message_id = r.message_id;

        l_success_count := l_success_count + 1;
        COMMIT;
      EXCEPTION
        WHEN OTHERS THEN
          ROLLBACK TO one_message;
          UPDATE xxint_messages
             SET status_code = CASE
                                 WHEN retry_count + 1 >= 5 THEN 'DEAD'
                                 ELSE 'RETRY'
                               END,
                 retry_count = retry_count + 1,
                 next_retry_date = SYSDATE
                   + (POWER(2, LEAST(retry_count + 1, 8)) / 1440),
                 error_code = TO_CHAR(SQLCODE),
                 error_message = SUBSTR(SQLERRM, 1, 2000),
                 last_update_date = SYSDATE,
                 last_updated_by = fnd_global.user_id
           WHERE message_id = r.message_id;
          l_error_count := l_error_count + 1;
          COMMIT;
      END;
    END LOOP;

    IF l_error_count > 0 THEN
      retcode := 1; -- Warning
      errbuf := l_error_count || ' message(s) failed';
    END IF;

    fnd_file.put_line(fnd_file.output,
      'success=' || l_success_count || ', error=' || l_error_count);
  EXCEPTION
    WHEN OTHERS THEN
      ROLLBACK;
      retcode := 2;
      errbuf := SUBSTR(SQLERRM, 1, 240);
      fnd_file.put_line(fnd_file.log,
        'Fatal error: ' || DBMS_UTILITY.format_error_backtrace);
  END main;
END xxint_worker_pkg;
/
```

R12.2 自定义数据库对象应放在自定义 Schema/APPS 同义词策略下，并通过 ADOP/Edition-Based Redefinition 合规发布。Worker 使用 `FOR UPDATE SKIP LOCKED` 支持多并发实例，并在达到批次大小后停止领取。

<a id="src-docs-09-technical-interfaces--3-标准-plsql-api-调用模板"></a>
### 3. 标准 PL/SQL API 调用模板

```sql
DECLARE
  l_return_status VARCHAR2(1);
  l_msg_count     NUMBER;
  l_msg_data      VARCHAR2(2000);
BEGIN
  fnd_global.apps_initialize(:p_user_id, :p_resp_id, :p_resp_appl_id);
  mo_global.init(:p_application_short_name);
  mo_global.set_policy_context('S', :p_org_id);

  fnd_msg_pub.initialize;

  xx_public_api.do_business_action(
    p_api_version      => 1.0,
    p_init_msg_list    => fnd_api.g_true,
    p_commit           => fnd_api.g_false,
    p_business_key     => :p_business_key,
    x_return_status    => l_return_status,
    x_msg_count        => l_msg_count,
    x_msg_data         => l_msg_data
  );

  IF l_return_status <> fnd_api.g_ret_sts_success THEN
    FOR i IN 1 .. l_msg_count LOOP
      dbms_output.put_line(
        fnd_msg_pub.get(p_msg_index => i, p_encoded => 'F'));
    END LOOP;
    ROLLBACK;
    raise_application_error(-20060,
      'API failed: ' || SUBSTR(l_msg_data, 1, 1800));
  END IF;

  COMMIT;
END;
/
```

`XX_PUBLIC_API` 只是调用模板占位符。实际 API 名、版本、Record Type、`G_MISS_*` 默认值、是否自动 Commit，必须从当前实例 Integration Repository/Package Specification 获取。

<a id="src-docs-09-technical-interfaces--4-isg-rest-服务部署与调用"></a>
### 4. ISG REST 服务部署与调用

<a id="src-docs-09-technical-interfaces--41-管理端步骤"></a>
#### 4.1 管理端步骤

1. 在 Integration Repository 按 Internal Name 搜索标准 API/Concurrent/Open Interface。
2. 检查方法签名、方向、生命周期状态和产品补丁说明。
3. 在 REST Web Service 页设置唯一 Service Alias，仅勾选需要的方法/HTTP Verb。
4. 设置 Authentication Type，Deploy 后为专用用户建立 Grant。
5. 从当前实例下载 WADL/XSD，生成客户端契约测试。
6. 使用最小权限职责/MOAC/Data Access Set 验证不同组织的数据隔离。

<a id="src-docs-09-technical-interfaces--42-rest-调用"></a>
#### 4.2 REST 调用

```bash
curl --fail-with-body \
  --request POST \
  --url 'https://ebs.example.com/webservices/rest/<service-alias>/<operation>/' \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer <access-token>' \
  --header 'X-Correlation-ID: P2P-20260822-000001' \
  --data @request.json
```

REST Endpoint、资源路径、Context Header 和 Payload 必须从已部署服务的 WADL/XSD 获得。Token、密码和 Cookie 不写脚本、Git 或 Concurrent Log；Basic Authentication 只通过 HTTPS 使用。

<a id="src-docs-09-technical-interfaces--43-带退避的客户端示例"></a>
#### 4.3 带退避的客户端示例

```bash
#!/usr/bin/env bash
set -euo pipefail

endpoint="$1"
payload_file="$2"
correlation_id="$3"

for attempt in 1 2 3 4 5; do
  http_code="$(curl --silent --show-error \
    --output response.json \
    --write-out '%{http_code}' \
    --request POST \
    --url "$endpoint" \
    --header 'Content-Type: application/json' \
    --header "Authorization: Bearer ${EBS_ACCESS_TOKEN:?}" \
    --header "X-Correlation-ID: $correlation_id" \
    --data "@$payload_file")"

  case "$http_code" in
    200|201|202) exit 0 ;;
    408|429|500|502|503|504) sleep "$((2 ** attempt))" ;;
    *) exit 1 ;;
  esac
done
exit 2
```

客户端只能重试确认幂等的操作。连接超时不代表 EBS 未处理，应先以业务幂等键/Request ID 查询结果；HTTP 4xx 业务/权限错误不应自动重试。

<a id="src-docs-09-technical-interfaces--5-concurrent-program-异步状态-api"></a>
### 5. Concurrent Program 异步状态 API

```sql
SELECT fcr.request_id,
       fcr.parent_request_id,
       fcr.phase_code,
       fcr.status_code,
       fcr.actual_start_date,
       fcr.actual_completion_date,
       fcr.logfile_name,
       fcr.outfile_name,
       fcr.completion_text
  FROM fnd_concurrent_requests fcr
 WHERE fcr.request_id = :p_request_id;
```

外部 REST 接口推荐返回：

```json
{
  "correlationId": "P2P-20260822-000001",
  "requestId": 12345678,
  "status": "SUBMITTED",
  "statusUrl": "/integrations/P2P-20260822-000001"
}
```

状态查询服务将 EBS `PHASE_CODE/STATUS_CODE` 映射为 Submitted/Running/Success/Warning/Error，不直接把内部单字符状态暴露给消费者。

<a id="src-docs-09-technical-interfaces--6-ebs-调用外部-rest"></a>
### 6. EBS 调用外部 REST

Oracle 官方 Service Invocation Framework 使用 Workflow Business Event System；PL/SQL 通过 `WF_EVENT.RAISE` 触发，Java Deferred Agent Listener 调用服务，并可在 Service Invocation Monitor 监控。

```sql
DECLARE
  l_params wf_parameter_list_t := wf_parameter_list_t();
BEGIN
  wf_event.addparametertolist(
    p_name          => 'X-Correlation-ID',
    p_value         => :p_correlation_id,
    p_parameterlist => l_params
  );

  wf_event.raise(
    p_event_name => 'oracle.apps.xxint.rest.invoke',
    p_event_key  => :p_correlation_id,
    p_parameters => l_params,
    p_event_data => :p_json_payload
  );
END;
/
```

需要先定义 Event、REST Invocation Metadata 和带 Java Rule Function 的 Subscription。不要自行在数据库中保存明文密码或使用不受支持的网络 ACL 绕过框架。

<a id="src-docs-09-technical-interfaces--7-可观测性和错误分类"></a>
### 7. 可观测性和错误分类

```sql
SELECT interface_code,
       status_code,
       COUNT(*) message_count,
       MIN(creation_date) oldest_message,
       MAX(retry_count) max_retry_count
  FROM xxint_messages
 WHERE creation_date >= SYSDATE - 1
 GROUP BY interface_code, status_code
 ORDER BY interface_code, status_code;
```

| 错误类型 | 示例 | 自动重试 |
| --- | --- | --- |
| Payload | JSON/XML 格式、必填字段缺失 | 否 |
| Master Data | Supplier/Customer/Item/CCID 无效 | 否，修数后重提 |
| Business Rule | Period Closed、Hold、Credit Limit | 通常否 |
| Authorization | ISG Grant、MOAC、Data Access Set | 否 |
| Technical | Timeout、429、临时网络/DB 资源 | 有上限地重试 |
| Unknown Result | 请求超时但 EBS 可能已成功 | 先查询，不直接重放 |

日志必须包含 Correlation ID、Interface Code、External Key、Request ID、EBS ID 和阶段，但要脱敏银行账户、税号、身份证、Token、密码和完整 Payload。

<a id="src-docs-09-technical-interfaces--8-测试矩阵"></a>
### 8. 测试矩阵

- Contract：WADL/WSDL/XSD/JSON Schema 与客户端版本兼容。
- Functional：正常、缺字段、无效主数据、跨 OU、期间关闭、重复消息。
- Transaction：部分成功、API 回滚、Concurrent Warning/Error、超时结果未知。
- Performance：批量吞吐、Commit Size、并发 Worker、热点唯一键、大表查询计划。
- Security：最小 Grant、无权 OU/Ledger、Token 过期、TLS/证书轮换、日志脱敏。
- Recovery：Worker 中断、并发管理器重启、消息重放、Dead Letter 修复、灾备切换。
- Reconciliation：输入/接口/业务/SLA/GL/ACK 的数量和金额闭环。

<a id="src-docs-09-technical-interfaces--9-关联文档"></a>
### 9. 关联文档

- [开放接口、API 与迁移](#src-docs-09-technical-integration)
- [Concurrent Program](#src-docs-09-technical-concurrent-programs)
- [技术常用表](#src-docs-09-technical-tables)
- [端到端接口编排](09-end-to-end.md#src-docs-08-e2e-interfaces)
- [通用接口治理](01-foundation.md#src-docs-01-common-interfaces)

<a id="src-docs-09-technical-interfaces--10-官方参考"></a>
### 10. 官方参考

- [ISG Implementation Guide: Deploying REST Services](https://docs.oracle.com/cd/E26401_01/doc.122/e20925/T511175T513044.htm)
- [ISG Developer's Guide: Using PL/SQL APIs as Web Services](https://docs.oracle.com/cd/E26401_01/doc.122/e20927/T511473T522190.htm)
- [ISG Implementation Guide: Service Invocation Framework](https://docs.oracle.com/cd/E26401_01/doc.122/e20925/T511175T513090.htm)
- [Oracle E-Business Suite Concepts: Integration Repository](https://docs.oracle.com/cd/E26401_01/doc.122/e22949/T120505T120507.htm)


<!-- source: docs/09-technical/operations.md -->
<a id="src-docs-09-technical-operations"></a>
## 性能调优、权限审计与 R12.2 生产运维


<a id="src-docs-09-technical-operations--r122-运维边界"></a>
### R12.2 运维边界

- 应用层管理节点、WebLogic/OHS、Forms、Concurrent Processing、Workflow Mailer、OPP 和集成服务。
- R12.2 使用 Online Patching（adop）的 Prepare、Apply、Finalize、Cutover、Cleanup 周期，并基于 Run/Patch File System 与 EBR。
- 管理脚本和环境文件必须在正确节点/文件系统执行；不在未确认的环境中混用 run/patch edition。
- 数据库、中间件、EBS Codelevel/ETCC、Java 和浏览器兼容性应按 Oracle 证证矩阵和 Support 建议维护。

<a id="src-docs-09-technical-operations--性能诊断法"></a>
### 性能诊断法

```text
Business Symptom
→ User/Responsibility/Function/Request + Exact Time
→ Tier（Browser/OHS/OAF/Forms/Concurrent/DB/External）
→ Session/Request/SQL ID/Trace
→ Wait/Plan/Rows/Locks/IO/CPU
→ Reproduce → Fix → Regression → Baseline
```

1. 从用户、职责、功能/请求 ID、参数和精确时间段开始，避免“系统很慢”式无边界排查。
2. 使用 OAM/标准 Diagnostics/Trace 有限时间采样；AWR/ASH/SQL Monitor 使用需符合数据库许可。
3. 优先修正 SQL 选择性、Join、Bind、统计信息和数据倾斜，不盲目加 Hint/Index。
4. 对并发程序将性能与数据量、参数、Manager Queue/Processes、Incompatibility 和 OPP 分开分析。

<a id="src-docs-09-technical-operations--安全和审计"></a>
### 安全和审计

- 定期复核用户、职责、失效日期、User-level Profile、特权职责、共享账号和服务账号。
- 实施 SoD：Supplier/Bank Change、Invoice Approval、Payment Creation/Release、Journal Create/Approve/Post、User Admin 分离。
- 保护 APPS/APPLSYS/SYSTEM 和 WebLogic 管理凭据，轮换密码并验证下游集成。
- 日志/报表脱敏银行账号、税号、个人信息和 Token；限制输出保留和下载权限。

<a id="src-docs-09-technical-operations--实用-sql"></a>
### 实用 SQL

```sql
-- 失效对象；不要无差别全库重编译
SELECT owner, object_type, COUNT(*) invalid_count
  FROM dba_objects
 WHERE status = 'INVALID'
 GROUP BY owner, object_type
 ORDER BY owner, object_type;

-- 近期失败请求
SELECT request_id, program_application_id, concurrent_program_id,
       phase_code, status_code, actual_start_date,
       actual_completion_date, argument_text
  FROM fnd_concurrent_requests
 WHERE actual_start_date >= SYSDATE - :p_days
   AND status_code IN ('E','G')
 ORDER BY actual_start_date DESC;

-- 有效用户层 Profile 覆盖
SELECT fu.user_name, fpo.user_profile_option_name,
       fpov.profile_option_value
  FROM fnd_profile_option_values fpov
  JOIN fnd_profile_options_vl fpo
    ON fpo.application_id = fpov.application_id
   AND fpo.profile_option_id = fpov.profile_option_id
  JOIN fnd_user fu ON fu.user_id = fpov.level_value
 WHERE fpov.level_id = 10004
   AND NVL(fu.end_date, SYSDATE + 1) > SYSDATE
 ORDER BY fu.user_name, fpo.user_profile_option_name;
```

> `DBA_OBJECTS` 需 DBA 权限；普通账号使用 `ALL_OBJECTS`。生产性能诊断必须有时间范围，避免高成本全库查询。

<a id="src-docs-09-technical-operations--常见问题"></a>
### 常见问题

- adop 卡住：从当前 Phase/Worker 日志和 adopscanlog 定位首个错误，检查节点、空间、ETCC、无效对象、ADZDPATCH/并发程序，不盲目 `abort/cleanup`。
- Forms/OAF 单点故障：比较用户/职责/功能和节点，查 OHS/WebLogic/Managed Server 日志、会话和近期变更。
- Workflow Mailer 不发信：查 Component Status、Inbound/Outbound Account、SMTP/IMAP/TLS、Notification Mail Status、Deferred/Error Queue 和日志。
- 补丁后性能回退：比较 SQL Plan/Stats/Codelevel、无效对象和定制兼容，使用可重现证据建 SR。

<a id="src-docs-09-technical-operations--关联"></a>
### 关联

- [Security](01-foundation.md#src-docs-01-common-security)
- [Concurrent Processing](#src-docs-09-technical-concurrent-programs)
- [Customization](#src-docs-09-technical-customization)

<a id="src-docs-09-technical-operations--官方参考"></a>
### 官方参考

- [Oracle E-Business Suite Concepts R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e22949/)
- [Oracle E-Business Suite R12.2 Documentation Library](https://docs.oracle.com/cd/E26401_01/index.htm)


<!-- source: docs/09-technical/tables.md -->
<a id="src-docs-09-technical-tables"></a>
## FND、Concurrent、Workflow 与运维常用表结构


<a id="src-docs-09-technical-tables--业务说明"></a>
### 业务说明

FND 是 EBS 应用对象字典、用户/职责、Profile、菜单、并发处理和 Lookup 的共享基础。Workflow 保存 Item/Activity/Notification 状态与延迟队列。运维 SQL 应优先查询状态与日志证据，不通过直接更新 FND/WF 表来“修复”请求或流程。

<a id="src-docs-09-technical-tables--fnd-安全与设置表"></a>
### FND 安全与设置表

| 表/视图 | 中文名 | 粒度/用途 | 关键字段 |
| --- | --- | --- | --- |
| `FND_USER` | EBS 用户 | 每个应用用户 | `USER_ID`, `USER_NAME`, `EMPLOYEE_ID`, `START_DATE/END_DATE` |
| `FND_RESPONSIBILITY` / `_TL` / `_VL` | 职责 | Responsibility+Application+语言 | `RESPONSIBILITY_ID`, `APPLICATION_ID`, `MENU_ID`, `REQUEST_GROUP_ID` |
| `FND_USER_RESP_GROUPS_DIRECT` | 用户直接职责分配 | User+Responsibility+有效期 | `USER_ID`, `RESPONSIBILITY_ID`, `RESPONSIBILITY_APPLICATION_ID` |
| `FND_MENUS` | 菜单 | 每个 Menu | `MENU_ID`, `MENU_NAME` |
| `FND_MENU_ENTRIES` | 菜单条目 | Menu+Sequence | `MENU_ID`, `ENTRY_SEQUENCE`, `SUB_MENU_ID`, `FUNCTION_ID` |
| `FND_FORM_FUNCTIONS` | 应用功能 | 每个 Function | `FUNCTION_ID`, `FUNCTION_NAME`, `FORM_ID`, `PARAMETERS` |
| `FND_PROFILE_OPTIONS` / `_TL` / `_VL` | Profile 定义 | 每个 Profile | `PROFILE_OPTION_ID`, `PROFILE_OPTION_NAME` |
| `FND_PROFILE_OPTION_VALUES` | Profile 设置值 | Profile+设置层级 | `LEVEL_ID`, `LEVEL_VALUE`, `LEVEL_VALUE_APPLICATION_ID` |
| `FND_LOOKUP_VALUES` / `_VL` | Lookup 代码与含义 | Lookup Type+Code+Language | `LOOKUP_TYPE`, `LOOKUP_CODE`, `MEANING` |
| `FND_APPLICATION` / `_TL` / `_VL` | EBS 应用 | 每个 Application | `APPLICATION_ID`, `APPLICATION_SHORT_NAME` |

<a id="src-docs-09-technical-tables--profile-levelid"></a>
#### Profile `LEVEL_ID`

| 值 | 层级 | `LEVEL_VALUE` 含义 |
| --- | --- | --- |
| `10001` | Site | 通常不使用具体业务 ID |
| `10002` | Application | `APPLICATION_ID` |
| `10003` | Responsibility | `RESPONSIBILITY_ID`，并结合 `LEVEL_VALUE_APPLICATION_ID` |
| `10004` | User | `USER_ID` |

最终 Profile 值优先级通常为 User > Responsibility > Application > Site。但 Profile 可更新层级受其定义限制；诊断时应同时查显式设置和 `FND_PROFILE.VALUE` 运行时最终值。

<a id="src-docs-09-technical-tables--concurrent-processing-表"></a>
### Concurrent Processing 表

| 表/视图 | 中文名 | 粒度/用途 | 关键字段 |
| --- | --- | --- | --- |
| `FND_CONCURRENT_PROGRAMS` / `_TL` / `_VL` | 并发程序定义 | Program+Application+语言 | `CONCURRENT_PROGRAM_ID`, `EXECUTABLE_ID`, `OUTPUT_FILE_TYPE` |
| `FND_EXECUTABLES` | 可执行对象 | 每个 Executable | `EXECUTABLE_ID`, `EXECUTION_METHOD_CODE`, `EXECUTION_FILE_NAME` |
| `FND_CONCURRENT_REQUESTS` | 并发请求 | 每次请求 | `REQUEST_ID`, `PHASE_CODE`, `STATUS_CODE`, Program/Application IDs |
| `FND_CONCURRENT_QUEUES` / `_VL` | 并发管理器队列 | 每个 Manager Queue | `CONCURRENT_QUEUE_ID`, `RUNNING_PROCESSES`, `MAX_PROCESSES` |
| `FND_CONCURRENT_PROCESSES` | 并发 Worker 进程 | 每个 Manager Process | `CONCURRENT_PROCESS_ID`, `CONCURRENT_QUEUE_ID`, `PROCESS_STATUS_CODE` |
| `FND_REQUEST_SETS` | 请求集 | 每个 Request Set | `REQUEST_SET_ID`, `REQUEST_SET_NAME` |
| `FND_REQUEST_SET_STAGES` | 请求集阶段 | Set+Stage | `REQUEST_SET_STAGE_ID`, `STAGE_NAME` |
| `FND_RUN_REQUESTS` | 请求子请求关系/打印选项 | Parent Request 执行元数据 | `PARENT_REQUEST_ID` 等 |

<a id="src-docs-09-technical-tables--fndconcurrentrequests"></a>
#### `FND_CONCURRENT_REQUESTS`

| 字段 | 中文名 | 业务说明 |
| --- | --- | --- |
| `REQUEST_ID` | 请求 ID | 用户、日志、DB Session、子请求跟踪的核心键 |
| `PARENT_REQUEST_ID` | 父请求 ID | Request Set/父程序提交子请求时使用；根请求通常为特定默认值/NULL，以 eTRM 为准 |
| `PHASE_CODE` | 请求阶段 | 常见 `P`等待、`R`运行、`C`完成、`I`非活动；必须与 Status 组合解码 |
| `STATUS_CODE` | 请求状态 | Normal/Warning/Error/On Hold/No Manager/Cancelled/Terminated 等，单字母在不同 Phase 下解读，使用 `CP_STATUS_CODE` Lookup |
| `ARGUMENT_TEXT` | 参数文本 | 位置受 Program Parameter 定义决定，不可只凭逗号位置猜业务含义 |
| `ACTUAL_START/COMPLETION_DATE` | 实际开始/完成 | 计算运行时长；等待时长还要用 Requested Start Date |
| `CONCURRENT_PROCESS_ID` | Worker 进程 ID | 关联 Concurrent Process，用于定位 Manager/OS Process |
| `ORACLE_PROCESS_ID` | 数据库进程线索 | 值在 RAC/架构下需结合 Node/Session 使用 |
| `LOGFILE_NAME/OUTFILE_NAME` | 日志/输出路径 | 应通过 EBS 标准页面/授权访问，路径可受节点和迁移影响 |

<a id="src-docs-09-technical-tables--workflow-表"></a>
### Workflow 表

| 表 | 中文名 | 粒度/用途 |
| --- | --- | --- |
| `WF_ITEMS` | Workflow 项实例 | `ITEM_TYPE + ITEM_KEY` 每个流程实例 |
| `WF_ITEM_ACTIVITY_STATUSES` | Workflow 活动状态历史 | Item+Process Activity+执行历史 |
| `WF_ITEM_ACTIVITY_STATUSES_H` | Workflow 已归档活动历史 | 已移入历史的活动状态 |
| `WF_NOTIFICATIONS` | Workflow 通知 | 每条审批/通知 |
| `WF_NOTIFICATION_ATTRIBUTES` | 通知属性 | Notification+Attribute |
| `WF_DEFERRED` | Workflow 延迟队列 | 等待 Background Engine/事件处理 |
| `WF_ERROR` | Workflow 错误队列 | 处理异常消息 |
| `WF_LOCAL_ROLES` / `WF_LOCAL_USER_ROLES` | Workflow 角色/成员 | 用户、职责、审批角色目录映射 |

`WF_ITEM_ACTIVITY_STATUSES.ACTIVITY_STATUS` 常见 Notified、Active、Complete、Error、Deferred、Suspended 等含义，必须结合 Activity Result、Error Name/Message、Begin/End Date 和归档表查询。`WF_NOTIFICATIONS.STATUS/MAIL_STATUS` 分别表示业务通知状态与邮件发送状态，Open 通知不等于邮件必然未发出。

<a id="src-docs-09-technical-tables--状态解码-sql"></a>
### 状态解码 SQL

```sql
SELECT lookup_type, lookup_code, meaning, description
  FROM fnd_lookup_values_vl
 WHERE lookup_type IN ('CP_PHASE_CODE', 'CP_STATUS_CODE')
 ORDER BY lookup_type, lookup_code;

SELECT fcr.request_id,
       phase.meaning phase_name,
       status.meaning status_name,
       fcr.requested_start_date,
       fcr.actual_start_date,
       fcr.actual_completion_date
  FROM fnd_concurrent_requests fcr
  LEFT JOIN fnd_lookup_values_vl phase
    ON phase.lookup_type = 'CP_PHASE_CODE'
   AND phase.lookup_code = fcr.phase_code
  LEFT JOIN fnd_lookup_values_vl status
    ON status.lookup_type = 'CP_STATUS_CODE'
   AND status.lookup_code = fcr.status_code
 WHERE fcr.request_id = :p_request_id;
```

> Lookup 视图可包含多语言/多行有效性，如返回重复，增加 `LANGUAGE = USERENV('LANG')`、启用标志和有效日条件。

<a id="src-docs-09-technical-tables--官方参考"></a>
### 官方参考

- [Oracle E-Business Suite eTRM User's Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/)
- [Oracle E-Business Suite Concepts R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e22949/)


<!-- source: docs/09-technical/workflow-ame-oaf-governance.md -->
<a id="src-docs-09-technical-workflow-ame-oaf-governance"></a>
## Workflow、AME、OAF/Forms 与配置迁移治理


<a id="src-docs-09-technical-workflow-ame-oaf-governance--分工"></a>
### 分工

- Oracle Workflow 负责业务流程、通知、活动、Business Event 与后台引擎处理。
- AME 负责规则化审批人清单和条件；业务单据仍由各产品的审批集成点驱动。
- OAF/Forms Personalization 用于受支持的界面行为调整；复杂定制需要遵守 R12.2 EBR/ADOP、扩展点、安全和回归要求。

<a id="src-docs-09-technical-workflow-ame-oaf-governance--实施与发布顺序"></a>
### 实施与发布顺序

1. 用业务流程图定义事件、状态、审批角色、超时、代理、拒绝/撤回和异常补偿。
2. 将 AME 条件、属性、规则、动作类型和测试样例纳入版本控制；覆盖金额、组织、币种、项目、税务等边界值。
3. 区分 Personalization 与代码扩展：优先使用页面/Forms Personalization；代码仅使用受支持扩展点。
4. 使用 FNDLOAD、WFLOAD、OAF/Forms 发布工件或 Oracle 受支持工具迁移，并在 ADOP 流程中完成多环境回归。

<a id="src-docs-09-technical-workflow-ame-oaf-governance--诊断-sql"></a>
### 诊断 SQL

```sql
-- Workflow 项目状态按业务键收缩；敏感内容和通知正文不应随意导出。
select wi.item_type,
       wi.item_key,
       wi.root_activity,
       wi.begin_date,
       wi.end_date
  from wf_items wi
 where wi.item_type = :p_item_type
   and wi.item_key = :p_item_key;

-- 并发/Workflow 诊断前先确认请求和业务单据的关联，避免仅按用户名全库检索。
select wias.item_type,
       wias.item_key,
       wias.process_activity,
       wias.activity_status,
       wias.begin_date,
       wias.end_date
  from wf_item_activity_statuses wias
 where wias.item_type = :p_item_type
   and wias.item_key = :p_item_key
 order by wias.begin_date;
```

<a id="src-docs-09-technical-workflow-ame-oaf-governance--常见问题"></a>
### 常见问题

- 审批人不正确：先检查交易属性和 AME 条件输入，再检查职责/人员/组织层级；不要直接修改已运行 Workflow 状态表。
- 通知未发：分辨 Workflow 状态、Background Engine、Mailer、邮件通道和外部 SMTP 断点。
- 页面修改在补丁后消失：检查是否使用受支持 Personalization/扩展及其迁移工件，避免运行文件系统临时改动。

<a id="src-docs-09-technical-workflow-ame-oaf-governance--官方参考"></a>
### 官方参考

- [Oracle E-Business Suite Technology Documentation](https://docs.oracle.com/cd/E26401_01/nav/technology.htm)
- [Oracle E-Business Suite Maintenance Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22954/toc.htm)
