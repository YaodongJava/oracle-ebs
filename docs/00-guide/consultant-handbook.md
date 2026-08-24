# Oracle EBS R12.2.x 财务功能与技术顾问学习手册

> 本手册用于建立完整认知框架和项目工作方法，不替代目标实例的配置、补丁说明、Oracle 官方文档或 My Oracle Support（MOS）方案。示例会计分录仅用于理解数据流，实际科目、事件、借贷方向和入账时点必须以本项目配置与测试结果为准。

## 1. 如何使用本手册

| 读者 | 首轮重点 | 第二轮重点 | 能力验收 |
| --- | --- | --- | --- |
| 财务功能顾问（Financial Functional Consultant） | 第 2～7 章 | 第 10～13 章 | 能解释配置如何影响交易、会计、对账和关账 |
| 财务技术顾问（Financial Technical Consultant） | 第 2、5、8～12 章 | 第 13～15 章 | 能从业务单据追溯到接口、SLA、GL、日志和部署工件 |
| 集成顾问（Integration Consultant） | 第 5、9、11 章 | 各模块接口手册 | 能设计可幂等、可重放、可对账且受支持的接口 |
| 测试/运维人员 | 第 5、10、12～15 章 | 对应业务模块 | 能按证据链定位故障，并验证数量、金额和会计结果 |

建议每学完一个流程，至少完成四个动作：

1. 画出业务单据与状态流转。
2. 在测试环境完成一笔正常交易和一笔冲销/异常交易。
3. 从业务单据追溯到 Subledger Accounting（SLA，子账会计）和 General Ledger（GL，总账）。
4. 写出该流程的关账检查、对账公式、失败重跑边界和审计证据。

## 2. EBS 财务的六层认知模型

遇到任何配置、接口或会计问题，都先判断它属于哪一层，不要直接从表或报错开始猜。

| 层级 | 核心问题 | 典型对象 | 顾问输出 |
| --- | --- | --- | --- |
| 1. 企业与责任边界 | 谁拥有交易、资产、现金和法定责任？ | Legal Entity、Ledger、Operating Unit、Inventory Organization | 企业结构图、数据责任矩阵 |
| 2. 主数据与控制 | 与谁交易、用什么科目、币种、税和付款条件？ | Supplier、Customer、Bank、Item、COA、Tax、Payment Term | 主数据规则、配置工作簿 |
| 3. 业务交易 | 单据如何创建、审批、履行、冲销？ | PO、Receipt、Invoice、Receipt、Asset、Project、Material Transaction | 流程图、状态字典、例外流程 |
| 4. 子账会计 | 何时形成会计事件，如何派生科目和描述？ | Event、SLA Rule、Journal Line、Account Combination | 会计事件矩阵、会计分录样例 |
| 5. 总账与报告 | 如何传输、导入、过账、折算和报告？ | Journal、Ledger、Balance、FSG、Trial Balance | 过账控制、报表与对账包 |
| 6. 技术与运维 | 如何集成、监控、发布、恢复和审计？ | Interface、API、Concurrent、Workflow、ADOP、Log | 技术设计、运行手册、发布证据 |

常用故障定位顺序：

```text
业务事实 → 组织/账簿/期间 → 单据状态 → 接口/并发请求
        → 会计事件/SLA → GL 导入/过账 → 余额/报表 → 外部系统对账
```

## 3. 企业结构与共享基础

### 3.1 关键组织对象

| 英文名称 | 中文说明 | 主要作用 | 常见误区 |
| --- | --- | --- | --- |
| Business Group | 业务组 | 主要是人力资源和组织定义的顶层上下文 | 把它当成财务核算主体 |
| Legal Entity | 法律实体/法人实体 | 表示依法承担权利义务、税务和法定报告责任的实体 | 认为一个法人只能对应一个经营单位 |
| Ledger | 账簿 | 由 Chart of Accounts、Accounting Calendar、Currency、Accounting Method 等核算属性构成 | 为数据权限隔离而滥建账簿 |
| Primary Ledger | 主账簿 | 交易的主要会计记录 | 与 Reporting Currency 混淆 |
| Secondary Ledger | 辅助账簿 | 按不同会计方法、科目表、日历或币种保留另一套会计表示 | 未设计转换层级和失败对账 |
| Ledger Set | 账簿集 | 对多个账簿执行查询、报告或部分批处理的逻辑集合 | 误认为它会合并或复制交易 |
| Operating Unit（OU） | 经营单位 | 多组织访问控制下处理采购、应付、应收等交易的组织范围 | 忽略 MOAC 上下文导致查不到或误查数据 |
| Inventory Organization | 库存组织 | 管理库存、物料交易、成本和制造业务 | 认为它天然等于法人或 OU |
| Data Access Set | 数据访问集 | 控制用户可访问的账簿、平衡段值及读写范围 | 只配置职责，不验证数据访问集 |

企业结构设计应从法律责任、管理责任、业务执行、核算要求和数据权限五个视角同时验证。详细模型见 [企业组织、法人、账簿与经营单位](../01-common/organization.md)。

### 3.2 账簿的核心核算属性

- Chart of Accounts（COA，科目表）：定义会计科目结构、段、值集和验证规则。
- Accounting Calendar（会计日历）：定义期间、季度、年度和开关期节奏。
- Ledger Currency（账簿币种）：定义本位币及总账余额的主要币种视角。
- Subledger Accounting Method（子账会计方法）：关联会计规则定义，决定子账如何生成分录。
- Balancing Segment Value（平衡段值）：常用于公司/法人平衡；具体法律含义由企业设计决定。

COA 设计不仅是段数和段名。应同步明确：段的业务所有者、值的新增流程、父子汇总、交叉验证规则、动态插入、预算控制、公司间平衡、报表维度以及停用后的历史查询策略。

### 3.3 多组织访问控制

Multi-Org Access Control（MOAC，多组织访问控制）允许一个职责在安全配置允许时访问多个 OU。分析问题必须保存：用户、职责、应用、Security Profile（安全配置文件）、当前 OU、Ledger 和 Data Access Set。仅看到 `ORG_ID` 不代表已经理解数据权限。

### 3.4 共享主数据

- Trading Community Architecture（TCA，贸易社区架构）统一建模 Party（参与方）、Party Site（参与方地点）、Account（账户）和关系。供应商与客户是 TCA 参与方在不同产品中的业务角色。
- Centralized Bank Account Model（集中式银行账户模型）集中维护银行、分行、内部银行账户、账户所有者和用途；CE、AP、AR、IBY 等产品在其上使用不同业务能力。
- E-Business Tax（EBTax，电子商务税）以税制、税种、状态、税率、司法管辖区、税务登记和确定因素规则计算交易税。
- Key Flexfield（KFF，关键弹性域）构成具有业务意义的组合键；Descriptive Flexfield（DFF，说明性弹性域）扩展标准对象的附加属性。

## 4. 从业务交易到总账

### 4.1 标准会计链路

```text
业务交易
  ↓ 生成会计事件
XLA Events（会计事件）
  ↓ Create Accounting（创建会计）
XLA Accounting Entries（SLA 会计分录）
  ↓ Transfer to General Ledger（传送至总账）
GL Interface / Journal Import（总账接口/日记账导入）
  ↓ Posting（过账）
GL Balances（总账余额）
  ↓
Trial Balance / Account Analysis / Financial Report（试算平衡/科目分析/财务报告）
```

SLA 的价值是把“业务事件”与“会计表示”分离。功能顾问需要理解 Event Class（事件类）、Event Type（事件类型）、Journal Line Type（日记账行类型）、Account Derivation Rule（账户派生规则）、Description Rule（说明规则）和 Mapping Set（映射集）；技术顾问需要能用业务主键、`APPLICATION_ID`、`ENTITY_ID`、`EVENT_ID`、`AE_HEADER_ID`、`AE_LINE_NUM`、`GL_SL_LINK_ID` 等标识逐层追溯。

常见数据链如下，但列和连接条件必须在目标实例验证：

```text
业务表
  → XLA_TRANSACTION_ENTITIES
  → XLA_EVENTS
  → XLA_AE_HEADERS / XLA_AE_LINES
  → GL_IMPORT_REFERENCES（启用并保留导入引用时）
  → GL_JE_HEADERS / GL_JE_LINES
  → GL_BALANCES
```

不要把 `GL_BALANCES` 当作交易明细表，也不要假设所有来源都以完全相同方式填充 GL 导入引用。追溯应同时使用来源、类别、批次、请求号、期间、金额、币种和业务主键交叉验证。

### 4.2 会计状态的四个问题

对任何“没有进总账”的问题依次回答：

1. 业务交易是否已达到允许会计的状态？
2. 会计事件是否已创建，是否存在处理错误？
3. SLA 分录是否 Final（最终）而不是 Draft（草稿），是否已传 GL？
4. GL 日记账是否已导入、平衡并过账到正确期间和账簿？

### 4.3 对账的基本公式

对账不等于只比较总数。最低应按组织、账簿、期间、币种和业务分类比较数量与金额：

```text
期初余额 + 本期增加 - 本期减少 ± 调整/重估 = 期末余额
子账期末余额 + 已解释的时间性差异 = 总账控制账户余额
来源控制总额 = 成功记录 + 拒绝记录 + 在途记录（数量与金额分别成立）
```

## 5. 核心端到端流程

### 5.1 Record to Report（R2R，记录到报告）

```text
子账交易 → SLA → GL 导入 → 日记账审核/过账
        → 重估/折算/公司间/合并 → 对账 → 财务报告 → 关期
```

功能顾问重点：Ledger、Journal Source/Category、汇率、Suspense（暂记）、Intercompany（公司间）、Revaluation（重估）、Translation（折算）、Consolidation（合并）、关账日历和报表。

技术顾问重点：`GL_INTERFACE` 控制、Journal Import 请求、来源追溯、并发日志、SLA 到 GL 链、批量导入性能和失败重跑。

最低控制：日记账来源和类别受控；手工日记账审批与职责分离；子账控制账户禁止随意手工记账；未过账、未平衡、Suspense 和跨期日记账均有审阅。

详见 [GL 流程](../04-gl/process.md)、[SLA/FAH/AGIS](../04-gl/sla-fah-agis.md) 和 [R2R 关账](../08-e2e/record-to-report-close.md)。

### 5.2 Procure to Pay（P2P，采购到付款）

```text
供应商 → 请购 → 采购订单 → 收货/检验/入库
      → 应付发票匹配 → 验证/审批 → 付款批 → 银行文件/回执
      → 现金清算/银行对账 → SLA → GL
```

关键匹配概念：

- 2-way Match（二单匹配）：发票与采购订单比较。
- 3-way Match（三单匹配）：发票、采购订单与收货比较。
- 4-way Match（四单匹配）：再加入检验/接受数量。
- Tolerance（容差）：价格、数量、汇率等差异的允许范围。
- Hold（暂停）：阻止付款或后续处理的控制状态，必须区分系统暂停与人工暂停。

典型库存采购会计示意：

| 事件 | 借方示例 | 贷方示例 |
| --- | --- | --- |
| 接收 | Receiving Inspection（接收检验） | AP Accrual（应付暂估） |
| 入库 | Inventory Valuation（库存估值） | Receiving Inspection（接收检验） |
| 发票会计 | AP Accrual、税额、差异 | AP Liability（应付负债） |
| 付款 | AP Liability | Cash Clearing/现金 |
| 银行对账 | Cash Clearing | 现金（视清算设置） |

费用采购、期间末暂估、费用目的地、汇率差异和税务处理可能不同。必须用本项目设置和 SLA 输出确认。

最低对账：PO/收货未开票、收货暂估、发票匹配差异、AP Trial Balance（应付试算表）、付款在途、银行清算账户与 GL。

详见 [P2P 端到端](../08-e2e/procure-to-pay.md)、[AP 发票](../02-ap/invoices.md)、[付款](../02-ap/payments.md) 和 [IBY/费用报销](../02-ap/payments-iby-expenses.md)。

### 5.3 Order/Credit to Cash（O2C/C2C，订单/信用到收款）

```text
客户/TCA → 信用检查 → 销售订单 → 发运
         → AutoInvoice/AR 交易 → 收入/应收
         → 收款/核销/汇款 → 银行对账 → 催收/争议 → GL
```

功能顾问重点：Transaction Source/Type（事务处理来源/类型）、AutoAccounting（自动会计）、Payment Term（付款条件）、Receipt Class/Method（收款分类/方法）、AutoCash（自动核销）、Credit Management（信用管理）、Collections（催收）和 Revenue Recognition（收入确认）。

典型会计示意：

| 事件 | 借方示例 | 贷方示例 |
| --- | --- | --- |
| AR 发票 | Receivable（应收账款） | Revenue（收入）、Tax（税额） |
| 收款确认 | Cash/Remittance（现金/汇款在途） | Unapplied/Unidentified 或 Receivable |
| 核销 | Unapplied Cash（未核销收款） | Receivable |
| 收款冲销 | 原分录反向或按冲销规则处理 | 原分录反向或按冲销规则处理 |

最低对账：AutoInvoice 控制总额、AR Trial Balance、Aging（账龄）、未识别/未核销收款、汇款在途、坏账/调整、收入和税额与 GL。

详见 [O2C 端到端](../08-e2e/order-to-cash.md)、[AR 交易](../03-ar/transactions.md)、[收款](../03-ar/receipts.md) 和 [催收账龄](../03-ar/collections-aging.md)。

### 5.4 Acquire to Retire（A2R，资产取得到退出）

```text
AP/Projects/手工来源 → Mass Additions（成批增加）
                  → 资产新增/资本化 → 折旧/调整/转移
                  → 报废/恢复 → SLA/GL → 资产与总账对账
```

关键配置：Asset Book（资产账簿）、Category（类别）、Depreciation Method（折旧方法）、Prorate Convention（折旧分摊惯例）、Location（地点）、Distribution（分配）和资产清算/成本/累计折旧/折旧费用科目。

必须区分 Date Placed in Service（DPIS，启用日期）、会计日期、交易日期和折旧期间。折旧完成后对历史期间的调整可能触发追溯调整，不能仅凭界面金额判断影响。

最低对账：Mass Additions 来源与处理状态、资产成本、CIP（在建工程）、累计折旧、本期折旧、报废损益、资产明细与 GL 控制账户。

详见 [固定资产流程](../05-fa/process.md)、[折旧与会计](../05-fa/depreciation-accounting.md) 和 [项目资本化](../05-fa/projects-capitalization.md)。

### 5.5 Inventory/WIP/Cost to GL（库存/在制品/成本到总账）

```text
采购/接收、库存移动、销售发运、WIP 领料/资源/完工
  → Material/Resource Transactions
  → Cost Processor/Cost Distribution
  → SLA/GL → 库存估值、WIP 价值、差异和 COGS 对账
```

功能顾问应掌握 Cost Organization/Inventory Organization（成本/库存组织）、Cost Type（成本类型）、Cost Element（成本要素）、Subelement（子要素）、Standard/Average Costing（标准/平均成本）、WIP Variance（在制品差异）和 COGS Recognition（销售成本确认）。

技术顾问排错时必须区分：业务交易已发生但未成本、成本已计算但未分配、会计事件失败、SLA 未传 GL。直接修改成本处理标志会破坏处理链和审计证据。

详见 [成本会计流](../06-cost/accounting-flow.md)、[成本方法](../06-cost/costing-methods.md) 和 [库存/WIP 到 GL](../08-e2e/inventory-wip-cost-gl.md)。

### 5.6 Cash、Payments 与 Bank Reconciliation（现金、支付与银行对账）

产品边界：

- Payables（AP，应付）确定负债、到期计划和付款选择资格。
- Oracle Payments（IBY，Oracle 支付）编排付款批、格式、支付指令、传输和回执。
- Cash Management（CE，现金管理）管理银行对账单、现金头寸和交易对账。
- General Ledger（GL，总账）保存最终会计和现金账户余额。

Payment Process Request（PPR，付款流程请求）“完成”不必然等于银行已扣款。应分别记录选择、构建、格式化、传输、银行接收、银行接受/拒绝、清算和对账状态。

银行接口必须验证文件级、批次级和交易级控制总额；敏感文件使用批准的传输通道、加密、签名、密钥轮换和最小权限。

详见 [现金管理](../07-ce-tax/cash-management.md)、[现金预测与接口](../07-ce-tax/cash-forecast-interfaces.md) 和 [CE/IBY/EBTax 接口](../07-ce-tax/interfaces.md)。

### 5.7 Tax Determination and Reporting（税务确定与申报）

税务问题按以下顺序定位：Configuration Owner（配置所有者）→ Party Tax Profile/Registration（参与方税务配置/登记）→ Tax Regime/Tax（税制/税种）→ Applicability/Place of Supply（适用性/纳税地点）→ Status/Rate（状态/税率）→ Recovery（抵扣）→ 交易税行 → SLA → Tax Reporting Ledger（税务报告台账）。

不要只检查税率。零税率、免税、不适用、反向征税、自计税和不可抵扣税的业务及会计含义不同，证据要求也不同。

详见 [EBTax](../07-ce-tax/ebtax.md) 和 [税务报告与本地化](../07-ce-tax/tax-reporting-localization.md)。

## 6. 功能顾问的配置方法

### 6.1 推荐配置顺序

```text
范围与法规要求
  → 企业结构/法人/账簿/OU/库存组织
  → COA/日历/币种/汇率/期间
  → 安全、职责、数据访问、审批
  → TCA、供应商、客户、银行、物料等主数据
  → 税务、付款、收款和模块选项
  → SLA/公司间/序列/报表
  → 接口、转换、关账和运维配置
```

顺序不是机械脚本；项目应维护配置依赖矩阵，明确前置对象、配置所有者、迁移工具、环境差异和回退方式。

### 6.2 配置工作簿最低字段

| 字段 | 说明 |
| --- | --- |
| 配置对象和业务目的 | 为什么配置，解决哪个已批准需求 |
| 导航路径与责任 | 在哪个职责/功能下维护 |
| 前置依赖 | 组织、值集、账户、主数据或补丁依赖 |
| 配置值及中文解释 | 记录代码、名称、含义和默认值理由 |
| 环境差异 | DEV、TEST、UAT、PROD 中哪些值不得照搬 |
| 会计/权限/接口影响 | 下游影响和需回归范围 |
| 迁移方式 | 手工、FNDLOAD、API、Loader 或其他受支持工具 |
| 验证证据 | 截图、查询、请求号、报表和测试用例 |
| 回退/补偿 | 如何停用、恢复或用反向交易处理 |

### 6.3 会计设计矩阵

每个模块至少按“业务事件 × 条件 × 借贷行 × 科目来源 × 币种 × 会计日期 × SLA 事件类型 × 冲销方式”记录。必须覆盖正常、冲销、取消、跨期、外币、税、差异、舍入、公司间和异常场景。

### 6.4 不可逆或高影响配置

以下设置在启用交易后通常难以更改或更改代价很高：COA 结构与段语义、账簿核心属性、会计日历、资产账簿与类别关键设置、成本方法、组织归属、主数据编码规则。决策前应完成原型验证、数据量评估、报表样例和正式签字。

## 7. 功能顾问的日常诊断

### 7.1 收集事实

至少记录：实例与补丁级别、用户/职责、组织/账簿/期间、业务单据号和主键、金额/币种、当前状态、预期结果、实际结果、发生时间、并发请求号、相关接口批次和最近变更。

### 7.2 业务状态优先

先在页面和标准报表确认交易是否合法，再查表。常见根因包括：期间未开放、组织上下文错误、审批未完成、Hold 未释放、前置交易未完成、账户组合无效、税务登记/规则不匹配、汇率缺失、并发程序参数或责任权限错误。

### 7.3 用反向交易而不是数据修补

已会计或已过账交易通常应使用产品支持的 Cancel、Reverse、Unapply、Void、Credit Memo、Return、Retirement/Reinstatement 等业务动作纠正。是否允许、在哪个期间纠正及其会计影响，必须按具体产品规则验证。

## 8. 技术架构基础

### 8.1 R12.2 逻辑分层

| 层 | 主要组件 | 技术顾问关注点 |
| --- | --- | --- |
| 客户端/入口 | 浏览器、Forms 客户端、负载均衡 | URL、会话、TLS、代理和客户端兼容性 |
| Web 层 | Oracle HTTP Server（OHS） | 路由、静态内容、反向代理、访问日志 |
| 应用层 | WebLogic、OAF、Forms、Concurrent、Workflow、OPP | Managed Server、JVM、并发队列、通知和输出生成 |
| 数据库层 | Oracle Database、APPS/产品 Schema、EBR | 数据模型、事务、锁、Edition、对象有效性 |
| 外部集成层 | 文件/SFTP、REST/SOAP、SOA/OIC、银行/税务/外围系统 | 契约、安全、幂等、监控和对账 |

R12.2 使用 Online Patching（在线补丁）和 Edition-Based Redefinition（EBR，基于版本的重定义）支持应用层在线修补。技术方案必须区分 Run File System（运行文件系统）、Patch File System（补丁文件系统）和 Non-Editioned File System（非版本化文件系统），并通过 AD Online Patching（`adop`）生命周期发布受影响工件。

### 8.2 常见技术组件中英文对照

- Oracle Application Framework（OAF，Oracle 应用框架）：EBS HTML 页面开发框架。
- Oracle Forms（Forms，表单）：传统 Forms 业务界面与运行时。
- Concurrent Processing（并发处理）：后台批处理、报表和接口任务调度框架。
- Output Post Processor（OPP，输出后处理器）：处理 XML Publisher/BI Publisher 等输出转换。
- Workflow（工作流）：业务流程、通知、后台活动和事件处理。
- Approvals Management Engine（AME，审批管理引擎）：基于规则确定审批链。
- Integrated SOA Gateway（ISG，集成 SOA 网关）：从 Integration Repository 部署和调用受支持服务。
- AutoConfig（自动配置）：根据上下文文件维护许多 EBS 配置文件；不要把手工改生成文件作为长期方案。

## 9. 数据模型与安全查询

### 9.1 对象命名只是线索

| 常见形式 | 通常含义 | 注意事项 |
| --- | --- | --- |
| `_ALL` | 常含组织维度的业务表 | 仍需确认 `ORG_ID`、Ledger 和访问策略 |
| `_B` | 多语言对象的基础表 | 不应把命名规律当成所有产品的硬规则 |
| `_TL` | Translation Table（翻译表） | 连接时关注 `LANGUAGE` 和来源语言 |
| `_VL` | 常见的多语言视图 | 适合展示，性能和列定义需验证 |
| `_INTERFACE` / `_IFACE` | 开放接口或处理接口对象 | 必须配合标准导入程序和错误处理 |
| `_GT` / `_TEMP` | 全局临时或处理临时对象 | 不应作为持久集成或报告来源 |

WHO Columns（WHO 审计列）通常包括创建/更新用户和时间，有些对象还记录登录或请求信息。它们用于基础追溯，但不能替代完整审计设计。

### 9.2 APPS Schema 与产品 Schema

APPS 通常通过同义词和授权提供统一访问入口；产品对象由相应 Schema 拥有。不要在 APPS 中随意创建无治理对象，也不要直接修改 Oracle 拥有的对象。自定义对象的 Schema、授权、同义词、Edition 属性、安装和卸载脚本应在技术设计中明确。

### 9.3 查询准则

1. 只读优先，使用绑定变量。
2. 先从主键或业务单号获得小集合，再连接大表。
3. 大表限定组织、账簿、期间、日期或主键范围。
4. 明确金额币种、精度和换算口径；不要对不同币种直接求和。
5. 使用目标实例的 `ALL_TAB_COLUMNS`、eTRM 和执行计划验证列与索引。
6. 查询敏感字段前确认权限和脱敏要求。

详细数据字典入口见 [模块数据字典](../README.md#模块数据字典) 和 [数据模型](../09-technical/data-model.md)。

## 10. 接口与扩展选型

### 10.1 选型顺序

| 方式 | 适用场景 | 优点 | 主要风险/控制 |
| --- | --- | --- | --- |
| 标准页面/批处理 | 人工或标准批量业务 | 产品校验最完整 | 操作权限、批次证据 |
| Open Interface（开放接口） | 官方提供接口表和导入程序的批量导入 | 适合大批量、错误可分批处理 | 不等于只插接口表；必须运行标准导入并闭环错误 |
| Public PL/SQL API（公开 PL/SQL API） | 实时或受控事务调用 | 复用产品校验与消息栈 | 签名、上下文、事务和版本必须从目标实例确认 |
| Integration Repository / ISG | 已登记并可部署的服务 | 标准化服务发现和治理 | 并非所有 API 都可直接暴露；需验证授权和服务状态 |
| Business Event / Workflow | 事件驱动和异步处理 | 降低耦合 | 重复事件、乱序、Deferred 队列和补偿 |
| Web ADI / Loader | 受控配置或用户批量录入 | 面向业务用户 | 模板版本、权限、数据验证 |
| 自定义 Staging + 标准入口 | 复杂映射、清洗、重试和对账 | 可观测、可恢复 | 自定义状态机与数据保留需要治理 |

禁止把直接 `INSERT/UPDATE/DELETE` Oracle EBS 业务基表当作接口或常规修复方案。

### 10.2 接口最小状态机

```text
RECEIVED → VALIDATED → READY → SUBMITTED → PROCESSING
        → SUCCEEDED
        → REJECTED（业务错误，可修正后重放）
        → FAILED（技术错误，按策略重试）
        → QUARANTINED（超过重试上限，人工处理）
```

每条记录至少保留：来源系统、业务唯一键、批次号、关联号、负载版本、组织/账簿、状态、重试次数、错误分类、请求号、创建/更新时间和目标业务主键。幂等设计应能证明同一业务消息重放不会重复记账或重复付款。

### 10.3 提交成功不等于业务成功

API 返回成功、HTTP 2xx、文件传输成功或并发请求提交成功，只证明一个技术步骤成功。最终验收必须检查目标业务对象、数量/金额、业务状态、会计结果和外部回执。

## 11. Concurrent、Workflow 与页面定制

### 11.1 Concurrent Processing（并发处理）

技术顾问应区分 Executable（可执行）、Concurrent Program（并发程序）、Parameter/Value Set（参数/值集）、Request Group（请求组）、Request Set（请求集）、Manager/Queue（管理器/队列）、Incompatibility（不兼容性）和 Output Post Processor。

诊断顺序：请求参数 → Phase/Status（阶段/状态）→ 请求日志/输出 → 父子请求 → Manager 队列与专业化规则 → 数据库会话/锁/SQL → OPP 或外部依赖。不要仅凭 `FND_CONCURRENT_REQUESTS` 的最终状态断言业务结果。

### 11.2 Workflow 与 AME

Workflow 排错需要保存 Item Type（项目类型）、Item Key（项目键）、Process（流程）、Activity（活动）、Notification（通知）和相关业务键。AME 决定审批人时，还要检查 Transaction Type（事务类型）、Attribute（属性）、Condition（条件）、Action Type（操作类型）、Rule（规则）和审批组。

重复通知、活动 Deferred（延迟）、Background Engine 未处理、Mailer 失败和业务回调错误是不同问题，不能一概通过重试通知解决。

### 11.3 OAF、Forms 与 Personalization

优先级通常为：标准配置 → Personalization（个性化）→ 受支持扩展 → 自定义页面/程序。每项定制都应记录 CEMLI 分类、业务依据、标准对象依赖、权限、版本、迁移方式、回归范围和退役条件。禁止直接覆盖 Oracle 标准文件。

## 12. R12.2 发布、ADOP 与 EBR

### 12.1 ADOP 基本周期

| 阶段 | 中文说明 | 顾问关注点 |
| --- | --- | --- |
| Prepare | 准备 | 创建/同步补丁版本环境，检查系统健康和前置条件 |
| Apply | 应用 | 在补丁版本应用补丁或自定义发布工件，记录 Worker/失败信息 |
| Finalize | 最终化 | 完成切换前准备，降低 Cutover 停机工作量 |
| Cutover | 切换 | 停止相关服务并切换 Run/Patch 角色；需要正式窗口和业务验证 |
| Cleanup | 清理 | 清理旧版本数据和对象；按维护策略选择清理方式 |

`fs_clone`、Abort、Restart 和其他维护动作有明确前提，必须依据当前周期状态和官方维护指南执行，不应把网上命令直接复制到生产。

### 12.2 自定义对象的 EBR 合规

发布前至少确认：对象是否 Editioned（版本化）或 Noneditioned（非版本化）、依赖是否跨 Edition、同义词/授权是否正确、数据库对象是否有效、文件部署位置是否正确、Patch 文件系统是否同步、Cutover 后 Smoke Test（冒烟测试）是否覆盖关键财务路径。

功能顾问也应参与补丁回归：登录与职责、关键主数据、AP/AR/GL/FA/CE/CST 代表性交易、Create Accounting、传 GL、核心报表、银行/税务/外围接口和审批通知。

## 13. 测试、迁移和切换

### 13.1 测试层次

| 测试 | 中文说明 | 财务重点 |
| --- | --- | --- |
| Unit Test | 单元测试 | 单一配置、API、程序或报表正确 |
| CRP | 会议室演练 | 用原型验证流程和蓝图决策 |
| SIT | 系统集成测试 | 跨模块/跨系统状态、错误和重跑 |
| UAT | 用户验收测试 | 业务可用性、控制、会计、报表和职责 |
| Regression Test | 回归测试 | 补丁/变更未破坏既有关键流程 |
| Performance Test | 性能测试 | 代表性数据量、并发量、关账窗口和批处理时长 |
| Security Test | 安全测试 | 最小权限、职责分离、敏感数据和接口认证 |

每个财务测试用例至少断言：业务状态、数量、交易币/本位币金额、税、会计日期、SLA 分录、GL 传输/过账、报表展示、冲销结果和审计证据。

### 13.2 数据迁移分层

```text
配置/参考数据 → 主数据 → 期初余额 → 未结交易 → 必要历史数据
```

每一层都需要 Extract（抽取）、Profile（剖析）、Cleanse（清洗）、Map（映射）、Transform（转换）、Load（装载）、Reconcile（对账）和 Sign-off（签字）。至少完成多轮 Mock Conversion（模拟转换），记录批次耗时、错误率、人工处理量和最终切换窗口。

### 13.3 Cutover（切换）最低控制

- 冻结范围、最终增量、责任人、前后依赖和 Go/No-Go 条件明确。
- 每个装载和接口步骤有控制总额、开始/结束时间、请求号和重跑边界。
- 余额、未结 AP/AR、库存、资产、项目、银行和 GL 按批准口径对账。
- 回退方案区分技术回退与业务补偿；进入不可逆点前必须再次审批。
- 上线后执行 Daily Reconciliation（每日对账）和缺陷分级，直至移交标准满足。

## 14. 月结与运行管理

### 14.1 典型关账依赖

```text
采购/接收、库存/WIP/成本、项目
        → AP/AR/FA/CE 等子账处理完成
        → Create Accounting / Transfer to GL
        → 子账与 GL 对账
        → GL 日记账、重估、折算、公司间、合并
        → 财务与管理报告 → 关期与签字
```

实际顺序受产品、会计政策和企业批处理设计影响。关账清单必须包含负责人、截止时间、前置条件、程序参数、成功标准、异常升级、输出证据和重开期间审批。

### 14.2 运行指标

- 接口：接收量、成功/拒绝/在途量、金额、最老在途时间、重试次数。
- 并发：排队时间、运行时间、失败率、关键请求是否错过窗口。
- Workflow：Deferred 积压、错误活动、通知延迟和 Mailer 状态。
- 会计：未处理事件、会计错误、未传 GL、未导入和未过账数量/金额。
- 关账：按模块完成时间、对账差异、手工调整和重开次数。
- 基础设施：服务可用性、JVM/OPP、数据库会话、存储、证书和批处理容量。

## 15. 通用故障排查与证据包

### 15.1 五类根因

| 根因类别 | 例子 | 首选动作 |
| --- | --- | --- |
| 业务状态 | 未审批、Hold、前置交易未完成 | 按产品流程完成或合法冲销 |
| 配置/主数据 | 账户无效、期间关闭、税率或银行用途错误 | 修正配置并评估已发生交易影响 |
| 数据/接口 | 重复键、映射错误、控制总额不平 | 隔离批次、修正来源、按幂等策略重放 |
| 技术运行 | Manager、Workflow、OPP、网络或证书失败 | 恢复服务后从受支持断点重启 |
| 缺陷/补丁 | 标准代码错误或补丁冲突 | 最小复现、查官方资料/MOS、按批准方案修复 |

### 15.2 标准证据包

提交内部升级或 MOS Service Request（服务请求）前，准备：

- EBS、AD/TXK、数据库和相关产品补丁级别。
- 发生时间、时区、节点、用户/职责、组织/账簿/期间。
- 可脱敏的业务主键、接口批次、请求号、Workflow Item Key。
- 清晰的重现步骤、预期/实际结果和影响范围。
- 相关日志中首个有意义的错误，而非只截最后一行。
- 只读诊断结果、配置截图、近期变更和非生产复现情况。
- 已尝试动作及其结果；不要反复执行可能重复业务的步骤。

## 16. 项目交付物检查表

### 16.1 功能顾问

- 已批准的业务流程和例外流程。
- 企业结构、COA、会计日历、币种和安全设计。
- 配置工作簿、主数据规则、会计事件矩阵。
- 报表清单、对账公式、关账日历和控制证据。
- CRP/SIT/UAT 用例、缺陷决定和用户培训材料。
- 数据迁移映射、切换清单、Hypercare 和支持移交。

### 16.2 技术顾问

- CEMLI 清单与逐项技术设计。
- 接口契约、状态机、幂等/重试/补偿、对账和监控。
- 数据模型、对象依赖、权限、敏感数据和容量评估。
- 源码、数据库/文件部署工件、ADOP/EBR 合规说明和回退步骤。
- 单元/SIT/性能/安全测试证据和运行手册。
- 日志规范、告警、支持知识、版本记录和退役方案。

## 17. 延伸阅读

- [按角色阅读路径](reading-paths-by-role.md)
- [财务产品地图与边界](financials-product-map.md)
- [范围、版本与适用性](scope-and-version.md)
- [生产安全与支持边界](safety-and-production-boundaries.md)
- [中英文术语与缩略语](../90-reference/glossary-and-acronyms.md)
- [Oracle 官方资料与验证顺序](official-sources.md)
- [模块接口实现手册索引](../README.md#模块接口实现手册)
