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
| KFF | Key Flexfield；关键弹性域；COA、Item、资产地点等结构化编码 |
| DFF | Descriptive Flexfield；描述性弹性域；通过上下文和 `ATTRIBUTE*` 扩展业务属性 |
| Configuration Owner | 配置所有者；EBTax 等共享配置的拥有范围，不等于最终交易法人 |

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
| Source / Accounting Attribute | SLA 来源/会计属性 | 交易、事件、币种、主体、汇率、反转、多期间等 | 规则条件、账户、描述和行属性输入 |
| Event Class / Event Type | 事件类/事件类型 | 业务事件分组和具体动作 | 决定适用的 AAD/JLD/JLT |
| Journal Line Type（JLT） | 日记账行类型 | Side、Accounting Class、Balance Type、条件、合并、转 GL | 决定生成借/贷/损益行的行为 |
| Account Derivation Rule（ADR） | 账户推导规则 | 完整账户、段/限定段或值集、优先级 | 生成目标 Accounting Flexfield |
| Mapping Set | 映射集 | Source 输入到账户、段或值的映射 | ADR 的条件化账户映射 |
| Journal Lines Definition（JLD） | 日记账行定义 | JLT、ADR、JED、Supporting Reference 的组合 | 为事件类/类型组成完整分录 |
| Application Accounting Definition（AAD） | 应用会计定义 | 事件类/类型、头描述、JLD、Create Accounting、验证状态 | 规定应用如何生成 SLA |
| Subledger Accounting Method（SAM） | 子分类账会计方法 | 多应用 AAD 的会计政策集合 | 分配到 Ledger 形成会计表示 |

### 名词解释

| 名词 | 解释 |
| --- | --- |
| Journal Batch | 日记账批；一组受控日记账头和行，可统一导入/过账 |
| Journal Header/Line | 日记账头/行；头存来源和期间，行存科目、币种和借贷金额 |
| Journal Source / Category | GL 日记账来源/类别；分别表示来源系统和业务性质，不等同 SLA Accounting Class |
| Posting | 过账；把已批准日记账更新到 GL 余额 |
| Accounting Event | 会计事件；业务动作触发的会计处理单位 |
| Transfer to GL | 传送到总账；SLA 分录传入 GL 接口/导入链 |
| Source | SLA 规则可读取的交易/事件属性；必须在目标 Event Class 上可用 |
| Accounting Attribute | 参与金额、币种、主体、反转、多期间或对账等特殊处理的属性 |
| Event Class / Event Type | 事件类/事件类型；前者按会计模型分组，后者表示具体业务动作 |
| Accounting Class | SLA 行的业务语义标签，如 Expense、Liability、Revenue、Tax、Cash |
| Balance Type | JLT 的余额类型；Actual、Budget 或 Encumbrance，需结合产品和启用范围确认 |
| Side | JLT 的行方向；Debit、Credit 或 Gain/Loss |
| Transfer Level | SLA 向 GL 传送的粒度；Detail 保留明细，Summary 按账户组合汇总 |
| Post-Accounting Program | 按 Accounting Class 选择 SLA 行供后续产品处理的定义，例如资产 Mass Additions |
| Journal Line Type（JLT） | 定义行的借/贷/损益方向、余额类型、条件、合并和转 GL 粒度 |
| Account Derivation Rule（ADR） | 按 Source、Mapping Set、Constant 或其他 ADR 推导完整账户、段或值集 |
| Mapping Set | 将一个或多个输入 Source 值映射到账户、段或值集输出的可维护映射 |
| Journal Entry Description（JED） | 生成 SLA 日记账头和行描述的规则，可引用 Source 和常量 |
| Journal Lines Definition（JLD） | 将 JLT、ADR、JED 和 Supporting Reference 组合成事件级行集合 |
| Application Accounting Definition（AAD） | 为应用 Event Class/Type 分配 JLD、头描述和支持性参考，并控制是否 Create Accounting |
| Subledger Accounting Method（SAM） | 将多个应用的 AAD 组合成共同会计政策，再分配给 Ledger |
| Supporting Reference | 用于按项目、客户、资产或外部批次建立 SLA 余额和对账维度的可选参考 |
| Business Flow Method | None、Same Entry 或 Prior Entry；控制关联交易之间的账户/属性继承 |
| Multiperiod Accounting | 将递延/预付等金额按起止日期和 GL 期间生成 Accrual/Recognition 分录 |
| Draft / Final Accounting | Draft 用于检查规则且不可传 GL；Final 是可传送、过账的正式 SLA 分录 |
| Accounting COA | SLA 生成分录所依据的 Accounting Flexfield 结构；Secondary Ledger 可与交易 COA 不同 |
| Gain/Loss Side | JLT 的汇兑损益方向；由 SLA 根据相关交易汇率差计算，不是普通负数行 |
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
| Purchase Order Type | 采购文档类型 | Standard、Planned、Blanket、Contract | 决定订单是否直接执行、是否需要 Release 及已知信息范围 |
| Blanket/Scheduled Release | 协议/计划释放 | `PO_RELEASE_ID`、来源协议/计划 PO、释放号、交付计划 | 将框架价格或计划总量转成实际可收货订单 |
| `RCV_TRANSACTIONS` | 接收事务 | `TRANSACTION_ID`、`PO_LINE_LOCATION_ID`、事务类型、数量 | Receipt/Delivery/Return |
| Receipt Routing | 收货路由 | Standard、Inspection Required、Direct Delivery | 决定 Receive、Inspect、Deliver 的顺序和库存生效时点 |
| `AP_INVOICES_ALL` | AP 发票头 | `INVOICE_ID`、`ORG_ID`、供应商、金额、`WFAPPROVAL_STATUS` | 发票业务头 |
| `AP_INVOICE_DISTRIBUTIONS_ALL` | 发票分配 | `INVOICE_DISTRIBUTION_ID`、`INVOICE_ID`、CCID、金额、匹配信息 | 会计和三单匹配 |
| Invoice Type | 发票类型 | Standard、Mixed、Credit Memo、Debit Memo、Prepayment、Expense Report 等 | 决定发票正负方向、匹配和核销方式；以实例 Lookup 为准 |
| Invoice Line Type | 发票行类型 | ITEM、TAX、FREIGHT、MISCELLANEOUS、PREPAY/AWT | 区分成本、税、运费、杂项和预扣/预付核销分配 |
| Invoice Hold | 发票挂起 | `AP_HOLDS_ALL`、Hold Code、Reason、Release | 阻止验证、付款或会计的控制结果 |
| Match Option | 匹配选项 | 2-way、3-way、4-way、PO/Receipt/Inspection 引用 | 规定发票验证时必须存在的采购事实 |
| `AP_CHECKS_ALL` / IBY | 付款/支付指令 | `CHECK_ID`、付款状态、Payment Instruction | 付款执行和银行文件 |

### 名词解释

| 名词 | 解释 |
| --- | --- |
| 2-way/3-way match | 二单/三单匹配；发票与 PO，或发票、PO、接收数量/金额核对 |
| 4-way match | 四单匹配；在 PO、发票和接收基础上核对 Inspection/Acceptance 数量或状态 |
| Standard Purchase Order | 标准采购订单；商品/服务、价格、数量、交付计划已知，批准后可直接收货和开票 |
| Planned Purchase Order | 计划采购订单；预计商品/服务和总量已知，具体交付计划通过 Scheduled Release 细化 |
| Blanket Purchase Agreement | 总括采购协议；价格和条款已谈妥，数量/交付按需通过 Blanket Release 消耗 |
| Contract Purchase Agreement | 合同采购协议；先锁定条款，商品/服务和金额未定，后续引用合同创建实际 Standard PO |
| Blanket Release | 总括协议释放；将 Blanket Agreement 的一部分价格/数量/交付转成可执行订单 |
| Scheduled Release | 计划订单释放；把 Planned PO 的预计量细化为具体日期、地点、数量和分配 |
| Receipt | 收货；记录供应商交付到 Receiving 的事实 |
| Delivery | 交付；把接收数量送入库存、费用或项目目的地 |
| Receipt Routing | 收货路由；Standard、Inspection Required、Direct Delivery 三种 Receive/Inspect/Deliver 顺序 |
| Accrual | 应计；收货与 AP 发票之间的暂估负债/清算机制 |
| Invoice Validation | 发票验证；匹配、税、期间、账户、Hold 和审批检查 |
| Hold | 挂起；阻止验证、付款或会计的业务控制 |
| Final Match | 最终匹配；声明 PO 分配不再接受后续发票，启用承诺会计时可冲回未用承诺 |
| Prepayment | 预付款；货物/服务交付前支付给供应商或员工的发票，Temporary 可核销、Permanent 不可核销 |
| PO Price Adjustment | 采购价格调整；追溯价格变化与原发票之间的差额单据，可正可负 |
| Credit Memo / Debit Memo | 贷项/借项通知单；均可为负数，前者由供应商开具，后者由买方记录应减少的供应商负债 |
| Expense Report | 费用报销发票；员工差旅、里程、招待或公司卡费用经审批后导入 AP |
| Invoice Workbench / Quick Invoices | 发票工作台/快速发票；前者适合复杂在线处理，后者适合大量简单记录 |
| PPR | Payment Process Request；支付流程请求；按模板和选择规则生成付款 |
| IBY | Oracle Payments；支付引擎，负责指令、格式、传输和回执 |

官方基线：[Purchasing User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48931/toc.htm)、[Payables User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48760/toc.htm)、[Payables Reference](https://docs.oracle.com/cd/E26401_01/doc.122/e48763/h/uaework/tmp/archive/e48763/toc.htm)。

<a id="dict-04"></a>
## 04 信用到收款：TCA、AR、收款和核销

### 核心对象/字段

| 对象/字段 | 粒度 | 关键字段或关系 | 用途 |
| --- | --- | --- | --- |
| `HZ_PARTIES` | Party | `PARTY_ID`、`PARTY_TYPE`、名称 | 客户/供应商共享身份 |
| `HZ_CUST_ACCOUNTS` | Customer Account | `CUST_ACCOUNT_ID`、`PARTY_ID`、账户编号 | 客户商业关系 |
| `HZ_CUST_ACCT_SITES_ALL` | Account Site | `CUST_ACCT_SITE_ID`、地址/用途 | Bill-to/Ship-to 等地点 |
| `HZ_CUST_SITE_USES_ALL` | Site Use | `SITE_USE_ID`、`SITE_USE_CODE`、`ORG_ID`、主用途 | BILL_TO、SHIP_TO、STATEMENT、DUN 等交易用途 |
| Customer Profile / Credit Profile | 客户/信用档案 | 付款条件、额度、风险类别、AutoCash、Collector、催收标志 | 在账户或地点层默认应收和信用策略 |
| `RA_CUSTOMER_TRX_ALL` | AR 交易头 | `CUSTOMER_TRX_ID`、`ORG_ID`、交易号、类型、日期 | 发票/贷项/借项 |
| `RA_CUSTOMER_TRX_LINES_ALL` | AR 交易行 | `CUSTOMER_TRX_LINE_ID`、交易头、Line Type、数量/金额 | 交易明细和税/运费行 |
| `RA_CUST_TRX_LINE_GL_DIST_ALL` | 交易 GL 分配 | `CUST_TRX_LINE_GL_DIST_ID`、`ACCOUNT_CLASS`、CCID、金额 | Receivable、Revenue、Tax、Freight、Unearned/Unbilled |
| `AR_PAYMENT_SCHEDULES_ALL` | 付款计划 | `PAYMENT_SCHEDULE_ID`、交易/收款、到期日、余额 | 账龄和未结余额 |
| `AR_CASH_RECEIPTS_ALL` | 收款头 | `CASH_RECEIPT_ID`、金额、币种、`STATUS`、Method | 收款和核销维度 |
| `AR_RECEIVABLE_APPLICATIONS_ALL` | 收款核销行 | `RECEIVABLE_APPLICATION_ID`、收款、交易、`AMOUNT_APPLIED` | Invoice/On-account/Unapplied |
| `AR_CASH_RECEIPT_HISTORY_ALL` | 收款生命周期 | Receipt、`STATUS`、日期 | Confirmed/Remitted/Cleared/Reversed |
| `AR_ADJUSTMENTS_ALL` | 应收调整 | `ADJUSTMENT_ID`、交易、类型、金额、状态、Activity | 差异、坏账、手续费和核销调整 |
| `RA_INTERFACE_LINES_ALL` / `RA_INTERFACE_ERRORS_ALL` | AutoInvoice 接口/错误 | 外部键、Source、金额、`INTERFACE_LINE_ID`、错误消息 | 外部交易导入、验证和拒绝回写 |
| `AR_PAYMENTS_INTERFACE_ALL` | Lockbox 接口 | Transmission、记录类型、客户/发票引用、金额、状态 | 银行收款导入和 QuickCash |

### 名词解释

| 名词 | 解释 |
| --- | --- |
| Party | TCA 现实主体；可以拥有 Customer Account 或 Supplier 角色 |
| Customer Account | 客户账户；Party 与企业之间的商业关系层 |
| Site Use | 地点用途；如 Bill-to、Ship-to、Statements、Legal |
| Customer Profile Class | 客户档案类别；向账户/地点默认付款条件、信用、账龄、催收和 AutoCash 规则 |
| Credit Profile | 信用档案；额度、风险类别、信用检查和逾期策略的输入 |
| Transaction Source | 交易来源；控制手工/导入交易编号、批次和 AutoInvoice 验证 |
| Transaction Type | 交易类型；定义 Invoice、Credit Memo、Debit Memo 等类别及 Open Receivable、应用和会计行为 |
| Payment Term | 付款条件；根据交易日期、到期日、折扣日和分期规则生成 Payment Schedule |
| AutoAccounting | AR 账户默认引擎；按交易类型、来源、Memo Line、Salesperson 等派生 Receivable/Revenue/Tax 账户 |
| AutoInvoice | 自动开票；从 OM/Projects/外部接口生成 AR 交易 |
| Lockbox | 自动收款箱；导入银行收款并按规则识别/核销 |
| Receipt Class / Receipt Method | 收款分类/方法；定义确认、汇款、清算步骤以及银行账户、编号和会计活动 |
| AutoCash Rule Set | 自动核销规则集；规定按发票、到期日、折扣、余额或客户参考应用收款的顺序 |
| Application Rule Set | 核销规则集；规定收款在交易行、税、运费和滞纳金之间的分配顺序 |
| Unapplied | 未核销；已入账但尚未应用到具体交易的收款金额 |
| Unidentified | 未识别收款；尚未确定付款客户，不能直接应用到发票 |
| On-account | 挂账户；归客户账户但暂不指定某张发票 |
| Chargeback | 扣款转借项；原发票减少，同时创建新的 Chargeback 应收项目 |
| Deduction / Dispute | 扣款/争议；客户短款或异议的原因、责任、证据和结案流程 |
| Credit Memo | 贷项通知单；减少客户应收，可关联原交易、行、税或运费 |
| Refund | 退款；将客户贷项或多收款退回，需经过余额、审批、付款和清算控制 |
| Dunning / Collections Strategy | 催收信函/策略；按账龄、风险和承诺付款生成催收行动 |
| Application | 核销；把收款应用到发票、贷项或其他应收项目 |

官方基线：[Receivables User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f10570/toc.htm)、[Receivables Reference Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f10312/toc.htm)、[TCA Administration Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48940/toc.htm)、[Credit Management User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48901/toc.htm)。

<a id="dict-05"></a>
## 05 资产与项目：FA、Project Costing 和 Billing

### 核心对象/字段

| 对象/字段 | 粒度 | 关键字段或关系 | 用途 |
| --- | --- | --- | --- |
| `FA_ADDITIONS_B` | 资产头 | `ASSET_ID`、资产编号、类别、标签 | 资产身份 |
| `FA_ADDITIONS_TL` | 资产多语言描述 | `ASSET_ID`、`LANGUAGE`、Description | 多语言资产名称和说明 |
| `FA_BOOK_CONTROLS` | 资产账簿控制 | `BOOK_TYPE_CODE`、会计账簿、折旧/期间日历 | Book 期间和控制参数 |
| `FA_BOOKS` | 资产-账簿 | `ASSET_ID`、`BOOK_TYPE_CODE`、成本、DPIS、折旧方法 | Corporate/Tax 账簿价值 |
| `FA_CATEGORIES_B` / `FA_CATEGORY_BOOKS` | 资产类别/类别账簿 | `CATEGORY_ID`、类别段、成本/折旧/CIP 账户 | 类别默认规则和账户 |
| `FA_DISTRIBUTION_HISTORY` | 资产分配历史 | 资产、账簿、地点、员工、CCID、有效日期 | 资产责任和费用归属 |
| `FA_TRANSACTION_HEADERS` | 资产事务 | `TRANSACTION_HEADER_ID`、资产、事务类型/日期 | 增加、调整、转移、退休 |
| `FA_DEPRN_PERIODS` | FA 折旧期间 | `BOOK_TYPE_CODE`、`PERIOD_COUNTER`、开/关日期 | Book 折旧期间状态 |
| `FA_DEPRN_SUMMARY` / `FA_DEPRN_DETAIL` | 折旧汇总/分配明细 | Asset、Book、Period、Distribution、折旧金额 | 折旧费用和累计折旧 |
| `FA_RETIREMENTS` | 资产退休 | `RETIREMENT_ID`、资产、日期、成本/单位、状态 | 全额/部分处置和恢复 |
| `FA_MASS_ADDITIONS` | 资产新增接口 | `MASS_ADDITION_ID`、来源、成本、队列/过账状态 | AP/Projects/外部资产暂存 |
| `PA_PROJECTS_ALL` / `PA_TASKS` | 项目/任务 | `PROJECT_ID`、`TASK_ID`、项目状态、客户/组织 | 项目管理和成本归集 |
| `PA_EXPENDITURE_ITEMS_ALL` | 支出项目 | 支出、项目/任务、类型、日期、数量/成本 | Project Costing 原始支出 |
| `PA_COST_DISTRIBUTION_LINES_ALL` | 成本分配行 | 支出、Raw/Burden 成本、CCID、会计状态 | 项目成本会计 |
| `PA_PROJECT_ASSET_LINES_ALL` | 项目资产线 | Project/Task、Asset、CIP/RWIP、成本、接口状态 | 资本项目成本到 FA 的汇总/明细边界；以目标实例 eTRM 核对 |
| `PA_EVENTS` | 项目事件 | Project/Task、事件类型、日期、金额、Billable/Revenue 标志 | 里程碑、进度、预付款、贷项等独立计费依据 |
| Draft Revenue / Revenue Distribution 对象 | 草稿收入及分配 | Project、Event/Expenditure、Revenue、状态、会计期间 | 收入生成、释放和 SLA |
| Draft Invoice / Invoice Distribution 对象 | 草稿发票及分配 | Project、Customer、Bill Through、金额、状态 | 项目开票、释放和 AR 接口 |
| Project Transaction Interface | 项目交易接口 | 外部键、Project/Task、Type、Org、日期、数量、状态 | 外部成本/库存/时间导入；具体表以 eTRM 核对 |

### 名词解释

| 名词 | 解释 |
| --- | --- |
| Asset Book | 资产账簿；定义折旧日历、方法、币种和会计边界 |
| Corporate Book | 公司账簿；企业财务报表使用的主要 FA 账簿，一项资产通常只能有一本 Corporate Book |
| Tax Book | 税务账簿；按税法或其他法定口径保存独立折旧和价值，可通过 Mass Copy 复制允许的交易 |
| Asset Category | 资产类别；资产 KFF 组合，在每个 Book 维护默认账户和折旧规则 |
| Asset Type | 资产类型；常见 Capitalized、CIP、Expensed、Group，具体值以 FA Lookup/启用功能为准 |
| Prorate Convention | 比例分摊惯例；定义 DPIS/退休日期在首期、末期或日折旧中的起算方式 |
| CIP | Construction in Progress；在建工程；达到可使用状态前归集的资本项目成本 |
| RWIP | Retirement Work in Progress；退休调整在建成本，如拆除、移除和处置准备成本 |
| DPIS | Date Placed in Service；投入使用日期；影响折旧起算和期间 |
| Mass Additions | 资产新增暂存；须 Prepare/Post 后才成为正式资产 |
| Mass Additions Queue | 资产批量增加队列；按来源/处理状态组织待审、挂起、错误和可过账行 |
| Asset Assignment | 资产分配；把单位分到地点、员工和费用 CCID，影响责任和折旧费用归属 |
| Cost Adjustment | 成本调整；对已入账资产增加或冲减成本，可选择当期费用或剩余寿命摊销 |
| Retirement / Reinstatement | 退休/恢复；分别记录处置和撤销错误退休的标准事务 |
| Gain/Loss on Retirement | 资产处置损益；基于退休成本、转出累计折旧、处置收入和移除成本计算 |
| Physical Inventory | 实物盘点；用标签、地点、员工和状态核对账面资产与实物差异 |
| Project Costing | 项目成本；支出验证、Raw/Burden 分配和成本会计 |
| Project Type | 项目类型；决定项目类别和成本、负担、资本化、收入/开票默认 |
| Expenditure Item | 支出项目；项目实际工作或成本的最小可追溯交易单元 |
| Expenditure Organization | 支出组织；承担或执行项目交易的组织，影响成本率、跨组织和会计 |
| Transaction Control | 交易控制；按人员、日期、资源、支出类型和任务限制项目可接受交易 |
| Raw Cost | 原始成本；直接成本，例如数量乘成本率 |
| Burden Cost | 负担成本；按 Burden Structure/Schedule 对 Raw Cost 计算的间接成本 |
| Burdened Cost | 负担后成本；Raw Cost + Burden Cost |
| Burden Structure | 负担结构；定义成本基础、负担代码及组件层级 |
| Burden Schedule | 负担费率表；按组织、期间或项目类型提供有效 multiplier |
| Commitment | 承诺；采购申请、采购订单等预期成本，不等同已发生支出 |
| Capital Project | 资本项目；归集 CIP/RWIP 并生成资产线、传送 FA 的项目类型 |
| Asset Grouping Level | 资产分组级别；WBS 中决定成本汇总到项目、任务或共同成本的层级 |
| Asset Line | 项目资产线；将资本化成本按资产、分组和 CIP/RWIP 账户汇总后传送 FA |
| Generate Asset Lines | 生成资产线；只选取满足资本化资格、日期和 Final Accounting 条件的成本 |
| Interface Assets | 传送资产线；将有效项目资产线创建为 FA Mass Addition |
| Capitalization Date | 资本化/投产日期；决定成本转固和折旧起算，必须与验收/可使用事实一致 |
| Agreement / Funding | 协议/资金；合同项目的客户合同、额度、有效期、优先级和预收款来源 |
| Event | 项目事件；独立于支出项的里程碑、进度、预收、贷项或交付计费依据 |
| Revenue Method | 收入确认方法；例如 Time and Materials、Percent Complete、Cost Plus 或 Fixed Price 规则 |
| Billing Method | 开票方法；决定可收费成本/事件如何生成项目发票 |
| Draft Revenue | 草稿收入；生成、复核、释放后进入收入会计 |
| Draft Invoice | 草稿发票；批准、释放、接口 AR 并 Tieback |
| UBR / UER | Unbilled Receivable / Unearned Revenue；收入与发票不同步时使用的未开票应收/递延收入 |
| Bill Through Date | 截止计费日期；收入或发票生成选择交易和事件的上限日期 |
| Revenue Release / Invoice Release | 收入/发票释放；将草稿变成可会计或可接口的正式状态 |
| AutoInvoice | AR 自动开票；接收 Projects 发票接口并创建 Receivables 交易 |
| Tieback | 回写；把 AR AutoInvoice 结果回写 Projects |
| Project Billing Currency | 项目计费币种；需区分项目币、项目功能币、资金币和发票交易币 |
| Invoice Reduction / Credit Invoice | 发票减少/贷项发票；释放后更正项目发票的标准审计方式 |

官方基线：[Assets User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48755/toc.htm)、[Project Costing](https://docs.oracle.com/cd/E26401_01/doc.122/e48918/toc.htm)、[Project Billing](https://docs.oracle.com/cd/E26401_01/doc.122/e49079/toc.htm)。

<a id="dict-06"></a>
## 06 现金、资金与税务：CE、IBY 和 EBTax

### 核心对象/字段

| 对象/字段 | 粒度 | 关键字段或关系 | 用途 |
| --- | --- | --- | --- |
| `CE_BANK_ACCOUNTS` | 内部银行账户 | `BANK_ACCOUNT_ID`、账户名/币种、所有者组织、有效期 | 现金账户主数据 |
| `CE_BANK_ACCT_USES_ALL` | 银行账户用途 | `BANK_ACCT_USE_ID`、`BANK_ACCOUNT_ID`、`ORG_ID`、应用用途 | AP/AR/Payroll/Treasury/CE 授权边界 |
| `CE_STATEMENT_HEADERS` | 银行对账单头 | `STATEMENT_HEADER_ID`、账户、编号、日期、期初/期末余额 | 对账单控制总额 |
| `CE_STATEMENT_LINES` | 银行流水行 | `STATEMENT_LINE_ID`、交易日期、码、金额、状态 | 自动/手工匹配 |
| `CE_STATEMENT_HEADERS_INT` | 对账单头接口 | 编号、账户、日期、控制余额、`ORG_ID`（按场景） | Bank Statement Loader 输入 |
| `CE_STATEMENT_LINES_INTERFACE` | 对账单行接口 | 行号、日期、交易码、金额、银行参考 | 银行文件暂存 |
| Bank Statement Open Interface | 银行对账单开放接口 | Header/Line 接口行、状态、控制总额 | 银行文件加载与校验 |
| Reconciliation Open Interface | 对账开放接口 | 外部交易号、账户、日期、币种、金额、状态 | Treasury/外部系统交易参与 CE 对账 |
| `IBY_PAY_SERVICE_REQUESTS` | Payment Process Request | `PAYMENT_SERVICE_REQUEST_ID`、调用应用外部键、状态 | PPR 选择、分组和付款流程 |
| `IBY_PAY_INSTRUCTIONS_ALL` | 付款指令 | `PAYMENT_INSTRUCTION_ID`、PPR、格式、状态 | 付款文件和传输分组 |
| `IBY_PAYMENTS_ALL` | IBY 付款 | `PAYMENT_ID`、付款指令、状态 | 单笔付款生命周期 |
| `IBY_EXT_BANK_ACCOUNTS` | 外部银行账户 | `EXT_BANK_ACCOUNT_ID`、收款人/国家/银行信息 | 供应商、客户或员工收款账户 |
| IBY Payment Instruction | 支付指令 | Instruction、付款、文件、银行回执状态 | 支付执行链 |
| `ZX_LINES` | 税行 | `APPLICATION_ID`、实体/事件类、`TRX_ID`、税制/税率、税额 | EBTax 结果 |
| `ZX_LINES_DET_FACTORS` | 税务确定因素 | Entity/Event/Transaction/Line 组合及输入快照 | 解释税务规则命中结果 |
| `ZX_RATES_B` | 税率 | Regime、Tax、Status、Rate、有效期 | 百分比/数量税率主数据 |
| `ZX_PARTY_TAX_PROFILE` / `ZX_REGISTRATIONS` | 交易方税档案/登记 | Party、法人、税制/辖区、登记号、有效期 | 注册、免税和税务报告 |
| Tax configuration | 税务配置 | Regime、Tax、Status、Rate、Jurisdiction、Rule | 税务确定规则 |

### 名词解释

| 名词 | 解释 |
| --- | --- |
| CE | Cash Management；现金管理；银行账户、对账单、自动对账和现金控制 |
| Bank Statement | 银行对账单；银行提供的账户级流水和余额 |
| Bank Account Use | 银行账户用途；按应用和 OU 授权 AP、AR、Payroll、Treasury 或 CE 使用内部账户 |
| Statement Header / Line | 对账单头/行；头保存账户、编号、日期和控制总额，行保存每笔银行流水 |
| Bank Transaction Code | 银行交易代码；把银行格式中的付款、收款、费用、利息等映射为 CE 交易类型 |
| Value Date | 价值日；银行资金实际起息/可用日期，可与交易日、会计日不同 |
| AutoReconciliation | 自动对账；按交易码、参考号、金额/日期容差匹配 |
| Manual Reconciliation | 手工对账；用户选择源交易或建立银行来源杂项行并记录原因 |
| Matched / Cleared / Reconciled | 匹配/清算/对账；分别表示找到源交易、更新清算状态、完成银行对账确认 |
| Unmatched / Multiple Match | 未匹配/多重匹配；前者无候选，后者候选不唯一，不应自动过账 |
| Miscellaneous Transaction | 杂项交易；银行手续费、利息等无 AP/AR 源单据的银行行，需批准的 Activity/账户 |
| Reconciliation Open Interface | 对账开放接口；让 Treasury 或外部系统交易出现在 CE 可匹配交易中 |
| Cash Clearing | 现金清算；源模块现金/清算账户和银行事实之间的过渡 |
| IBY | Oracle Payments；支付指令、格式、传输和回执框架 |
| PPR | Payment Process Request；付款流程请求，按模板和选择规则挑选发票并分组付款 |
| Payment Instruction | 付款指令；把 Proposed Payments 按银行/格式/账户生成待传输文件 |
| Technical ACK / Business ACK | 技术回执/业务回执；分别表示文件可达与银行受理/拒绝，不等同结算 |
| Lockbox | 自动收款箱；导入银行收款批次并按客户/发票/参考号核销 |
| Cash Position | 现金头寸；按已确认/已清算事实计算当前可用现金 |
| Cash Forecast | 现金预测；按来源和时间桶估计未来现金，不等同银行或 GL 实际余额 |
| Cash Forecast Template | 现金预测模板；定义来源、日期、时间桶、币种、状态过滤和重复消除 |
| Sweep / Cash Pool | 归集/资金池；在多个账户间按规则转移现金，需分别核对两端结算与对账 |
| EBTax/ZX | E-Business Tax；交易税务确定和税行引擎，不等于全部法定申报 |
| Configuration Owner | 配置所有者；控制税务配置适用范围 |
| Tax Regime | 税制；按国家/地区或业务建立 Tax、Status、Rate、Jurisdiction 的容器 |
| Tax | 税种；VAT/GST/Sales Tax 等税务类别 |
| Tax Status | 税状态；同一 Tax 下的 Standard、Zero、Exempt 等状态 |
| Tax Rate | 税率；按百分比或数量费率及有效期定义 Tax Amount 计算输入 |
| Tax Jurisdiction | 税务辖区；由 Place of Supply 决定，可维护辖区级税率 |
| Tax Determining Factors | 税务确定因素；交易日期、Party/Registration、产品分类、地点、类型等输入 |
| Party Tax Profile / Registration | 交易方税档案/登记；法人、客户、供应商及地点的税号和税务属性 |
| Fiscal Classification | 财政/税务分类；产品、交易方或交易的税务分类码 |
| Place of Supply | 供应地；税务判定交易发生地的规则结果 |
| Taxable Basis / Tax Formula | 计税基础/税务公式；决定 Taxable Amount 和特殊计算逻辑 |
| Recovery | 税额抵扣/不可抵扣规则；影响税额去向和账户 |
| Recoverable / Nonrecoverable Tax | 可抵扣/不可抵扣税；按 Recovery Rate 分配到抵扣账户或成本 |
| Recovery Rate | 抵扣率；税法允许从进项税中抵扣的百分比，可按规则或默认值取得 |
| Offset Tax | 反向计税/自计税抵销税；为原税负债创建匹配的抵销税行和会计 |
| Inclusive / Compound Tax | 含税/复合税；分别影响计税基础和多税种叠加顺序 |
| Tax-only Line | 仅税行；记录与采购/销售行分离的税额，须符合产品规则 |
| Tax Reporting Type | 税务报告类型；将税行分类到法定提取或申报口径 |

官方基线：[Cash Management](https://docs.oracle.com/cd/E26401_01/doc.122/e48900/toc.htm)、[Cash Management 自动对账与多币种](https://docs.oracle.com/cd/E26401_01/doc.122/e48900/T359831T359837.htm)、[EBTax Implementation](https://docs.oracle.com/cd/E26401_01/doc.122/e48750/toc.htm)、[EBTax User](https://docs.oracle.com/cd/E26401_01/doc.122/e48751/toc.htm)、[Oracle Payments/Payables PPR](https://docs.oracle.com/cd/E26401_01/doc.122/e48760/T295436T369088.htm)。

<a id="dict-07"></a>
## 07 供应链财务与成本：INV、WIP、CST 和 LCM

### 核心对象/字段

| 对象/字段 | 粒度 | 关键字段或关系 | 用途 |
| --- | --- | --- | --- |
| `MTL_SYSTEM_ITEMS_B` | 物料-组织 | `INVENTORY_ITEM_ID`、`ORGANIZATION_ID`、Item 状态/属性 | 物料组织级定义 |
| `MTL_PARAMETERS` | 库存组织参数 | `ORGANIZATION_ID`、成本方法/组织参数 | INV/CST 边界 |
| `MTL_ONHAND_QUANTITIES_DETAIL` | 现有量明细 | `ONHAND_QUANTITIES_ID`、物料/组织/子库存/货位/批次、主单位数量 | 当前现有量、成本层或收货层对账 |
| `MTL_MATERIAL_TRANSACTIONS` | 物料事务 | `TRANSACTION_ID`、物料、组织、数量、日期、成本状态 | 收发、转移、调整 |
| `MTL_TRANSACTION_ACCOUNTS` | 物料事务账户 | 事务、账户、借贷/价值 | 库存会计分配 |
| `MTL_TRANSACTION_TYPES` / `MTL_TRANSACTION_SOURCES` | 事务类型/来源 | `TRANSACTION_TYPE_ID`、Action、Source Type、Source ID | 解释 Receipt、Issue、Transfer、WIP、Sales Order 等事务语义 |
| `MTL_TRANSACTIONS_INTERFACE` | 物料事务接口 | 接口 ID、事务类型、物料/组织、数量、处理状态 | 外部库存事务输入 |
| `MTL_MATERIAL_TRANSACTIONS_TEMP` | 物料事务工作层 | `TRANSACTION_TEMP_ID`、`TRANSACTION_HEADER_ID`、处理状态 | Transaction Worker 临时处理；不得手工改状态 |
| `RCV_SHIPMENT_HEADERS/LINES` | 接收装运头/行 | `SHIPMENT_HEADER_ID`、`SHIPMENT_LINE_ID`、供应商、PO、物料/数量 | Receipt/Deliver 来源与接收截止 |
| `RCV_TRANSACTIONS` | 接收事务 | `TRANSACTION_ID`、`PARENT_TRANSACTION_ID`、PO 发运行、事务类型、数量 | Receive、Inspect、Deliver、Return、Correct 事件链 |
| `CST_COST_TYPES` | 成本类型 | `COST_TYPE_ID`、Cost Type、更新许可 | Frozen、Pending、Simulation、Average/Periodic Rates 版本 |
| `CST_ITEM_COSTS` | 物料成本汇总 | 物料、组织、Cost Type、要素汇总、Item Cost | Frozen/Pending 成本版本与单价 |
| `CST_ITEM_COST_DETAILS` | 物料成本明细 | 成本要素、子要素、层级、资源/费率、Item Cost | Material、Resource、OSP、Overhead 分解 |
| `CST_QUANTITY_LAYERS` | 数量/成本层 | Layer、Item、Org、Cost Group、数量、Item Cost | FIFO/LIFO 或层成本收发及结转 |
| Cost Rollup/Cost Update | 成本卷积/更新批次 | Cost Type、物料/组织、BOM/Routing/费率、生效日、请求 ID | Pending → Frozen、标准成本重估和成本历史 |
| `WIP_ENTITIES` | 工单实体 | `WIP_ENTITY_ID`、工单号、组织、状态 | 离散制造工单 |
| `WIP_DISCRETE_JOBS` | 离散工单 | `WIP_ENTITY_ID`、`STATUS_TYPE`、主装配、数量、Accounting Class | 工单状态、计划量、完工和关闭 |
| `WIP_OPERATIONS` | 工序 | 工单、工序号、部门、资源、计数点/完成点 | Move、资源报工、工序完工 |
| `WIP_TRANSACTIONS` | WIP 事务 | 工单、事务类型、数量、资源/材料 | 发料、退料、移动、资源、OSP、完工、废品、关闭 |
| `WIP_TRANSACTION_ACCOUNTS` | WIP 会计分布 | `TRANSACTION_ID`、Accounting Line Type、Reference Account | WIP 要素、吸收、差异和清算账户 |
| BOM/Routing/Resource/Overhead | 制造成本来源 | BOM 组件、Routing 工序、Resource/Rate、Overhead/Basis | 成本卷积的 Previous/This Level 输入 |
| `ORG_ACCT_PERIODS` | 库存会计期间 | `ACCT_PERIOD_ID`、组织、期间、`OPEN_FLAG`、关闭日期 | 成本事务、库存关期和截止控制 |
| COGS Recognition / Revenue Matching | 销售成本匹配 | Order/Delivery/Line、Inventory Issue、收入事件、处理状态 | Deferred COGS → COGS 与收入确认比例匹配 |
| Landed Cost / Cost Factor | 到岸成本因子 | Shipment/Receipt、费用类型、分摊基础、估计/实际、币种 | 运费、保险、关税、处理费等进入库存/在途/费用 |
| `XLA_EVENTS` | 成本会计事件 | `EVENT_ID`、实体、事件类型、状态和会计日期 | 物料/WIP/COGS/LCM 事务是否具备 SLA 会计资格 |
| `XLA_AE_HEADERS/LINES` | SLA 分录头/行 | `AE_HEADER_ID`、事件、会计类、CCID、借贷、传送状态 | 成本子账分录、账户推导和 GL 下钻 |
| `GL_IMPORT_REFERENCES` | GL 来源引用 | `JE_HEADER_ID`、`JE_LINE_NUM`、来源事务标识 | 从总账行回溯成本事务或 SLA |
| `FND_CONCURRENT_REQUESTS` | 并发请求 | `REQUEST_ID`、程序、参数、Phase/Status、日志/输出 | Transaction Manager、Cost Manager、Create Accounting 和报表执行证据 |

### 关键字段判读

| 字段/状态 | 判读方式 | 常见误区与控制 |
| --- | --- | --- |
| `TRANSACTION_QUANTITY` / `PRIMARY_QUANTITY` | 前者是事务 UOM 数量，后者是主单位数量；跨来源对账优先统一主单位 | 不能把不同 UOM 的数量直接相加；需同时查看 UOM 转换和负数方向 |
| `COSTED_FLAG` / `ACTUAL_COST` | 判断物料事务是否已成本及其单位成本；状态码需结合 `ERROR_CODE`、错误说明和 Cost Manager 日志解码 | 数量已写入 MMT 不代表已完成成本；不要直接修改标志位 |
| `PROCESS_FLAG` / `LOCK_FLAG` | 判断接口行是否等待、处理中或错误，以及是否被工作器锁定 | 不要通过手工解锁或重置 Flag 规避标准重处理；先保留批次和请求证据 |
| `TRANSACTION_DATE` / `ACCT_PERIOD_ID` | 决定成本事务落在哪个库存组织期间；会计日期和业务发生日期需分开记录 | 跨期间回溯事务可能阻塞关期或改变历史成本，须有审批和截止规则 |
| `COST_TYPE_ID` / `LEVEL_TYPE` | 解释成本版本、成本要素和 This/Previous Level 的来源 | 只看 `CST_ITEM_COSTS.ITEM_COST` 会丢失子要素、层级和费率细节 |
| WIP `STATUS_TYPE` / Accounting Class | 决定工单可执行的领料、移动、资源、完工、废品和关闭动作及其账户 | 状态名称和可执行事务受实例 Lookup/补丁影响，不能硬编码业务规则 |
| `EVENT_ID` / `EVENT_STATUS_CODE` / `PROCESS_STATUS_CODE` | 连接来源事务与 SLA 会计资格、处理状态和会计日期 | 事件已生成不代表已 Final 或已传 GL，必须继续核对 XLA 和 Journal Import |
| `SOURCE_CODE` / `HEADER_ID` / `LINE_ID` | 组成外部系统来源唯一键，连接接口批次与 EBS 事务 | 超时重试前先查询已生成事务，避免数量、成本和会计事件重复 |
| `REQUEST_ID` | 关联并发程序、参数、日志、输出和重跑范围 | 只记录程序名而不记录请求 ID，无法证明处理时点和结果 |

表名、列名、状态码和可查询视图会随 R12.2 补丁、已安装产品和本地化变化；带有“/”或功能名的条目是逻辑对象或对象集合，实施 SQL 需先用目标实例 eTRM、`ALL_TAB_COLUMNS`、Lookup 和 Integration Repository 核对。

### 名词解释

| 名词 | 解释 |
| --- | --- |
| Inventory Organization | 库存组织；库存数量、制造和物料事务边界 |
| Cost Organization | 成本组织；为一个或多个库存组织提供成本方法、成本类型、成本要素和账户边界；与 Master Organization/Cost Group 关系需按实例确认 |
| Costing Method | 成本方法；Standard、Average、FIFO/LIFO、Periodic 等决定库存出入库和制造成本的计价逻辑，不能与 Cost Type 混淆 |
| Cost Type | 成本类型/成本版本；Frozen、Pending、Simulation、Average 或 Periodic Rates 用于发布、模拟和周期费率维护 |
| Cost Group | 成本组；启用 Project References/Cost Collection 时可按项目等维度隔离平均成本，否则通常使用组织默认成本组；具体可用性以组织参数为准 |
| Cost Element | 成本要素；Material、Material Overhead、Resource、Outside Processing、Overhead 五类成本构成 |
| Cost Sub-element | 成本子要素；Item、Activity、Resource、Overhead Rate 等更细的费率或来源维度 |
| This Level / Previous Level | 本层/前层成本；区分当前装配的材料、资源、间接费与下层子装配传入成本 |
| Frozen/Pending/Simulation | 冻结/待更新/模拟成本版本；Pending 或 Simulation 只有经过审批和标准程序才会影响 Frozen |
| Cost Rollup | 成本卷积；按 BOM、Routing、资源、OSP 和 Overhead 计算多层物料成本并写入指定 Cost Type |
| Standard Cost Update | 标准成本更新；将 Pending 成本发布为 Frozen，重估库存并生成成本更新/调整报告 |
| Average Cost Update | 平均成本更新；按指定调整方式改变移动平均成本，通常重估现有及在途库存，WIP 边界须单独确认 |
| FIFO/LIFO Layer | FIFO/LIFO 成本层；按收发顺序维护数量和单位成本层，负库存和跨期间收发会增加层维护风险 |
| Periodic Costing | 周期成本；在期间内累计数量/成本，期末按周期平均或增量层计算，不等同于实时 Perpetual Costing |
| Costed / Uncosted | 已成本/未成本；事务数量已落地但成本计算尚未完成或失败时，不能直接作为库存价值最终结果 |
| PPV/IPV | Purchase/Invoice Price Variance；采购价格与标准/接收价格、发票价格与采购/接收价格的差异 |
| Usage/Rate/Efficiency Variance | 用量/费率/效率差异；分别解释材料数量、资源/间接费率和实际工时/产出效率偏差 |
| WIP Close Variance | 工单关闭差异；工单投入、完工、废品和标准/实际成本在关闭时的未吸收差异 |
| WIP | Work in Process；在制品；由工单、工序、材料、资源、OSP、Overhead、完工和关闭差异组成 |
| WIP Accounting Class | WIP 会计分类；为离散工单定义材料、资源、外协、间接费、估价、吸收和差异账户 |
| Discrete Job / Repetitive Schedule | 离散工单/重复计划；分别按工单实体或重复生产计划收集材料、资源、完工和差异 |
| Material Issue / Backflush | 手工发料/倒冲；分别由仓库事务或工序/完工规则自动扣减组件，必须防止重复扣料 |
| Resource / OSP / Overhead | 资源/外协/制造费用；分别按人工机器费率、外协采购服务和间接费用池吸收 WIP 成本 |
| Receipt / Deliver / Return / Correct | 接收/交付/退货/更正；组成接收事件链，需通过父子事务追溯原始数量和会计 |
| Receiving Inspection / Accrual | 接收检验/接收应计；分别是质量路由和收货到发票之间的暂估负债/清算控制 |
| COGS Matching | 销售成本匹配；发运产生的库存成本先进入 Deferred COGS，再按收入确认规则转入 COGS |
| Inventory Issue | 库存发出；销售、WIP、杂项或组织间发出导致现有量减少并生成库存价值释放 |
| LCM | Landed Cost Management；到岸成本管理，将运费、保险、关税、处理和仓储等费用分摊至相关收货/库存/在途 |
| Cost Factor | 到岸成本因子；定义费用类型、供应商/服务商、币种、有效期和分摊基础 |
| Estimated / Actual Landed Cost | 预计/实际到岸成本；预计可在收货前或后计算，实际由费用发票/单据匹配并形成差异 |
| Cost Manager / Transaction Manager | 成本管理器/事务管理器；异步处理物料事务、成本计算和会计前置，不应通过手改状态恢复 |
| Cost Period Close | 成本期间关闭；完成未处理事务、未计成本、WIP、接收应计、COGS 和 GL 对账后关闭库存组织期间 |
| Lot/Serial / Negative Inventory | 批次/序列/负库存；影响追溯、成本层和允许事务，需在接口和关期前重点检查 |
| SLA Cost Accounting | 成本子账会计；把库存、WIP、COGS 和 LCM 事件按规则生成 XLA 分录并传入 GL |

官方基线：[Inventory User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48820/toc.htm)、[Cost Management](https://docs.oracle.com/cd/E26401_01/doc.122/e48829/toc.htm)、[WIP User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48905/toc.htm)、[Landed Cost Management](https://docs.oracle.com/cd/E26401_01/doc.122/e48799/toc.htm)。

<a id="dict-08"></a>
## 08 报表、关账与治理

### 核心对象/字段

| 对象/字段 | 粒度 | 关键字段或关系 | 用途 |
| --- | --- | --- | --- |
| `GL_BALANCES` | 账簿-科目-期间余额 | Ledger、CCID、期间、余额类型、借贷余额 | FSG/管理报表基础 |
| `GL_CODE_COMBINATIONS` | 会计组合 | `CODE_COMBINATION_ID`、Accounting Flexfield 段值、启用/有效日期 | 账户范围、安全和报表行取数 |
| `GL_PERIOD_STATUSES` | Ledger-期间状态 | Ledger、应用、期间、打开/关闭状态、打开/关闭日期 | 报表是否可取数、迟到调整和关账门禁 |
| `GL_JE_BATCHES` | 日记账批次 | `JE_BATCH_ID`、来源、类别、期间、状态、控制总额 | Journal Import/Posting 批次控制 |
| `GL_JE_HEADERS/LINES` | 日记账头/行 | 来源、类别、期间、CCID、借贷金额、过账状态 | 报表下钻和控制总额 |
| `GL_INTERFACE` | GL 日记账接口 | Ledger/CCID、期间、币种、借贷、来源/类别、Group ID、错误信息 | 外部/子账批量导入 Journal Import |
| `GL_IMPORT_REFERENCES` | GL 来源引用 | `JE_HEADER_ID`、`JE_LINE_NUM`、来源事务标识 | 从总账行回溯成本事务或 SLA |
| `XLA_AE_HEADERS/LINES` | SLA 分录 | 事件、会计类、账户、借贷、传送状态 | 子账到 GL 血缘 |
| `XLA_EVENTS` | SLA 事件 | `EVENT_ID`、实体、事件类型、状态、会计日期 | 判断事务是否已生成/处理会计事件 |
| `FND_CONCURRENT_REQUESTS` | 报表请求 | `REQUEST_ID`、程序、参数、Phase/Status、输出 | 报表执行证据 |
| `FND_CONCURRENT_PROGRAMS` / Request Group | 并发程序/请求组 | 程序、执行文件、参数、职责可见范围、互斥规则 | 报表注册、调度和权限 |
| FSG Row/Column/Content Set | FSG 定义组件 | Row/Column、账户范围、金额类型、期间偏移、段拆分 | 可复用财务报表定义 |
| FSG Report/Report Set/Access Set | FSG 报表与权限 | 组合定义、运行参数、Use/View/Modify、Segment Security | 批量运行、版本和访问控制 |
| BI Publisher Data Model/Template | 数据模型/模板 | 数据集、参数、模板版本、输出格式 | 模板报表和分发 |
| BI Publisher Bursting | 分发定义 | 分割键、收件人、文件名、语言、输出格式、传输渠道 | 按法人/组织拆分和外发控制 |
| Web ADI Integrator/Layout | Excel 集成定义 | 字段、默认值、值集、校验、上传目标和职责 | 受控批量下载/上传 |
| ECC Load Definition | 加载定义 | Full/Incremental/Push、计划、索引新鲜度 | 运营看板数据新鲜度 |
| ECC Data Set/Page/Region | ECC 逻辑对象 | 数据集、页面、搜索、指标、图表、动作和角色 | 运营检索、可视化和数据安全 |
| ECC Load Request/Audit | ECC 加载请求/审计 | 请求、批次水位、开始/结束时间、行数、错误和状态 | 数据加载追踪、重跑和审计 |
| `GL_DAILY_RATES` | 日汇率 | From/To Currency、Conversion Date/Type、正向/反向汇率 | 外币日记账、重估、折算和报告币种 |
| `GL_DAILY_RATES_INTERFACE` | 日汇率接口 | 币种对、日期范围、类型、汇率、模式、错误码 | 受控批量插入/更新/删除日汇率 |
| Historical Rates | 历史汇率/金额 | Ledger、Target Currency、期间、账户、Rate/Amount、Rate Type | 权益、非货币性项目和时态法翻译 |
| Report Catalog/Version | 报表目录/版本 | Owner、Data Steward、来源、参数、版本、状态、保留期、替代关系 | 生命周期、重复清理和审计定位 |
| Control Total/Sign-off | 控制总额/签核 | 来源/子账/SLA/GL/报表金额、差异、复核人、批准 | 月结完整性和管理责任 |
| Audit/Retention Policy | 审计/保留策略 | 对象、审计字段、保留期限、脱敏、归档、销毁批准 | SOX/法定/隐私和诉讼保全 |

### 关键字段判读

| 字段/对象 | 判读方式 | 常见误区与控制 |
| --- | --- | --- |
| `LEDGER_ID` / `SET_OF_BOOKS_ID` | 确认报表、日记账、余额和子账事件属于同一 Ledger；旧版本字段可能仍出现在接口/历史对象 | 不能用 Ledger Name 代替 ID；跨 Ledger Set 必须逐 Ledger 对账 |
| `CODE_COMBINATION_ID` | 连接 Accounting Flexfield 段值、GL 余额、日记账和安全规则 | 同一自然科目在不同法人/成本中心不是同一账户；报表要保留完整 CCID/段值 |
| `PERIOD_NAME` / `PERIOD_NUM` / `PERIOD_YEAR` | 确认会计期间、财务日历和调整期间；运行期间不一定等于交易日期 | 不要用自然月份字符串替代期间；年结/调整期需单独标识 |
| `ACTUAL_FLAG` / `BALANCE_TYPE` | 区分 Actual、Budget、Encumbrance、Statistical 或平均余额 | Actual 与 Budget 混列必须明确列标题、来源和比较基准 |
| `STATUS` / `POSTED_FLAG` | 判断日记账、批次、请求或输出是否完成；具体代码需查 Lookup | Request Normal 不等于 Journal Posted；必须继续核对 Journal Import/Posting |
| `GROUP_ID` / `REFERENCE*` | 连接 GL_INTERFACE 批次、外部来源和下钻引用 | 外部批次必须有唯一键和控制总额，避免重传重复分录 |
| FSG `Amount Type` / `Period Offset` | 决定 PTD/QTD/YTD、Actual/Budget 和相对运行期间 | 期间偏移、CPOI、调整期和平均余额要用测试期间验证 |
| FSG `Display Type` / `Change Sign` | Expand/Total/Both 控制行粒度；Change Sign 只改变显示 | 显示正负不能改变实际借贷；汇总行可能与明细行重复计算 |
| BI Publisher Parameter/Template Version | 参数决定数据范围，模板决定版式；两者必须独立版本 | 模板中的硬编码过滤/合计会造成口径漂移，不能只审 PDF 外观 |
| ECC Load Request/Last Successful Load | 判断数据集水位、行数、错误数和最后成功时间 | 页面显示旧数据时先查加载请求和索引，不直接修改来源交易 |
| `REQUEST_ID` / Output File | 关联并发程序、参数、日志、输出、校验值和归档 | 只保留文件名无法复现运行条件；请求和输出均需受访问控制 |
| Access Set / Responsibility / MOAC | 叠加决定 Ledger、OU、组织、菜单、请求和段值可见范围 | 只测页面可见性不够，还要测试下钻、导出、Bursting 和接口写入 |

对象名称、列名、状态码和可查询视图会随 R12.2 补丁、已安装产品和本地化变化；带有“/”或功能名的条目是逻辑对象或对象集合，实施 SQL 需先用目标实例 eTRM、`ALL_TAB_COLUMNS`、Lookup 和 Integration Repository 核对。

### 名词解释

| 名词 | 解释 |
| --- | --- |
| FSG | Financial Statement Generator；基于 GL 余额和行/列集生成财务报表 |
| Row Set | FSG 行集；定义账户范围、行标签、计算、显示类型和格式 |
| Column Set | FSG 列集；定义 Amount Type、期间偏移、标题、币种、格式和计算 |
| Content Set | FSG 内容集；按法人、部门、成本中心、产品、Ledger 或报告币种拆分输出 |
| Row Order | FSG 行顺序；按段值、描述或列金额排序并控制段显示 |
| Display Set / Display Group | FSG 显示集/显示组；控制行列显示、隐藏和组范围，不改变余额 |
| Report Definition | FSG 报表定义；组合 Row/Column Set 及可选组件，运行时参数不反写定义 |
| Report Set | FSG 报表集；将多个报表按顺序和共同参数批量运行 |
| Amount Type | FSG 金额类型；PTD/QTD/YTD 的 Actual、Budget、Encumbrance、Variance 或平均余额 |
| Currency Control Value | FSG 币种控制值；把 Converted、Entered、Statistical、Total 及 Entered/Ledger Currency 绑定到控制值编号 |
| Balance Control Currency | FSG 行/列的余额控制币种；决定未指定 Currency Control Value 时的取数币种；可为 `STAT` |
| Converted / Entered / Statistical / Total | FSG 四类币种余额取数方式；分别表示已转换、原始输入、统计量和账簿/报告币种总额 |
| Period Offset / POI | 期间偏移/期间关注点；相对运行期间选择前后期间，`0` 表示当前期间 |
| Constant Period of Interest | 固定期间关注点；无论运行期间如何变化都引用同一会计期间 |
| Segment Override | FSG 段覆盖；在列集、报表定义或运行时限定账户段值，优先级需受控 |
| Change Sign | FSG 显示符号；只改变报表显示，不改变 GL 借贷余额 |
| Definition Access Set | 定义访问集；控制 FSG 定义及汇率类型的 Use/View/Modify 权限 |
| Report Manager | EBS 原生报表提交/查看框架；与外部 Office 插件不同 |
| BI Publisher | 模板化报表和批量分发工具 |
| RXi | Report eXchange；可配置的标准报表输出框架 |
| Smart View | Office 分析插件；需单独部署连接器，不是 EBS 原生 GL 报表引擎 |
| ECC | Enterprise Command Center；运营搜索和可视化；新鲜度由加载模式/计划决定 |
| Control Total | 控制总额；来源、子账、SLA、GL 或报表之间的数量/金额校验 |
| Data Lineage | 数据血缘；报表单元格到余额、日记账、SLA、来源交易的追溯链 |
| Conversion Rate Type | 汇率类型；Spot、Corporate、User、EMU Fixed 或客户自定义类型，决定同一币种对/日期的汇率口径 |
| Spot Rate | 即期汇率；指定日期的市场报价，通常从日汇率表取得 |
| Corporate Rate | 公司标准汇率；由财务统一发布，供多个模块采用 |
| User Rate | 用户汇率；在交易/日记账录入时直接指定，不在日汇率窗口维护 |
| EMU Fixed | 欧元/EMU 固定汇率关系；只适用于法定固定转换，不是一般市场汇率 |
| Period-End Rate | 期末汇率；通常用于资产负债表余额的期末折算 |
| Period-Average Rate | 期间平均汇率；通常用于收入费用的期间折算，具体口径依 Ledger 配置 |
| Historical Rate/Amount | 历史汇率/金额；按期间和账户维护，覆盖 Period-End/Period-Average 的默认选择 |
| Conversion | 多币种换算；交易录入时将 Entered Currency 转成 Ledger Currency |
| Revaluation | 重估；按指定日期汇率调整外币货币性账户并生成未实现汇兑损益 |
| Translation | 折算；把 Ledger 余额按期末/平均/历史规则重述到报告币种，差额通常进 CTA |
| Remeasurement | 再计量/时态法；非货币性项目使用历史汇率，差额按规则进入损益 |
| Cumulative Translation Adjustment (CTA) | 累计折算调整；用于平衡权益法折算产生的净差额，账户类型和追踪段需配置 |
| Rate Date / Conversion Date | 汇率日期；决定从日汇率表取哪一天的 Type；不一定等于交易日期或会计日期 |
| Inverse Rate | 反向汇率；To→From 的换算率，是否强制为正向率倒数由实例 Profile 控制 |
| Currency Rates Manager | 汇率管理器；统一维护、上传、下载日汇率/历史汇率并计算 Cross Rates |
| Ledger Set | 账簿集；将多个 Ledger 作为一组运行查询或 FSG 报表，结果需保留各账簿及合计口径 |
| Source / Category | 日记账来源/类别；用于区分子账、手工、接口和调整性质，也是报表过滤维度 |
| Journal Import | 日记账导入；将 `GL_INTERFACE` 验证并创建 GL 日记账，失败行需根据错误报告修正 |
| Posting | 过账；将已导入日记账更新到 `GL_BALANCES`；导入成功不等于余额已更新 |
| Suspense / Invalid Account | 暂记/无效账户；临时承接不能确定或不合法的账户，月结前必须解释并清零或批准例外 |
| FSG Display Type | Expand、Total、Both；控制段值展开、合计或同时显示明细/合计 |
| FSG Calculation | FSG 行/列计算；用于合计、差异、比例和统计，需明确计算优先级与舍入 |
| FSG Rounding | FSG 显示/计算舍入；可能导致报表行合计与原始余额存在小额差异，必须设定阈值 |
| Segment Value Security | 段值安全；按用户/职责限制 FSG 可输出的账户段值，需要启用相应 Profile |
| Report Catalog | 报表目录；集中登记用途、Owner、数据源、参数、权限、版本、刷新和保留信息 |
| Data Steward | 数据管理员；维护业务定义、映射、质量规则和来源解释，不等同于技术开发者 |
| Semantic Layer | 语义层；统一期间、币种、维度、状态、指标公式和下钻键，隔离展现格式 |
| Refresh SLA | 刷新服务级别；定义数据最长允许延迟、成功水位、监控和超时升级 |
| Concurrent Program | 并发程序；注册执行文件、参数和值集，通过 SRS/请求组统一运行 |
| Concurrent Request | 并发请求；某次带参数的程序运行实例，包含 Phase/Status、日志和输出 |
| Request Set | 请求集；按顺序或条件组织多个并发请求，适用于日批、月结和报表集 |
| BI Publisher Data Definition | BI Publisher 数据定义；登记 XML 数据源/并发程序/Web Service 与参数 |
| BI Publisher Data Model | BI Publisher 数据模型；定义数据集、参数、SQL、事件和安全过滤 |
| BI Publisher Template | BI Publisher 模板；将 XML 数据渲染为 RTF、PDF、XSL、Excel 等版式输出 |
| Bursting | 分发拆分；按分割键生成多个文件并映射收件人、语言、格式和渠道 |
| Web ADI Integrator | Web ADI 集成器；定义 Excel 与 EBS 任务之间的元数据和上传目标 |
| Web ADI Layout | Web ADI 布局；定义字段位置、默认值、值列表、必填和显示规则 |
| ECC Data Set | ECC 数据集；将来源数据加载到索引，供页面、搜索、指标和图表使用 |
| ECC Load Request | ECC 加载请求；记录 Full/Incremental/Push 加载的批次、行数、错误和水位 |
| Full/Incremental/Push Load | ECC 全量/增量/推送加载；分别适合初始化、变化同步和事件驱动更新 |
| Drilldown | 下钻；从报表单元格/余额进入日记账、SLA、子账和业务单据的追溯动作 |
| Control Total | 控制总额；来源、子账、SLA、GL、报表或分发批次之间的数量/金额证明 |
| Reconciliation | 对账；按相同 Ledger、期间、币种、过滤和舍入口径比较两侧证据 |
| Exception Item | 例外项；未达到控制条件但有金额、风险、责任人、临时控制和解决期限的事项 |
| Compensating Control | 补偿控制；无法及时修复系统/数据缺陷时，经批准的人工或替代核验措施 |
| Audit Trail | 审计轨迹；记录用户、时间、前后值、原因、批准、请求和对象版本 |
| Retention / Archive | 保留/归档；按法规、税务、合同、审计和诉讼保全保留定义、输出、日志和签核 |
| Data Masking | 数据脱敏；在非生产环境屏蔽个人、银行、税务和商业敏感字段 |
| Output Checksum | 输出校验值；验证 PDF/XML/Excel 文件未被替换或损坏，配合版本和参数复现 |

官方基线：[General Ledger User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48748/toc.htm)、[Financial Reporting/FSG](https://docs.oracle.com/cd/E26401_01/doc.122/e48748/T312864T313620.htm)、[Report Manager](https://docs.oracle.com/cd/E26401_01/doc.122/e22006/toc.htm)、[EBS Concepts — Web ADI/BI Publisher](https://docs.oracle.com/cd/E26401_01/doc.122/e22949/T120505T120508.htm)、[ECC Administrator](https://docs.oracle.com/cd/E26401_01/doc.122/f34732/toc.htm)、[ECC Extending](https://docs.oracle.com/cd/E26401_01/doc.122/f21671/T673609T673614.htm)、[ECC Push Load/Java API](https://docs.oracle.com/cd/E26401_01/doc.122/f21671/T673609T676700.htm)。

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
| `FND_USER` | EBS 应用用户 | `USER_ID`、`USER_NAME`、起止日期、启用状态 | 本地账户与 SSO 映射 |
| `FND_PROFILE_OPTIONS` / `FND_PROFILE_OPTION_VALUES` | Profile 定义与设置值 | Profile 名、层级、层级值、运行时最终值 | 密码、会话和功能配置 |
| `FND_UNSUCCESSFUL_LOGINS` | 失败登录记录 | 用户、时间、失败次数、来源线索 | 账号锁定和安全审计 |
| `WSDL` / `WADL` | SOAP/REST 服务契约 | Endpoint、操作、参数、认证策略 | 客户端生成与契约测试 |
| `WF_DEFERRED` / Service Invocation Monitor | 出站服务消息 | Event Key、状态、重试、响应/错误 | EBS 调用外部 Web Service |
| `fs1` / `fs2` | 版本化应用文件系统 | APPL_TOP、INST_TOP、Run/Patch 角色 | ADOP 在线补丁与代码发布 |
| `fs_ne` | 非版本化文件系统 | 并发日志/输出、导入导出和跨版本运行数据 | 保存跨 Cutover 的运行数据 |
| `APPL_TOP` / `COMMON_TOP` / `INST_TOP` | 应用目录层级 | 产品代码、公共文件、节点上下文和管理脚本 | 文件定位与配置排错 |
| OHS / WebLogic Managed Server | Web 中间件 | Web Entry、`oacore`、`oafm`、`forms`、`forms-c4ws` | OAF、Forms、ISG 服务承载 |
| ICM / GSM / Service Manager | 并发调度基础 | 队列、节点、工作班次、`FNDSM` | 并发服务启停与故障转移 |
| OPP / Workflow Mailer | 后台服务 | 输出后处理、通知发送/接收 | 报表输出与工作流通知 |
| `APPLCSF` / `NE_BASE` | 日志、输出和非版本化路径 | Context 变量、实例目录、`logs/appl/conc` | 跨 Cutover 保存并发/接口证据 |
| Context File / AutoConfig | 实例配置来源 | 节点、端口、路径、服务状态、模板 | 生成配置和环境一致性 |
| WebLogic Domain / Node Manager | 中间件运行域 | AdminServer、Managed Server、JRF、启动控制 | 应用服务启停和集群治理 |
| `FND_EXECUTABLES` / `FND_REQUEST_SETS` | 并发执行定义 | 执行方法、文件名、Stage、Link | 程序注册和请求集编排 |
| Value Set / `FND_FLEX_VALUE_SETS` | 并发参数和字段值验证 | 格式、验证类型、独立值/表验证 | 参数校验和职责级配置 |
| XDO Data Definition / Template | BI Publisher 报表 | XML 数据、RTF/Excel 模板、语言/地区 | OPP 后处理和输出分发 |
| XML Gateway / AQ | B2B 消息与队列 | Message Map、Trading Partner、ECX 队列 | XML/EDI 入站、出站和重试 |
| EDI Gateway / e-Commerce Gateway | EDI/B2B 交易 | Transaction Type、Trading Partner、Translator/文件 | 标准 EDI 报文交换和对账 |
| CEMLI 工件 | 定制交付物 | Config/Extension/Modification/Localization/Integration 分类 | 升级、补丁和回退治理 |

### 名词解释

| 名词 | 解释 |
| --- | --- |
| APPS | EBS 应用访问层；不应作为终端用户长期直连授权边界 |
| Custom Schema | 自定义数据库 Schema；按 Developer's Guide 注册并受控授权 |
| APPS_INITIALIZE | 初始化用户、职责和应用上下文的公开入口 |
| EBR | Edition-Based Redefinition；R12.2 在线补丁依赖的版本化数据库机制 |
| ADOP | Online Patching Utility；R12.2 在线补丁生命周期工具 |
| ISG | Integrated SOA Gateway；EBS API、并发程序和开放接口的服务发布框架 |
| Business Service Object（BSO） | Integration Repository 中可发布的业务服务对象；方法和数据类型以目标实例契约为准 |
| Business Event System（BES） | Workflow 业务事件发布/订阅框架；支持本地或 Deferred Subscription |
| OAF/Forms | Oracle Application Framework / Oracle Forms；EBS 页面技术栈 |
| Workflow | 工作流；Item、活动、通知和结果组成的业务流程引擎 |
| SSO | Single Sign-On；统一认证入口，不替代 EBS 职责、菜单和 MOAC 授权 |
| OAM / AccessGate | Oracle Access Manager / EBS AccessGate；将外部身份映射到 EBS 应用会话的桥接组件 |
| GUID | Globally Unique Identifier；目录与 `FND_USER` 之间的稳定身份关联键 |
| PasswordValidation | EBS 客制化密码校验 Java 接口；由 `SIGNON_PASSWORD_CUSTOM` 指定实现类 |
| WSDL / WADL | SOAP / REST 的机器可读服务契约；客户端应以部署后版本为准 |
| WS-Security | SOAP 消息安全规范；EBS ISG 可按策略使用 UsernameToken 或 SAML |
| SIF | Service Invocation Framework；EBS 通过 Workflow/JBES 调用外部 SOAP/REST 的框架 |
| Service Alias | ISG REST 服务别名；用于稳定定位已部署资源，不能与旧 Endpoint 混用 |
| Grant | 接口或服务授权；需按用户、职责、方法和数据范围最小化配置 |
| Correlation ID | 跨系统追踪号；串联传输日志、EBS 请求、业务主键和回执 |
| MOAC | Multi-Org Access Control；由职责/安全配置决定可访问的组织范围 |
| Run/Patch File System | Run/Patch 文件系统；`fs1` 与 `fs2` 在 ADOP 周期中交替承担运行和补丁角色 |
| fs_ne | Non-Editioned File System；跨版本保留并发日志、输出和运行数据，不存放应用代码 |
| APPL_TOP | EBS 应用顶层目录；包含技术文件、产品目录和环境文件 |
| COMMON_TOP | 公共目录；多个产品共用的脚本、模板、帮助和公共文件区域 |
| INST_TOP | 实例顶层目录；保存节点上下文、AutoConfig 和服务管理入口 |
| OHS | Oracle HTTP Server；EBS Web Entry Point 的 TLS、代理和静态资源入口 |
| WebLogic Managed Server | WebLogic 受管服务器；承载 `oacore/oafm/forms/forms-c4ws` 等 EBS 应用服务 |
| GSM | Generic Service Management；按节点和工作班次管理 EBS 后台服务 |
| ICM | Internal Concurrent Manager；控制并发管理器、Service Manager 和队列调度 |
| OPP | Output Post Processor；对并发请求输出执行 XML/PDF/Excel 等后处理 |
| PCP | Parallel Concurrent Processing；将并发管理器分布到多个应用节点并支持故障转移 |
| Service Group | AutoConfig 服务组；按 Root、Web、Batch、Other 等角色启用/停用应用服务 |
| Managed Server | WebLogic 受管服务器；`oacore`、`oafm`、`forms`、`forms-c4ws` 等服务实例的运行容器 |
| Node Manager | WebLogic 节点管理器；受管服务器的启动、停止和健康监控代理 |
| Context File | EBS AutoConfig 上下文文件；保存节点、端口、路径和服务参数，应用层通常位于 `INST_TOP` |
| AutoConfig | 根据 Context、模板和驱动生成/维护 EBS 配置；生成文件不应长期手工修改 |
| APPLCSF | Concurrent Processing 日志/输出根目录；R12.2 通常指向 `fs_ne` 的非版本化区域，实际值以 Context 为准 |
| NE_BASE | `fs_ne` 的环境变量；由 `s_ne_base` Context 变量提供非版本化文件系统路径 |
| FILE_EDITION | 当前环境文件系统角色标识；维护前确认是 `run` 还是 `patch` |
| Java Concurrent Program | 并发执行方法之一；由 Concurrent Manager 按参数、日志、输出和完成状态运行 Java 类 |
| OAF Extension | OAF Controller/AM/VO/EO 的受支持扩展；不等于修改 seeded XML 或 Java 类 |
| Value Set | 并发参数/字段的值验证定义；决定格式、列表、独立值或表验证 |
| Request Set | 由多个 Stage/Program 组成的批处理编排；需定义顺序、条件、失败和重启语义 |
| Data Definition | BI Publisher 数据定义；描述 XML 数据集、参数和输出结构 |
| XDO/Template | XML Publisher/BI Publisher 的数据与版式工件；模板版本、语言、字体需一并治理 |
| eText | BI Publisher 文本输出格式；常用于银行 EFT、EDI 或固定宽度文件，需按外部规范校验 |
| XML Gateway | 基于 Workflow Business Event、Message Map、AQ 和 Transport Agent 的 XML/B2B 集成组件 |
| Trading Partner | XML Gateway/e-Commerce Gateway 的外部交易伙伴、地址或站点定义 |
| Message Map | XML Gateway 的 RDBMS↔XML 字段、代码转换和流程控制映射 |
| Advanced Queuing（AQ） | Oracle 持久化消息队列；XML Gateway、Workflow 等组件可用其传递异步消息 |
| FNDLOAD | FND Generic Loader；使用 LCT/LDT 下载/上传受支持配置，参数必须按对象类型验证 |
| WFLOAD | Workflow Loader；迁移 Workflow 定义、活动、消息和相关元数据 |
| XDOLoader | BI Publisher/XML Publisher Loader；迁移 Data Definition、Template 等报表工件 |
| Rapid Clone | EBS 克隆工具；源端 `adpreclone.pl`、目标端 `adcfgclone.pl`，完成后需 Context/AutoConfig 和外联隔离 |
| RAC | Real Application Clusters；数据库多实例架构，需同时考虑 Service Name、全局缓存和许可证 |
| Data Guard | 数据库备用/灾备能力；切换后要验证应用连接、并发、Workflow、文件和对账 |
| ETCC | EBS Technology Codelevel Checker；检查数据库/应用技术栈所需补丁和代码级别 |
| AD/TXK | EBS 应用 DBA 与技术栈补丁组件；提供 ADOP、AutoConfig、文件系统和技术对象更新能力 |

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
| CRP | Conference Room Pilot；方案演示和早期流程验证 |
| SIT | System Integration Test；系统集成测试，覆盖跨模块接口和会计链 |
| UAT | User Acceptance Test；业务用户验收测试 |
| CEMLI | Configuration、Extension、Modification、Localization、Integration；EBS 定制分类 |
| Cutover | 切换；冻结、迁移、增量、验证、Go/No-Go 和业务开放 |
| Hypercare | 上线强化支持；上线后高频监控、缺陷分诊和每日对账阶段 |
| RCA | Root Cause Analysis；根因分析；区分触发、根因、修复和预防 |
| RPO/RTO | Recovery Point/Time Objective；灾备可接受数据丢失点/恢复时长 |

官方基线：[EBS Concepts](https://docs.oracle.com/cd/E26401_01/doc.122/e22949/toc.htm)、[Maintenance Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22954/toc.htm)、[EBS Developer's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22961/toc.htm)。

## 维护规则

1. 新增字段前记录产品、补丁、对象所有者、粒度、主键、状态和官方来源。
2. 任何列名或状态值若没有 eTRM/实例证据，标记为“待目标实例复核”，不得编写无界生产 SQL。
3. 业务状态、接口状态、会计状态和并发状态分栏维护；不要用一个 `STATUS` 代替四类状态。
4. 统一参考见 [`90-reference.md`](90-reference.md)；模块原有深度字典见各模块文档的“常用表结构”章节。
