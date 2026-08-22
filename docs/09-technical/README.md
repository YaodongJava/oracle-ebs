# EBS R12.2 技术、集成与运维

本目录覆盖 R12.2 技术开发和生产运行的公共规范。业务模块接口文档描述产品入口；本目录定义接口选型、并发程序、PL/SQL、Workflow、OAF/Forms、迁移、EBR/ADOP、性能、安全和可观测性的共性边界。

## 专题导航

- [开放接口、API、报表与迁移](integration.md)
- [技术接口实现手册](interfaces.md)
- [数据模型与元数据](data-model.md)
- [Concurrent Program](concurrent-programs.md)
- [PL/SQL、Forms、Personalization 与 OAF 定制](customization.md)
- [性能、权限审计与生产运维](operations.md)
- [R12.2 ADOP、EBR 与发布治理](adop-ebr-release.md)
- [Workflow、AME、OAF/Forms 与迁移治理](workflow-ame-oaf-governance.md)
- [FND、Concurrent、Workflow 表](tables.md)

## R12.2 不可省略的边界

1. 定制对象和部署必须遵循 Edition-Based Redefinition 与 Online Patching（ADOP）约束；不得以覆盖 Oracle 标准文件或直接修改标准对象作为常规交付方式。
2. 支持写入的路径依次为标准页面、公开 API、Open Interface、Integration Repository/ISG 和客户自定义对象；禁止直接 DML EBS 业务基表修复数据。
3. 接口应具备幂等键、状态机、错误分类、审计相关号、最小权限、监控、重试上限和人工补偿入口。
4. 性能问题先以并发请求、日志、SQL 执行计划和应用上下文定位；AWR、ASH、SQL Monitor 等能力须确认数据库许可证。

## 官方依据

- [Oracle E-Business Suite Technology Documentation](https://docs.oracle.com/cd/E26401_01/nav/technology.htm)
- [Oracle E-Business Suite Maintenance Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22954/toc.htm)
