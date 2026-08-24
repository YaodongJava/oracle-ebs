# Oracle 官方资料与验证顺序

本页说明“查什么资料、资料能证明什么、还需要怎样验证”。官方文档说明产品设计和标准行为，但无法自动证明目标实例的补丁、配置、权限和客户定制与文档完全一致。

## 证据优先级

| 层级 | 证据 | 能回答的问题 | 限制 |
| --- | --- | --- | --- |
| 1 | Oracle R12.2 Concepts/Implementation/User/Technical Guide | 标准产品概念、配置和操作意图是什么？ | 可能覆盖整个 R12.2 系列，不等同于目标补丁行为 |
| 2 | Release Notes、Readme、认证和 MOS 文档 | 某版本/补丁有哪些变化、缺陷和前置条件？ | MOS 需要客户授权；内容可能针对特定组合 |
| 3 | 目标实例元数据 | 产品是否安装？对象、列、API、菜单和参数是否存在？ | 存在不代表已许可、已配置或业务可用 |
| 4 | 目标实例配置与运行日志 | 当前 Ledger、OU、Profile、请求和服务如何配置/运行？ | 只能证明当前快照，需要记录时间和环境 |
| 5 | 非生产场景测试 | 正常、异常、冲销、重跑、会计和报表是否符合预期？ | 需记录与生产的版本/配置/数据量差异 |
| 6 | 生产审批与验证 | 是否允许在生产执行，执行后结果是否正确？ | 仅对批准范围和当前版本有效 |

## Oracle R12.2 文档总入口

- [Oracle E-Business Suite Release 12.2 Documentation Library](https://docs.oracle.com/cd/E26401_01/index.htm)：全部产品文档的首选入口。
- [Current Booklist](https://docs.oracle.com/cd/E26401_01/index.htm)：从总库的 Current Booklist 查找当前书名和格式。
- [Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)：财务概念、GL、AP、AR、FA、CE、SLA、FAH、AGIS、Payments、税务等。
- [Projects Documentation](https://docs.oracle.com/cd/E26401_01/nav/projects.htm)：Projects Foundation、Costing、Billing、Project to Asset 等。
- [Procurement Documentation](https://docs.oracle.com/cd/E26401_01/nav/procurement.htm)：Purchasing、iProcurement、iSupplier 等。
- [Supply Chain Management Documentation](https://docs.oracle.com/cd/E26401_01/nav/scm.htm)：Inventory、WIP、Costing、OM、Shipping 和制造相关产品。
- [Technology Documentation](https://docs.oracle.com/cd/E26401_01/nav/technology.htm)：架构、安全、开发、集成、维护、升级和系统管理。

## 财务顾问优先资料

| 资料 | 重点用途 |
| --- | --- |
| [Oracle Financials Concepts Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48836/toc.htm) | 企业结构、Ledger、Financials 共享概念和跨产品关系 |
| [Oracle Financials Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48783/toc.htm) | 公共财务设置、账簿、银行、税务等实施主题入口 |
| [Oracle Subledger Accounting Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48771/title.htm) | SLA 会计事件、AMB、账户派生、会计方法和 GL 传输 |
| [Oracle E-Business Suite Multiple Organizations Implementation Guide](https://docs.oracle.com/cd/E26401_01/index.htm) | 多组织、OU、安全配置和 MOAC；从总库按书名进入 |
| [Oracle Trading Community Architecture Guides](https://docs.oracle.com/cd/E26401_01/index.htm) | Party、Customer/Supplier 共享身份、关系、接口和数据质量 |
| [Financials 产品页](https://docs.oracle.com/cd/E26401_01/nav/financials.htm) | 查找 AP、AR、GL、FA、CE、Payments、EBTax、AGIS、FAH 的 User/Implementation Guide |

阅读顺序：Concepts Guide（概念）→ Implementation Guide（配置）→ User Guide（交易/操作）→ Technical/Reference Guide（接口/数据）→ Release Notes/MOS（版本差异）→ 实例验证。

## 技术顾问优先资料

| 资料 | 重点用途 |
| --- | --- |
| [Oracle E-Business Suite Concepts](https://docs.oracle.com/cd/E26401_01/doc.122/e22949/toc.htm) | R12.2 架构、应用层、数据库层和组件概念 |
| [Oracle E-Business Suite Setup Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22953/toc.htm) | 用户、职责、并发、Profile、审计等应用基础设置 |
| [Oracle E-Business Suite Security Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22952/toc.htm) | 身份、授权、安全配置、网络和应用安全原则 |
| [Oracle E-Business Suite Maintenance Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22954/toc.htm) | ADOP、维护、补丁和 R12.2 文件系统操作 |
| [Integrated SOA Gateway Developer's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e20927/toc.htm) | Integration Repository、服务部署、REST/SOAP 和安全 |
| [Electronic Technical Reference Manual User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/) | 使用 eTRM 查表、视图、依赖和技术元数据 |
| [Technology Documentation](https://docs.oracle.com/cd/E26401_01/nav/technology.htm) | OAF、Forms、Workflow、Concurrent、BI Publisher、安装升级等书目入口 |

## 目标实例的权威入口

### Integration Repository（集成信息库）

用于确认接口是否公开、所属产品、方法/参数、服务是否生成/部署及授权状态。网上博客或旧项目代码可作为线索，不能替代当前实例 Integration Repository 和 Package Specification（包规范）。

### eTRM 与数据库数据字典

eTRM 用于理解标准对象与依赖；`ALL_OBJECTS`、`ALL_TAB_COLUMNS`、`ALL_CONSTRAINTS`、`ALL_INDEXES` 和 Package Source/Specification 用于确认当前实例实物。数据字典证明对象结构，不证明直接写表是受支持的。

### 应用元数据与页面

通过有权职责确认菜单、Function、Concurrent Program、Request Group、Profile、Lookup、Value Set、Workflow、AME、Ledger、OU 和产品选项。页面标签可能因语言或 Personalization（个性化）不同，排错时记录内部名称/代码和显示名称。

### 日志和请求

保存 Concurrent Request ID、父子请求、参数、日志/输出、Workflow Item Type/Key、OAF/Forms 请求时间、节点和相关业务键。优先分析第一个有意义错误及其上游原因。

## My Oracle Support 使用边界

MOS 用于查询认证矩阵、补丁 Readme、已知问题、数据修复和提交 Service Request（SR，服务请求）。使用规则：

- 记录文档/补丁/SR 标识和适用版本，不在公开知识库复制受限全文。
- 数据修复步骤只在授权客户、适用实例和批准变更中使用。
- 执行前核对前置补丁、备份、停机、对象版本、数据条件和回退说明。
- 执行后保存日志、受影响记录、业务/SLA/GL 前后对账和 Oracle 建议结果。
- 不把一次性 Datafix（数据修复）改造成日常接口或通用脚本。

## 从资料到结论的验证模板

| 项目 | 记录内容 |
| --- | --- |
| 问题/结论 | 要证明的业务或技术命题 |
| 文档依据 | 书名、章节、URL、Release、访问日期 |
| 版本适用性 | EBS、AD/TXK、数据库、产品补丁、平台 |
| 实例证据 | 对象、配置、请求号、日志、查询或截图 |
| 测试场景 | 前置条件、步骤、输入、预期和实际结果 |
| 会计/对账 | 数量、交易币/本位币、SLA、GL 和报表结果 |
| 限制 | 未覆盖组织、币种、异常、数据量或可选产品 |
| 结论等级 | 文档推断、实例确认、场景验证或生产批准 |

## 常见错误

- 只看搜索摘要，不打开对应 Release 的完整官方章节。
- 将 Fusion Cloud、R12.1 或其他客户版本的内容混入 R12.2 结论。
- 用表存在证明产品已安装、许可并启用。
- 用 eTRM 字段清单代替目标实例列、索引和数据分布验证。
- 看到并发请求 Completed/Normal 就省略业务状态、SLA/GL 和报表检查。
- 未记录文档 Release、访问日期、环境和验证限制，导致结论无法复现。
