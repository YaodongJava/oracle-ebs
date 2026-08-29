# Oracle EBS R12.2 模块数据字典与名词解释

> 本页是各模块的统一速查入口。表、视图、列、状态值和关联关系按 R12.2 常见对象整理；具体补丁、本地化、许可证和客户定制可能改变实现，生产查询前必须用 eTRM、`ALL_TAB_COLUMNS`、Integration Repository 和目标实例日志复核。

## 使用约定

| 约定 | 解释 |
| --- | --- |
| 粒度 | 一行记录代表的业务层级；不要把 Header、Line、Distribution 混为一层 |
| 主键/业务键 | `_ID` 通常是数据库主键；单据号、批次号、来源号是业务键，可能需要组合唯一 |
| 组织字段 | `ORG_ID` 常表示 Operating Unit；库存模块常用 `ORGANIZATION_ID` 表示 Inventory Organization，不能仅按字段名推断 |
| 状态 | 并发请求、接口、业务单据、会计事件、银行处理各有独立状态，必须连同表和程序解释 |
| WHO 列 | `CREATED_BY/CREATION_DATE/LAST_UPDATED_BY/LAST_UPDATE_DATE/LAST_UPDATE_LOGIN` 用于审计追踪，不等于业务状态 |
| 金额 | 同时确认交易币、账簿币、报告币、借贷方向和汇率日期；不要只看一个金额列 |

<a id="dict-00"></a>
## 00 导航与治理：全局对象和术语

### 核心对象/字段

| 对象/字段 | 粒度 | 关键字段或关系 | 用途 |
| --- | --- | --- | --- |
| `GL_LEDGERS` | Ledger | `LEDGER_ID`、`CHART_OF_ACCOUNTS_ID`、`CURRENCY_CODE`、`PERIOD_SET_NAME` | 账簿、科目表、币种和日历边界 |
| `GL_CODE_COMBINATIONS` | 科目组合 | `CODE_COMBINATION_ID`、段值、`ENABLED_FLAG`、有效日期 | 账户组合和启用控制 |
| `FND_USER` | 用户 | `USER_ID`、`USER_NAME`、`START_DATE`、`END_DATE` | 登录身份和有效期 |
| `FND_RESPONSIBILITY_VL` | 职责 | `RESPONSIBILITY_ID`、`APPLICATION_ID`、`RESPONSIBILITY_KEY` | 功能菜单、权限和应用上下文 |
| Profile Option | 用户/职责/站点配置 | Profile Name、Level、Value | 默认组织、语言、行为和安全参数 |
| `FND_CONCURRENT_REQUESTS` | 并发请求 | `REQUEST_ID`、`PHASE_CODE`、`STATUS_CODE` | 请求执行、日志、输出和运行证据 |

### 名词解释

| 名词 | 英文 | 解释 |
| --- | --- | --- |
| EBS | E-Business Suite | Oracle 企业业务应用套件；R12.2 使用在线补丁和 EBR 架构 |
| Ledger | Ledger | 账簿；替代 11i 常说的 Set of Books（账套） |
| OU | Operating Unit | 业务实体/运营单元；许多 AP、AR、PO 交易以 `ORG_ID` 隔离 |
| LE | Legal Entity | 法人实体；承担法律和税务责任，不等同 OU |
| COA | Chart of Accounts | 会计科目表；定义段、值集和科目组合结构 |
| SLA | Subledger Accounting | 子账会计；把业务事件转换成可传 GL 的会计分录 |
| MOAC | Multi-Org Access Control | 多组织访问控制；决定职责可见的 OU/组织范围 |
| TCA | Trading Community Architecture | 贸易社区架构；统一 Party、Customer、Supplier 和地点身份 |

<a id="dict-01"></a>
## 01 财务公共基础：企业结构、权限和共享设置

### 核心对象/字段

| 对象/字段 | 粒度 | 关键字段或关系 | 用途 |
| --- | --- | --- | --- |
| `GL_LEDGERS` | Ledger | `LEDGER_ID`、`NAME`、`CURRENCY_CODE`、`CHART_OF_ACCOUNTS_ID` | 账簿主数据 |
| `GL_PERIOD_STATUSES` | 产品/账簿/期间 | `APPLICATION_ID`、`LEDGER_ID`、`PERIOD_NAME`、`CLOSING_STATUS` | 子账/GL 期间开关 |
| `XLE_ENTITY_PROFILES` | 法人 | `LEGAL_ENTITY_ID`、名称、登记信息 | 法人和法定报告主体 |
| `HR_OPERATING_UNITS` | OU | `ORGANIZATION_ID`、`NAME`、有效日期 | AP/AR/PO 业务边界 |
| `GL_CODE_COMBINATIONS` | 科目组合 | `CODE_COMBINATION_ID`、段值、启用/有效日期 | 会计账户 |
| `FND_USER_RESP_GROUPS_DIRECT` | 用户-职责分配 | `USER_ID`、`RESPONSIBILITY_ID`、`RESPONSIBILITY_APPLICATION_ID`、有效期 | 用户实际职责授权 |
| `FND_PROFILE_OPTION_VALUES` | Profile 值 | Profile、Level、Level Value、Value | 默认组织、并发和安全行为 |

### 名词解释

| 名词 | 解释 |
| --- | --- |
| Enterprise Structure | 企业结构；法人、Ledger、OU、库存组织和安全边界的组合 |
| Data Access Set | GL 数据访问集；控制 Ledger/平衡段等总账数据范围 |
| Security Profile | HR/MO 安全配置；控制组织、人员或 OU 可见范围 |
| KFF | Key Flexfield | 关键弹性域；COA、Item、资产地点等结构化编码 |
| DFF | Descriptive Flexfield | 描述性弹性域；通过上下文和 `ATTRIBUTE*` 扩展业务属性 |
| Configuration Owner | 配置所有者 | EBTax 等共享配置的拥有范围，不等于最终交易法人 |

官方基线：[Financials Concepts](https://docs.oracle.com/cd/E26401_01/doc.122/e48836/toc.htm)、[Multiple Organizations](https://docs.oracle.com/cd/E26401_01/doc.122/e48833/toc.htm)、[EBS Security](https://docs.oracle.com/cd/E26401_01/doc.122/e22952/toc.htm)。

<a id="dict-02"></a>
## 02 记录到报告：GL、SLA 和会计追溯

### 核心对象/字段

| 对象/字段 | 粒度 | 关键字段或关系 | 用途 |
| --- | --- | --- | --- |
| `GL_JE_BATCHES` | 日记账批 | `JE_BATCH_ID`、`STATUS`、`STATUS_VERIFIED` | 批次控制和过账范围 |
| `GL_JE_HEADERS` | 日记账头 | `JE_HEADER_ID`、`LEDGER_ID`、`PERIOD_NAME`、`JE_SOURCE`、`JE_CATEGORY` | 日记账来源、类别和期间 |
| `GL_JE_LINES` | 日记账行 | `JE_HEADER_ID`、`JE_LINE_NUM`、`CODE_COMBINATION_ID`、`ENTERED_DR/CR`、`ACCOUNTED_DR/CR` | 借贷金额和科目 |
| `GL_IMPORT_REFERENCES` | GL 行来源引用 | `JE_HEADER_ID`、`JE_LINE_NUM`、`GL_SL_LINK_ID/TABLE` | GL 与 SLA/来源下钻 |
| `GL_BALANCES` | 科目-期间余额 | Ledger、COA、`CODE_COMBINATION_ID`、`PERIOD_NAME`、借贷余额 | FSG/余额报表基础 |
| `XLA_EVENTS` | 会计事件 | `EVENT_ID`、`ENTITY_ID`、`EVENT_TYPE_CODE`、`EVENT_STATUS_CODE` | 业务事件会计资格 |
| `XLA_AE_HEADERS/LINES` | SLA 分录头/行 | `AE_HEADER_ID`、`AE_LINE_NUM`、`ACCOUNTING_CLASS_CODE`、借贷金额、传输状态 | 子账会计和 GL 传送 |
| `GL_INTERFACE` | GL 导入接口行 | `STATUS`、`GROUP_ID`、`LEDGER_ID`、`ACCOUNTING_DATE`、CCID、借贷金额 | Journal Import 暂存 |

### 名词解释

| 名词 | 解释 |
| --- | --- |
| Journal Batch | 日记账批；一组受控日记账头和行，可统一导入/过账 |
| Journal Header/Line | 日记账头/行；头存来源和期间，行存科目、币种和借贷金额 |
| Posting | 过账；把已批准日记账更新到 GL 余额 |
| Accounting Event | 会计事件；业务动作触发的会计处理单位 |
| Transfer to GL | 传送到总账；SLA 分录传入 GL 接口/导入链 |
| Revaluation | 重估；按期末汇率重算外币余额并生成差额 |
| Translation | 折算；将账簿余额转换为报告币种 |
| Summary Transfer | 汇总传送；以较少明细传入 GL，可能降低逐笔下钻粒度 |

官方基线：[General Ledger User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48748/toc.htm)、[SLA Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48771/title.htm)。

<a id="dict-03"></a>
## 03 采购到付款：PO、接收、AP 和支付

### 核心对象/字段

| 对象/字段 | 粒度 | 关键字段或关系 | 用途 |
| --- | --- | --- | --- |
| `PO_HEADERS_ALL` | 采购订单头 | `PO_HEADER_ID`、`ORG_ID`、供应商、批准状态、币种 | 采购合同/订单 |
| `PO_LINES_ALL` | 采购订单行 | `PO_LINE_ID`、`PO_HEADER_ID`、物料/描述、数量、单价 | 采购明细 |
| `PO_LINE_LOCATIONS_ALL` | 发运计划 | `LINE_LOCATION_ID`、交付日期、接收控制 | 交付和接收计划 |
| `PO_DISTRIBUTIONS_ALL` | 采购分配 | `PO_DISTRIBUTION_ID`、CCID、数量、项目/任务 | 费用/库存/项目分配 |
| `RCV_TRANSACTIONS` | 接收事务 | `TRANSACTION_ID`、`PO_LINE_LOCATION_ID`、事务类型、数量 | Receipt/Delivery/Return |
| `AP_INVOICES_ALL` | AP 发票头 | `INVOICE_ID`、`ORG_ID`、供应商、金额、`WFAPPROVAL_STATUS` | 发票业务头 |
| `AP_INVOICE_DISTRIBUTIONS_ALL` | 发票分配 | `INVOICE_DISTRIBUTION_ID`、`INVOICE_ID`、CCID、金额、匹配信息 | 会计和三单匹配 |
| `AP_CHECKS_ALL` / IBY | 付款/支付指令 | `CHECK_ID`、付款状态、Payment Instruction | 付款执行和银行文件 |

### 名词解释

| 名词 | 解释 |
| --- | --- |
| 2-way/3-way match | 二单/三单匹配；发票与 PO，或发票、PO、接收数量/金额核对 |
| Receipt | 收货；记录供应商交付到 Receiving 的事实 |
| Delivery | 交付；把接收数量送入库存、费用或项目目的地 |
| Accrual | 应计；收货与 AP 发票之间的暂估负债/清算机制 |
| Invoice Validation | 发票验证；匹配、税、期间、账户、Hold 和审批检查 |
| Hold | 挂起；阻止验证、付款或会计的业务控制 |
| PPR | Payment Process Request | 支付流程请求；按模板和选择规则生成付款 |
| IBY | Oracle Payments | 支付引擎，负责指令、格式、传输和回执 |

官方基线：[Payables User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48760/toc.htm)、[Payables Reference](https://docs.oracle.com/cd/E26401_01/doc.122/e48763/h/uaework/tmp/archive/e48763/toc.htm)。

<a id="dict-04"></a>
## 04 信用到收款：TCA、AR、收款和核销

### 核心对象/字段

| 对象/字段 | 粒度 | 关键字段或关系 | 用途 |
| --- | --- | --- | --- |
| `HZ_PARTIES` | Party | `PARTY_ID`、`PARTY_TYPE`、名称 | 客户/供应商共享身份 |
| `HZ_CUST_ACCOUNTS` | Customer Account | `CUST_ACCOUNT_ID`、`PARTY_ID`、账户编号 | 客户商业关系 |
| `HZ_CUST_ACCT_SITES_ALL` | Account Site | `CUST_ACCT_SITE_ID`、地址/用途 | Bill-to/Ship-to 等地点 |
| `RA_CUSTOMER_TRX_ALL` | AR 交易头 | `CUSTOMER_TRX_ID`、`ORG_ID`、交易号、类型、日期 | 发票/贷项/借项 |
| `RA_CUSTOMER_TRX_LINES_ALL` | AR 交易行 | `CUSTOMER_TRX_LINE_ID`、交易头、Line Type、数量/金额 | 交易明细和税/运费行 |
| `AR_PAYMENT_SCHEDULES_ALL` | 付款计划 | `PAYMENT_SCHEDULE_ID`、交易/收款、到期日、余额 | 账龄和未结余额 |
| `AR_CASH_RECEIPTS_ALL` | 收款头 | `CASH_RECEIPT_ID`、金额、币种、`STATUS`、Method | 收款和核销维度 |
| `AR_RECEIVABLE_APPLICATIONS_ALL` | 收款核销行 | `RECEIVABLE_APPLICATION_ID`、收款、交易、`AMOUNT_APPLIED` | Invoice/On-account/Unapplied |
| `AR_CASH_RECEIPT_HISTORY_ALL` | 收款生命周期 | Receipt、`STATUS`、日期 | Confirmed/Remitted/Cleared/Reversed |

### 名词解释

| 名词 | 解释 |
| --- | --- |
| Party | TCA 现实主体；可以拥有 Customer Account 或 Supplier 角色 |
| Customer Account | 客户账户；Party 与企业之间的商业关系层 |
| Site Use | 地点用途；如 Bill-to、Ship-to、Statements、Legal |
| AutoInvoice | 自动开票；从 OM/Projects/外部接口生成 AR 交易 |
| Lockbox | 自动收款箱；导入银行收款并按规则识别/核销 |
| Unapplied | 未核销；已入账但尚未应用到具体交易的收款金额 |
| On-account | 挂账户；归客户账户但暂不指定某张发票 |
| Application | 核销；把收款应用到发票、贷项或其他应收项目 |

官方基线：[Receivables User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f10570/toc.htm)、[TCA Administration Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48940/toc.htm)。

<a id="dict-05"></a>
## 05 资产与项目：FA、Project Costing 和 Billing

### 核心对象/字段

| 对象/字段 | 粒度 | 关键字段或关系 | 用途 |
| --- | --- | --- | --- |
| `FA_ADDITIONS_B` | 资产头 | `ASSET_ID`、资产编号、类别、标签 | 资产身份 |
| `FA_BOOKS` | 资产-账簿 | `ASSET_ID`、`BOOK_TYPE_CODE`、成本、DPIS、折旧方法 | Corporate/Tax 账簿价值 |
| `FA_DISTRIBUTION_HISTORY` | 资产分配历史 | 资产、账簿、地点、员工、CCID、有效日期 | 资产责任和费用归属 |
| `FA_TRANSACTION_HEADERS` | 资产事务 | `TRANSACTION_HEADER_ID`、资产、事务类型/日期 | 增加、调整、转移、退休 |
| `FA_MASS_ADDITIONS` | 资产新增接口 | `MASS_ADDITION_ID`、来源、成本、队列/过账状态 | AP/Projects/外部资产暂存 |
| `PA_PROJECTS_ALL` / `PA_TASKS` | 项目/任务 | `PROJECT_ID`、`TASK_ID`、项目状态、客户/组织 | 项目管理和成本归集 |
| `PA_EXPENDITURE_ITEMS_ALL` | 支出项目 | 支出、项目/任务、类型、日期、数量/成本 | Project Costing 原始支出 |
| `PA_COST_DISTRIBUTION_LINES_ALL` | 成本分配行 | 支出、Raw/Burden 成本、CCID、会计状态 | 项目成本会计 |
| Draft Revenue/Invoice 对象 | 草稿收入/发票 | Project、金额、状态、来源事件/支出 | Project Billing 到 AR |

### 名词解释

| 名词 | 解释 |
| --- | --- |
| Asset Book | 资产账簿；定义折旧日历、方法、币种和会计边界 |
| CIP | Construction in Progress | 在建工程；达到可使用状态前归集的资本项目成本 |
| DPIS | Date Placed in Service | 投入使用日期；影响折旧起算和期间 |
| Mass Additions | 资产新增暂存；须 Prepare/Post 后才成为正式资产 |
| Project Costing | 项目成本；支出验证、Raw/Burden 分配和成本会计 |
| Draft Revenue | 草稿收入；生成、复核、释放后进入收入会计 |
| Draft Invoice | 草稿发票；批准、释放、接口 AR 并 Tieback |
| Tieback | 回写；把 AR AutoInvoice 结果回写 Projects |

官方基线：[Assets User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48755/toc.htm)、[Project Costing](https://docs.oracle.com/cd/E26401_01/doc.122/e48918/toc.htm)、[Project Billing](https://docs.oracle.com/cd/E26401_01/doc.122/e49079/toc.htm)。

<a id="dict-06"></a>
## 06 现金、资金与税务：CE、IBY 和 EBTax

### 核心对象/字段

| 对象/字段 | 粒度 | 关键字段或关系 | 用途 |
| --- | --- | --- | --- |
| `CE_BANK_ACCOUNTS` | 内部银行账户 | `BANK_ACCOUNT_ID`、账户名/币种、所有者组织、有效期 | 现金账户主数据 |
| `CE_STATEMENT_HEADERS` | 银行对账单头 | `STATEMENT_HEADER_ID`、账户、编号、日期、期初/期末余额 | 对账单控制总额 |
| `CE_STATEMENT_LINES` | 银行流水行 | `STATEMENT_LINE_ID`、交易日期、码、金额、状态 | 自动/手工匹配 |
| `CE_STATEMENT_HEADERS_INT` | 对账单头接口 | 编号、账户、日期、控制余额、`ORG_ID`（按场景） | Bank Statement Loader 输入 |
| `CE_STATEMENT_LINES_INTERFACE` | 对账单行接口 | 行号、日期、交易码、金额、银行参考 | 银行文件暂存 |
| IBY Payment Instruction | 支付指令 | Instruction、付款、文件、银行回执状态 | 支付执行链 |
| `ZX_LINES` | 税行 | `APPLICATION_ID`、实体/事件类、`TRX_ID`、税制/税率、税额 | EBTax 结果 |
| Tax configuration | 税务配置 | Regime、Tax、Status、Rate、Jurisdiction、Rule | 税务确定规则 |

### 名词解释

| 名词 | 解释 |
| --- | --- |
| CE | Cash Management | 现金管理；银行账户、对账单、自动对账和现金控制 |
| Bank Statement | 银行对账单；银行提供的账户级流水和余额 |
| AutoReconciliation | 自动对账；按交易码、参考号、金额/日期容差匹配 |
| Cash Clearing | 现金清算；源模块现金/清算账户和银行事实之间的过渡 |
| IBY | Oracle Payments | 支付指令、格式、传输和回执框架 |
| EBTax/ZX | E-Business Tax | 交易税务确定和税行引擎，不等于全部法定申报 |
| Configuration Owner | 配置所有者；控制税务配置适用范围 |
| Recovery | 税额抵扣/不可抵扣规则；影响税额去向和账户 |

官方基线：[Cash Management](https://docs.oracle.com/cd/E26401_01/doc.122/e48900/toc.htm)、[EBTax Implementation](https://docs.oracle.com/cd/E26401_01/doc.122/e48750/toc.htm)、[EBTax User](https://docs.oracle.com/cd/E26401_01/doc.122/e48751/toc.htm)。

<a id="dict-07"></a>
## 07 供应链财务与成本：INV、WIP、CST 和 LCM

### 核心对象/字段

| 对象/字段 | 粒度 | 关键字段或关系 | 用途 |
| --- | --- | --- | --- |
| `MTL_SYSTEM_ITEMS_B` | 物料-组织 | `INVENTORY_ITEM_ID`、`ORGANIZATION_ID`、Item 状态/属性 | 物料组织级定义 |
| `MTL_PARAMETERS` | 库存组织参数 | `ORGANIZATION_ID`、成本方法/组织参数 | INV/CST 边界 |
| `MTL_MATERIAL_TRANSACTIONS` | 物料事务 | `TRANSACTION_ID`、物料、组织、数量、日期、成本状态 | 收发、转移、调整 |
| `MTL_TRANSACTION_ACCOUNTS` | 物料事务账户 | 事务、账户、借贷/价值 | 库存会计分配 |
| `MTL_TRANSACTIONS_INTERFACE` | 物料事务接口 | 接口 ID、事务类型、物料/组织、数量、处理状态 | 外部库存事务输入 |
| `CST_ITEM_COSTS` / Cost Type | 物料成本 | 物料、组织、Cost Type、成本要素、Item Cost | Frozen/Pending 成本 |
| `WIP_ENTITIES` | 工单实体 | `WIP_ENTITY_ID`、工单号、组织、状态 | 离散制造工单 |
| `WIP_TRANSACTIONS` | WIP 事务 | 工单、事务类型、数量、资源/材料 | 发料、资源、完工、关闭 |

### 名词解释

| 名词 | 解释 |
| --- | --- |
| Inventory Organization | 库存组织；库存数量、制造和物料事务边界 |
| Cost Organization/Method | 成本组织/方法；决定标准、平均等成本计算边界 |
| Cost Element | 成本要素；Material、Material Overhead、Resource、OSP、Overhead |
| Frozen/Pending | 冻结/待更新成本版本；Pending 不会自动成为 Frozen |
| PPV/IPV | Purchase/Invoice Price Variance；采购价差/发票价差 |
| WIP | Work in Process | 在制品；材料、资源、间接费和完工/关闭差异 |
| COGS Matching | 销售成本匹配；发运成本与收入确认协同 |
| LCM | Landed Cost Management | 到岸成本管理；运费、关税等附加费用分摊 |

官方基线：[Inventory User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48820/toc.htm)、[Cost Management](https://docs.oracle.com/cd/E26401_01/doc.122/e48829/toc.htm)、[WIP User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48905/toc.htm)。

<a id="dict-08"></a>
## 08 报表、关账与治理

### 核心对象/字段

| 对象/字段 | 粒度 | 关键字段或关系 | 用途 |
| --- | --- | --- | --- |
| `GL_BALANCES` | 账簿-科目-期间余额 | Ledger、CCID、期间、余额类型、借贷余额 | FSG/管理报表基础 |
| `GL_JE_HEADERS/LINES` | 日记账头/行 | 来源、类别、期间、CCID、借贷金额、过账状态 | 报表下钻和控制总额 |
| `XLA_AE_HEADERS/LINES` | SLA 分录 | 事件、会计类、账户、借贷、传送状态 | 子账到 GL 血缘 |
| `FND_CONCURRENT_REQUESTS` | 报表请求 | `REQUEST_ID`、程序、参数、Phase/Status、输出 | 报表执行证据 |
| BI Publisher Data Model/Template | 数据模型/模板 | 数据集、参数、模板版本、输出格式 | 模板报表和分发 |
| ECC Load Definition | 加载定义 | Full/Incremental/Push、计划、索引新鲜度 | 运营看板数据新鲜度 |

### 名词解释

| 名词 | 解释 |
| --- | --- |
| FSG | Financial Statement Generator | 基于 GL 余额和行/列集生成财务报表 |
| Report Manager | EBS 原生报表提交/查看框架；与外部 Office 插件不同 |
| BI Publisher | 模板化报表和批量分发工具 |
| RXi | Report eXchange | 可配置的标准报表输出框架 |
| Smart View | Office 分析插件；需单独部署连接器，不是 EBS 原生 GL 报表引擎 |
| ECC | Enterprise Command Center | 运营搜索和可视化；新鲜度由加载模式/计划决定 |
| Control Total | 控制总额 | 来源、子账、SLA、GL 或报表之间的数量/金额校验 |
| Data Lineage | 数据血缘 | 报表单元格到余额、日记账、SLA、来源交易的追溯链 |

官方基线：[General Ledger User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48748/toc.htm)、[Report Manager](https://docs.oracle.com/cd/E26401_01/doc.122/e22006/toc.htm)、[ECC Administrator](https://docs.oracle.com/cd/E26401_01/doc.122/f34732/toc.htm)。

<a id="dict-09"></a>
## 09 端到端流程与跨模块关联

### 关键键值链

| 链路 | 关键键 | 解释 |
| --- | --- | --- |
| PO→RCV→AP | `PO_HEADER_ID/LINE_ID` → `RCV_TRANSACTION_ID` → `INVOICE_ID`/分配 ID | 采购、接收和发票匹配链 |
| AR→Receipt | `CUSTOMER_TRX_ID` → `PAYMENT_SCHEDULE_ID` → `CASH_RECEIPT_ID` → Application ID | 应收、到期和核销链 |
| 子账→SLA→GL | 来源交易键 → `XLA_EVENT_ID/AE_HEADER_ID` → `JE_HEADER_ID/LINE_NUM` | 业务到会计和总账下钻 |
| Project→FA | `PROJECT_ID/TASK_ID` → 支出/资产行 → `ASSET_ID`/`BOOK_TYPE_CODE` | 项目成本到资本化 |
| 银行→对账 | `BANK_ACCOUNT_ID` → `STATEMENT_HEADER_ID/LINE_ID` → 源交易键 | 银行事实和账面现金匹配 |

### 名词解释

| 名词 | 解释 |
| --- | --- |
| Source System Key | 来源系统业务键；接口幂等、回执和追溯的起点 |
| EBS Surrogate Key | EBS 内部 ID；如 `*_ID`，不能替代来源业务键 |
| Interface Batch | 接口批次；控制文件/请求/重跑和控制总额 |
| Request ID | 并发请求 ID；定位程序参数、日志和输出 |
| Event ID | 会计事件 ID；连接源交易和 SLA |
| Reconciliation | 对账；在统一截止时点比较数量、金额、状态和账户 |
| Compensation | 补偿处理；用标准冲销、贷项或调整修复已成功交易 |

官方基线：[EBS Concepts](https://docs.oracle.com/cd/E26401_01/doc.122/e22949/toc.htm)、[SLA Implementation](https://docs.oracle.com/cd/E26401_01/doc.122/e48771/title.htm)。

<a id="dict-10"></a>
## 10 技术架构、开发与集成

### 核心对象/字段

| 对象/字段 | 粒度 | 关键字段或关系 | 用途 |
| --- | --- | --- | --- |
| `FND_CONCURRENT_REQUESTS` | 请求 | `REQUEST_ID`、程序、参数、`PHASE_CODE`、`STATUS_CODE` | 并发生命周期 |
| `FND_CONCURRENT_PROGRAMS` | 并发程序定义 | 程序、应用、执行方法、启用状态 | 程序元数据 |
| `FND_LOG_MESSAGES` | 应用日志 | 模块、级别、时间、消息 | 诊断和审计证据 |
| `WF_ITEMS` | Workflow Item | `ITEM_TYPE`、`ITEM_KEY`、用户键、状态 | 工作流实例 |
| `WF_ITEM_ACTIVITY_STATUSES` | Workflow 活动状态 | Item、活动、状态、结果、开始/结束时间 | 节点排错 |
| `AD_ZD_SESSIONS` / EBR 对象 | 在线补丁会话 | Session、Edition、节点/状态 | ADOP/EBR 维护追踪 |
| Integration Repository | API/服务元数据 | Internal Name、方法、方向、版本、WADL | ISG/API 契约 |

### 名词解释

| 名词 | 解释 |
| --- | --- |
| APPS | EBS 应用访问层；不应作为终端用户长期直连授权边界 |
| Custom Schema | 自定义数据库 Schema；按 Developer's Guide 注册并受控授权 |
| APPS_INITIALIZE | 初始化用户、职责和应用上下文的公开入口 |
| EBR | Edition-Based Redefinition | R12.2 在线补丁依赖的版本化数据库机制 |
| ADOP | Online Patching Utility | R12.2 在线补丁生命周期工具 |
| ISG | Integrated SOA Gateway | EBS API、并发程序和开放接口的服务发布框架 |
| OAF/Forms | Oracle Application Framework / Oracle Forms | EBS 页面技术栈 |
| Workflow | 工作流 | Item、活动、通知和结果组成的业务流程引擎 |

官方基线：[EBS Developer's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22961/toc.htm)、[ISG Implementation](https://docs.oracle.com/cd/E26401_01/doc.122/e20925/toc.htm)、[ISG Developer](https://docs.oracle.com/cd/E26401_01/doc.122/e20927/toc.htm)。

<a id="dict-11"></a>
## 11 实施与运维生命周期

### 交付数据对象

| 对象 | 粒度 | 最低字段/状态 | 用途 |
| --- | --- | --- | --- |
| Requirement | 需求 | ID、范围、优先级、验收标准、负责人 | 需求追踪 |
| Configuration Workbook | 配置项 | 模块、字段、值、依赖、迁移方式、复核人 | 配置基线 |
| CEMLI Object | 定制对象 | 类型、对象名、版本、部署路径、回退 | 定制开发治理 |
| Interface Contract | 接口契约 | 来源键、字段、必填、状态、幂等、错误、控制总额 | 集成交付 |
| Test Case/Defect | 测试/缺陷 | 前置、步骤、预期、实际、严重度、证据、决定 | SIT/UAT/回归 |
| Cutover Task | 切换任务 | 依赖、窗口、执行/验证人、输入/输出、回退点 | 上线 Runbook |
| Runbook/Evidence | 运行手册/证据 | 请求 ID、参数、日志、报表、对账、签核 | BAU/审计 |

### 名词解释

| 名词 | 解释 |
| --- | --- |
| CRP | Conference Room Pilot | 方案演示和早期流程验证 |
| SIT | System Integration Test | 系统集成测试，覆盖跨模块接口和会计链 |
| UAT | User Acceptance Test | 业务用户验收测试 |
| CEMLI | Configuration、Extension、Modification、Localization、Integration | EBS 定制分类 |
| Cutover | 切换 | 冻结、迁移、增量、验证、Go/No-Go 和业务开放 |
| Hypercare | 上线强化支持 | 上线后高频监控、缺陷分诊和每日对账阶段 |
| RCA | Root Cause Analysis | 根因分析；区分触发、根因、修复和预防 |
| RPO/RTO | Recovery Point/Time Objective | 灾备可接受数据丢失点/恢复时长 |

官方基线：[EBS Concepts](https://docs.oracle.com/cd/E26401_01/doc.122/e22949/toc.htm)、[Maintenance Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22954/toc.htm)、[EBS Developer's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22961/toc.htm)。

## 维护规则

1. 新增字段前记录产品、补丁、对象所有者、粒度、主键、状态和官方来源。
2. 任何列名或状态值若没有 eTRM/实例证据，标记为“待目标实例复核”，不得编写无界生产 SQL。
3. 业务状态、接口状态、会计状态和并发状态分栏维护；不要用一个 `STATUS` 代替四类状态。
4. 统一参考见 [`90-reference.md`](90-reference.md)；模块原有深度字典见各模块文档的“常用表结构”章节。
