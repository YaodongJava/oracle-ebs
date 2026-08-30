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
| CE | Cash Management；现金管理；银行账户、对账单、自动对账和现金控制 |
| Bank Statement | 银行对账单；银行提供的账户级流水和余额 |
| AutoReconciliation | 自动对账；按交易码、参考号、金额/日期容差匹配 |
| Cash Clearing | 现金清算；源模块现金/清算账户和银行事实之间的过渡 |
| IBY | Oracle Payments；支付指令、格式、传输和回执框架 |
| EBTax/ZX | E-Business Tax；交易税务确定和税行引擎，不等于全部法定申报 |
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
| WIP | Work in Process；在制品；材料、资源、间接费和完工/关闭差异 |
| COGS Matching | 销售成本匹配；发运成本与收入确认协同 |
| LCM | Landed Cost Management；到岸成本管理；运费、关税等附加费用分摊 |

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
| `GL_DAILY_RATES` | 日汇率 | From/To Currency、Conversion Date/Type、正向/反向汇率 | 外币日记账、重估、折算和报告币种 |
| `GL_DAILY_RATES_INTERFACE` | 日汇率接口 | 币种对、日期范围、类型、汇率、模式、错误码 | 受控批量插入/更新/删除日汇率 |
| Historical Rates | 历史汇率/金额 | Ledger、Target Currency、期间、账户、Rate/Amount、Rate Type | 权益、非货币性项目和时态法翻译 |
| FSG Row/Column/Content Set | FSG 报表组件 | 账户/计算行、金额类型/期间列、段拆分 | 可复用地定义报表内容和维度 |
| FSG Report/Report Set | FSG 报表/报表集 | Row Set、Column Set、运行参数、顺序和调度 | 月结报表、管理报表和批量输出 |

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
