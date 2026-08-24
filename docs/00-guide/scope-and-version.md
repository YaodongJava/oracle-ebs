# 范围、版本与适用性

## 文档适用范围

本知识库默认面向 Oracle E-Business Suite Release 12.2.x，重点是 Financials（财务）、Projects（项目）、Procurement（采购）、Supply Chain Financials（供应链财务）、通用技术开发和实施运维。以下内容不应在未验证时直接套用：

- Release 11i、R12.0、R12.1 或 Oracle Fusion Cloud ERP 的功能与数据模型。
- 其他客户实例的表列、API 签名、Lookup 状态、Profile 值和并发程序参数。
- My Oracle Support（MOS）中针对特定补丁、平台或数据状态的修复步骤。
- 依赖单独许可证、国家本地化、数据库选件或第三方组件的能力。

文档中的“R12.2.x”表示系列范围，并不保证每个小版本、RUP、AD/TXK 组合都具有完全相同的行为。

## 必须记录的版本基线

### 应用与技术栈

| 项目 | 应记录内容 | 为什么重要 |
| --- | --- | --- |
| EBS Release | 例如 R12.2.x 及当前 Release Update Pack | 菜单、功能、对象和缺陷修复随版本变化 |
| AD/TXK | Delta、代码级别、关键 one-off | 决定 ADOP、AutoConfig、WebLogic 等技术行为 |
| 产品补丁 | AP、AR、GL、FA、CE、ZX、XLA 等相关补丁 | 财务处理和报表结果可能由产品补丁改变 |
| Database | 数据库版本、RU/RUR、兼容性参数 | 影响支持矩阵、性能、EBR 和安全修补 |
| Middleware | WebLogic、OHS、Java 等版本 | 影响应用服务、TLS、客户端和补丁兼容性 |
| OS/平台 | 操作系统、节点、负载均衡和共享文件系统 | 影响路径、脚本、HA 和运维步骤 |

### 业务配置基线

| 项目 | 应记录内容 |
| --- | --- |
| 企业结构 | Business Group、Legal Entity、Ledger、OU、Inventory Organization |
| 核算基础 | COA、日历、币种、会计方法、Secondary Ledger/Reporting Currency |
| 数据访问 | User、Responsibility、Security Profile、Data Access Set、Role/Grant |
| 产品范围 | 已安装产品、共享/完全安装状态、许可证和国家本地化 |
| 外部依赖 | 银行、税务、支付网关、SOA/OIC、数据仓库、SSO、邮件和打印 |
| CEMLI | 配置、扩展、修改、本地化、接口及其版本和所有者 |

## 适用性标签

编写或评审专题时，可使用以下说明：

| 标签 | 含义 |
| --- | --- |
| 核心 | 典型 R12.2 财务实例普遍适用，但仍需目标实例验证 |
| 可选产品 | 需确认许可证和安装，如 FAH、Treasury、Loans、Advanced Collections |
| 本地化 | 仅对特定国家/地区或法定要求适用 |
| 补丁相关 | 行为依赖 RUP、AD/TXK、产品或 one-off 补丁 |
| 数据库选件 | 可能涉及额外数据库许可证，如部分诊断/调优能力 |
| 客户定制 | 只对特定客户 CEMLI 适用，不能冒充标准功能 |

## 如何验证目标实例

### 1. 从产品意图开始

阅读相同 Release 12.2 文档库中的 Concepts Guide（概念指南）、Implementation Guide（实施指南）、User Guide（用户指南）、Technical/Reference Guide（技术/参考指南）和 Release Notes（发行说明）。

### 2. 核对实例元数据

- Installed Products（已安装产品）和应用版本信息。
- Integration Repository（集成信息库）中的公开接口与服务状态。
- eTRM（Electronic Technical Reference Manual，电子技术参考手册）及数据库数据字典。
- 目标责任的菜单、请求组、Profile、Lookup、Value Set 和安全配置。
- Context File（上下文文件）、AutoConfig、节点和服务拓扑。

列定义检查示例：

```sql
select owner,
       table_name,
       column_id,
       column_name,
       data_type,
       data_length,
       nullable
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

### 3. 在非生产环境验证行为

使用与生产相同或明确记录差异的补丁/配置，执行正常、异常、冲销、跨期、外币、多组织、批量和重跑场景。保存业务主键、请求号、日志、会计、报表和对账证据。

### 4. 形成结论等级

| 等级 | 含义 |
| --- | --- |
| 文档推断 | 仅根据官方或项目文档推断，尚未在实例验证 |
| 实例确认 | 已确认对象/配置存在，但未完成业务测试 |
| 场景验证 | 已在指定环境和版本用明确场景验证 |
| 生产批准 | 已完成变更审批、测试、回退和上线授权；不代表永久适用于未来补丁 |

## 变更后的最低回归

升级、CPU（Critical Patch Update，关键补丁更新）、RUP、AD/TXK 变更、数据库 RU、证书/加密变更或重要 CEMLI 发布后，按影响范围验证：

1. 登录、职责、MOAC 和 Data Access Set。
2. AP、AR、GL、FA、CE、CST 等关键代表性交易。
3. Create Accounting、Transfer to GL、Journal Import 和 Posting。
4. 核心并发程序、Workflow/AME、OPP 和通知。
5. 银行、税务、外围系统、文件和 REST/SOAP 接口。
6. BI Publisher/FSG/法定报表、字体、打印和分发。
7. 月结关键请求的性能与批处理窗口。
8. 权限、审计、TLS、SSO 和敏感数据控制。

## 常见错误

- 看到表或菜单就推断产品已许可且可用。
- 用另一个实例的 API 样例替代当前 Package Specification（包规范）验证。
- 混用 R12.1 与 R12.2 的文件系统、部署和补丁步骤。
- 只记录 EBS 小版本，不记录 AD/TXK、数据库和产品补丁。
- 把“测试环境通过”表述为所有组织、账簿、币种和交易类型都适用。
- 将已过时的 MOS 建议或一次性数据修复固化为日常运维脚本。

## 官方依据

- [Oracle E-Business Suite Release 12.2 Documentation Library](https://docs.oracle.com/cd/E26401_01/index.htm)
- [Oracle E-Business Suite Concepts](https://docs.oracle.com/cd/E26401_01/doc.122/e22949/toc.htm)
- [Oracle E-Business Suite Maintenance Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22954/toc.htm)
- [Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
