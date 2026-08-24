# Oracle EBS R12.2.x 技术架构、开发与集成

本目录面向财务技术顾问、集成顾问、应用 DBA、测试和生产运维人员。它维护跨产品的技术原则；AP、AR、GL、FA、CE、CST 等具体接口表、业务规则和会计验证仍以对应产品目录为权威入口。

## 技术认知地图

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

## 专题导航

| 专题 | 中文说明 | 应掌握内容 | 现有详文 |
| --- | --- | --- | --- |
| [architecture](architecture/README.md) | 总体架构 | 请求链路、节点、OHS、WebLogic、Forms、Concurrent、数据库 | [旧版技术总览](../09-technical/README.md) |
| [data-model](data-model/README.md) | 数据模型 | APPS/产品 Schema、同义词、EBR View、WHO、共享模型 | [数据模型详文](../09-technical/data-model.md) |
| [database-development](database-development/README.md) | 数据库开发 | PL/SQL、Context、FND API、事务、锁、批处理和日志 | [定制开发](../09-technical/customization.md) |
| [integration](integration/README.md) | 系统集成 | Open Interface、Public API、ISG、事件、文件、幂等和对账 | [技术接口手册](../09-technical/interfaces.md) |
| [concurrent-processing](concurrent-processing/README.md) | 并发处理 | Program、Request Set、Manager、队列、OPP 和诊断 | [并发程序详文](../09-technical/concurrent-programs.md) |
| [workflow-and-ame](workflow-and-ame/README.md) | 工作流与审批 | Item/Activity、Notification、Mailer、AME Rule、重试和清理 | [Workflow/AME 治理](../09-technical/workflow-ame-oaf-governance.md) |
| [oaf](oaf/README.md) | Oracle 应用框架 | MVC、EO/VO/AM/CO、Personalization、Extension 和调试 | [定制开发](../09-technical/customization.md) |
| [forms](forms/README.md) | Oracle Forms | Template、PLL、Personalization、编译、部署和跟踪 | [定制开发](../09-technical/customization.md) |
| [reporting-development](reporting-development/README.md) | 报表开发 | BI Publisher、eText、Bursting、FSG、Web ADI、OPP | [报表接口](../04-gl/reporting-interfaces.md) |
| [configuration-migration](configuration-migration/README.md) | 配置迁移 | FNDLOAD、WFLOAD、XDOLoader、依赖、环境差异和验证 | [迁移治理](../09-technical/workflow-ame-oaf-governance.md) |
| [adop-and-ebr](adop-and-ebr/README.md) | 在线补丁与 EBR | Run/Patch、Edition、ADOP 周期、同步、恢复和边界 | [ADOP/EBR 详文](../09-technical/adop-ebr-release.md) |
| [database-platform](database-platform/README.md) | 数据库平台 | Schema、存储、统计、锁、备份、RAC/Data Guard 边界 | [运维详文](../09-technical/operations.md) |
| [middleware](middleware/README.md) | 中间件 | OHS、WebLogic、Node Manager、JVM、TLS、Wallet 和日志 | [运维详文](../09-technical/operations.md) |
| [performance](performance/README.md) | 性能 | 请求、SQL、锁、执行计划、JVM、OPP、容量和许可证 | [运维详文](../09-technical/operations.md) |
| [security](security/README.md) | 技术安全 | 用户/职责、RBAC、MOAC、TLS、SSO、DMZ、最小权限 | [财务安全](../01-common/security.md) |
| [audit-and-compliance](audit-and-compliance/README.md) | 审计与合规 | AuditTrail、Sign-On、WHO、SoD、保留和脱敏 | [生产安全边界](../00-guide/safety-and-production-boundaries.md) |
| [testing-and-automation](testing-and-automation/README.md) | 测试与自动化 | 接口契约、会计断言、回归、性能、安全和测试数据 | [角色学习路径](../00-guide/reading-paths-by-role.md) |
| [release-engineering](release-engineering/README.md) | 发布工程 | Git、评审、依赖、Patch Driver、晋级、回退和证据 | [ADOP/EBR 详文](../09-technical/adop-ebr-release.md) |

## 技术顾问的最低能力

### 业务与数据

- 能说明一笔 AP/AR/FA/INV 交易的业务状态、组织、账簿、期间和币种。
- 能从业务主键追溯 `XLA_TRANSACTION_ENTITIES`、`XLA_EVENTS`、`XLA_AE_HEADERS/LINES`，并在保留导入引用时追溯 GL Journal。
- 能解释 `_ALL`、`_B`、`_TL`、`_VL`、Interface 和临时对象的常见含义，同时用目标实例验证而非仅凭命名猜测。
- 能用只读、绑定变量和范围受限的 SQL 收集证据，不把直接 DML 基表当作修复。

### 集成与批处理

- 能按业务量、实时性和支持边界选择 Open Interface、Public API、ISG、Business Event、Concurrent 或文件接口。
- 能设计业务唯一键、批次、相关号、幂等、重试上限、人工补偿和数量/金额对账。
- 能区分 HTTP/API/文件/请求“技术成功”和 EBS 业务对象、会计、银行/税务回执“业务成功”。
- 能从请求号定位参数、日志、父子请求、Manager、数据库会话和最终业务结果。

### R12.2 发布与运行

- 理解 Run/Patch File System、Run/Patch Edition、`fs_ne`、AutoConfig 和 Context File。
- 能说明 `adop` Prepare、Apply、Finalize、Cutover、Cleanup 的目的和验证点。
- 自定义数据库对象、文件和配置迁移具有 EBR 合规说明、版本、依赖、安装/卸载和回退步骤。
- 性能和安全诊断遵守 AWR/ASH/SQL Monitor 等许可证边界及敏感数据最小暴露原则。

## 标准排障框架

```text
1. 明确实例/版本/时间/用户/职责/组织/账簿/期间
2. 确认业务主键、当前状态、预期状态和最近变更
3. 定位接口批次、并发请求、Workflow Item 或 Web 请求
4. 读取首个有意义错误并确认可否安全重试
5. 验证业务结果、SLA、GL、报表和外部回执
6. 使用标准流程/API/接口或批准的 Support 方案恢复
7. 保存前后数量金额对账、日志和审批证据
```

## 设计评审清单

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

## 安全边界

1. 禁止直接 DML Oracle EBS 业务、会计、FND 或 Workflow 运行时基表作为常规集成/修复。
2. 禁止覆盖 Oracle 标准文件或在 APPS Schema 中无治理地创建对象。
3. 禁止在未确认上下文、事务和公开性时调用来源不明的 Package。
4. 禁止在生产执行无范围限制的大表查询、Debug 或 Trace。
5. 任何数据修复都应优先使用标准业务反向流程；确需 Support 数据修复时保留 SR、批准、备份、演练和前后对账。

## 官方资料

- [Oracle E-Business Suite Technology Documentation](https://docs.oracle.com/cd/E26401_01/nav/technology.htm)
- [Oracle E-Business Suite Concepts](https://docs.oracle.com/cd/E26401_01/doc.122/e22949/toc.htm)
- [Oracle E-Business Suite Maintenance Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22954/toc.htm)
- [Oracle Integrated SOA Gateway Developer's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e20927/toc.htm)
- [Oracle E-Business Suite Electronic Technical Reference Manual User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/)
