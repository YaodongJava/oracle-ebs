# Oracle E-Business Suite R12.2.x 财务业务与技术知识库

本仓库面向 Oracle EBS R12.2.x 财务功能顾问、技术顾问、集成顾问、实施经理、测试人员和生产运维人员，规划覆盖核心财务、扩展财务、供应链财务边界、端到端流程、技术开发及实施运维全生命周期。

> 目标知识域已建立首批实际文档入口；现有文档仍不移动、不重命名，后续按专题逐步迁移、深化并建立权威正文链接。

## 快速入口

- [现有 R12.2.x 知识图谱、阅读路径与 SQL 约定](docs/README.md)
- [财务公共基础](docs/01-common/organization.md)
- [应付账款 AP](docs/02-ap/process.md)
- [应收账款 AR](docs/03-ar/process.md)
- [总账 GL](docs/04-gl/process.md)
- [固定资产 FA](docs/05-fa/process.md)
- [库存、WIP 与成本](docs/06-cost/accounting-flow.md)
- [现金管理与 E-Business Tax](docs/07-ce-tax/cash-management.md)
- [端到端业务流程](docs/08-e2e/procure-to-pay.md)
- [技术、接口与运维](docs/09-technical/integration.md)

## 目录状态

| 标记 | 含义 |
| --- | --- |
| ✅ 已存在 | 当前仓库已有内容，可直接访问 |
| 🔀 待迁移 | 已有内容，后续按目标结构调整归属 |
| 🚧 在建 | 已开始整理，但尚未完成评审 |
| ⬜ 规划 | 目录蓝图，暂不创建空文件 |
| 🧩 可选产品 | 依赖许可证、已安装产品或企业适用范围 |

文档成熟度与目录建设状态分开管理。后续单篇文档可使用 `draft`、`reviewed`、`verified`、`deprecated`；其中 `verified` 表示已对照 Oracle R12.2 官方资料并在明确补丁级别的实例验证，仍不能替代生产变更审批。

## 信息架构原则

1. 根 README 只负责产品全景和一级知识域导航，不长期平铺全部叶子文档。
2. 每个知识域设置 `README.md`，负责子模块导航；每个产品子模块再维护自己的专题清单。
3. 目录最多采用“知识域/产品子域/专题文档”三级结构，避免过深嵌套。
4. 一个文档只保留一个主主题，避免 `close-reports-interfaces.md` 一类混合文件继续膨胀。
5. 通用原理只维护一份权威正文，其他模块通过链接引用，避免 SLA、TCA、IBY、EBTax、接口和 SQL 重复。
6. 核心财务与可选产品分开标识，避免暗示所有 EBS 实例都安装 Treasury、Loans、FAH、Property Manager 等产品。
7. 技术内容必须体现 R12.2 的 WebLogic、双文件系统、Edition-Based Redefinition 和 Online Patching 边界。
8. 不直接 DML Oracle EBS 业务基表；写入示例优先使用公开 API、标准 Open Interface 和标准并发程序。

## 目标知识域总览

| 编号 | 目标目录 | 知识域 | 主要范围 | 当前状态 |
| --- | --- | --- | --- | --- |
| 00 | `docs/00-guide/` | 导航与知识库治理 | 阅读路径、版本、术语、规范、官方资料 | 🚧 |
| 01 | `docs/01-foundation/` | 财务公共基础 | 企业结构、Ledger、MOAC、COA、TCA、银行模型、审批 | 🚧 |
| 02 | `docs/02-record-to-report/` | Record to Report | GL、SLA、FAH、AGIS、预算、合并 | 🚧 |
| 03 | `docs/03-procure-to-pay/` | Procure to Pay | 供应商、采购收货、AP、IBY、iExpenses | 🚧 |
| 04 | `docs/04-credit-to-cash/` | Credit to Cash | TCA、AR、信用、催收、iReceivables、Loans | 🚧 |
| 05 | `docs/05-assets-projects/` | 资产、项目与资本化 | FA、iAssets、Projects、Grants、Property、Lease | 🚧 |
| 06 | `docs/06-cash-tax/` | 现金、资金与税务 | CE、Treasury、EBTax、银行接口、税务报告 | 🚧 |
| 07 | `docs/07-cost-accounting/` | 供应链财务与成本 | PO/RCV、INV、WIP、CST、OPM、LCM、COGS | 🚧 |
| 08 | `docs/08-reporting-governance/` | 报表、关账与治理 | 财务报告、ECC、对账、内控、审计、本地化 | 🚧 |
| 09 | `docs/09-end-to-end/` | 端到端流程 | R2R、P2P、O2C、A2R、项目、税务、公司间 | 🚧 |
| 10 | `docs/10-technical/` | 技术架构与开发 | 数据模型、接口、Concurrent、Workflow、OAF、Forms、EBR | 🚧 |
| 11 | `docs/11-implementation-operations/` | 实施与运维生命周期 | 蓝图、迁移、测试、切换、运维、性能、升级、灾备 | 🚧 |
| 90 | `docs/90-reference/` | 统一索引 | 表、SQL、API、并发程序、Profile、Lookup、错误码 | 🚧 |
| 99 | `docs/99-archive/` | 历史归档 | 已废弃或仅用于历史追溯的内容 | 🚧 |

## 完整目标目录蓝图

```text
docs/
├── 00-guide/
│   ├── README.md
│   ├── scope-and-version.md
│   ├── financials-product-map.md
│   ├── reading-paths-by-role.md
│   ├── reading-paths-by-lifecycle.md
│   ├── documentation-conventions.md
│   ├── safety-and-production-boundaries.md
│   └── official-sources.md
│
├── 01-foundation/
│   ├── README.md
│   ├── enterprise-organization/
│   ├── legal-entity-configurator/
│   ├── accounting-setup-manager/
│   ├── ledger-and-ledger-set/
│   ├── coa-and-flexfields/
│   ├── calendar-currency-period/
│   ├── multi-org-and-moac/
│   ├── tca-party-model/
│   ├── centralized-bank-account-model/
│   ├── workflow-and-ame/
│   ├── security-and-data-access/
│   ├── document-accounting-sequencing/
│   ├── attachments-dff-lookups/
│   └── master-data-governance/
│
├── 02-record-to-report/
│   ├── README.md
│   ├── general-ledger/
│   ├── subledger-accounting/
│   ├── financials-accounting-hub/
│   ├── agis-intercompany/
│   ├── budgetary-control/
│   ├── consolidation-and-elimination/
│   ├── secondary-ledger-reporting-currency/
│   └── record-to-report-close/
│
├── 03-procure-to-pay/
│   ├── README.md
│   ├── supplier-master/
│   ├── purchasing-and-iprocurement/
│   ├── receiving-and-accrual/
│   ├── payables/
│   ├── payments-iby/
│   ├── internet-expenses/
│   ├── isupplier-and-supplier-management/
│   └── procure-to-pay-controls/
│
├── 04-credit-to-cash/
│   ├── README.md
│   ├── tca-customer-master/
│   ├── order-shipping-boundary/
│   ├── receivables/
│   ├── credit-management/
│   ├── advanced-collections/
│   ├── ireceivables-and-bill-presentment/
│   ├── deductions-and-disputes/
│   ├── loans/
│   └── credit-to-cash-controls/
│
├── 05-assets-projects/
│   ├── README.md
│   ├── fixed-assets/
│   ├── iassets/
│   ├── asset-tracking-and-eam/
│   ├── projects-foundation/
│   ├── project-costing/
│   ├── project-billing/
│   ├── project-planning-control/
│   ├── project-contracts/
│   ├── grants-accounting/
│   ├── project-to-asset/
│   ├── property-manager/
│   └── lease-and-finance-management/
│
├── 06-cash-tax/
│   ├── README.md
│   ├── cash-management/
│   ├── treasury/
│   ├── e-business-tax/
│   ├── tax-reporting-ledger/
│   ├── bank-statement-integration/
│   ├── cash-position-and-forecast/
│   ├── bank-account-transfer-and-pooling/
│   └── tax-engines-and-external-services/
│
├── 07-cost-accounting/
│   ├── README.md
│   ├── purchasing-receiving-accounting/
│   ├── inventory-accounting/
│   ├── discrete-cost-management/
│   ├── work-in-process-accounting/
│   ├── process-manufacturing-costing/
│   ├── landed-cost-management/
│   ├── cogs-and-revenue-matching/
│   ├── project-manufacturing/
│   ├── eam-cost-and-capitalization/
│   └── scm-to-sla-gl/
│
├── 08-reporting-governance/
│   ├── README.md
│   ├── period-close-and-reconciliation/
│   ├── financial-reporting/
│   ├── management-reporting/
│   ├── bi-publisher-and-rxi/
│   ├── fsg-smart-view-webadi/
│   ├── enterprise-command-center/
│   ├── internal-controls-and-sox/
│   ├── audit-and-data-retention/
│   ├── localizations-and-regulatory/
│   └── public-sector-and-federal/
│
├── 09-end-to-end/
│   ├── README.md
│   ├── record-to-report.md
│   ├── procure-to-pay.md
│   ├── order-to-cash.md
│   ├── credit-to-cash.md
│   ├── acquire-to-retire.md
│   ├── expense-to-reimbursement.md
│   ├── project-to-cash.md
│   ├── project-to-asset.md
│   ├── inventory-wip-to-gl.md
│   ├── bank-statement-to-reconciliation.md
│   ├── tax-determination-to-reporting.md
│   ├── intercompany-to-elimination.md
│   ├── payroll-to-gl.md
│   ├── budget-to-control.md
│   ├── external-subledger-to-fah.md
│   └── close-to-report.md
│
├── 10-technical/
│   ├── README.md
│   ├── architecture/
│   ├── data-model/
│   ├── database-development/
│   ├── integration/
│   ├── concurrent-processing/
│   ├── workflow-and-ame/
│   ├── oaf/
│   ├── forms/
│   ├── reporting-development/
│   ├── configuration-migration/
│   ├── adop-and-ebr/
│   ├── database-platform/
│   ├── middleware/
│   ├── performance/
│   ├── security/
│   ├── audit-and-compliance/
│   ├── testing-and-automation/
│   └── release-engineering/
│
├── 11-implementation-operations/
│   ├── README.md
│   ├── assessment-scope-license/
│   ├── solution-blueprint/
│   ├── setup-sequence-and-workbooks/
│   ├── data-migration-and-conversion/
│   ├── integration-delivery/
│   ├── testing-strategy/
│   ├── cutover-and-rollback/
│   ├── hypercare-and-support-transition/
│   ├── period-close-operations/
│   ├── monitoring-and-diagnostics/
│   ├── incident-problem-change/
│   ├── patching-and-upgrade/
│   ├── cloning-and-refresh/
│   ├── backup-recovery-dr/
│   └── archive-purge-capacity/
│
├── 90-reference/
│   ├── README.md
│   ├── glossary-and-acronyms.md
│   ├── product-prefix-index.md
│   ├── data-dictionary-index.md
│   ├── table-status-index.md
│   ├── sql-index.md
│   ├── interface-api-index.md
│   ├── concurrent-program-index.md
│   ├── profile-option-index.md
│   ├── lookup-index.md
│   ├── accounting-event-index.md
│   ├── report-index.md
│   ├── menu-responsibility-index.md
│   ├── error-message-index.md
│   ├── patch-version-matrix.md
│   ├── mos-note-index.md
│   └── official-document-index.md
│
└── 99-archive/
    └── README.md
```

## 财务公共基础规划

| 子域 | 计划覆盖内容 |
| --- | --- |
| Enterprise Organization | Business Group、Legal Entity、Establishment、Ledger、OU、Inventory Organization |
| Accounting Setup Manager | Primary/Secondary Ledger、Reporting Currency、Ledger Set、Accounting Options |
| COA 与 Flexfields | Balancing/Management/Cost Center Segment、KFF、DFF、Value Set、CVR、安全规则 |
| 日历、币种和期间 | Accounting Calendar、Period Type、Currency、Rate Type、Daily Rates、Open/Close |
| Multiple Organizations | MOAC、Security Profile、Operating Unit Context、跨组织数据访问 |
| TCA 公共模型 | Party、Account、Site、Relationship、Supplier/Customer 的共享与差异 |
| 集中式银行模型 | Bank、Branch、Internal/External Account、Account Owner/Use、权限 |
| 公共控制 | Document Sequence、Accounting Sequence、Approval、AME、Attachment、Lookup、Profile |
| 主数据治理 | 编码、唯一性、有效期、共享服务、数据所有者、变更审批和审计 |

## Record to Report 规划

### General Ledger

- Ledger、Ledger Set、Reporting Currency 和 Data Access Set。
- Journal Source、Category、Batch、Header、Line、Approval、AutoPost 和 Reversal。
- Recurring Journal、MassAllocation、Statistical Journal、Suspense 和 Balancing。
- Budget、Encumbrance、Budgetary Control 和 Funds Availability。
- Revaluation、Translation、Consolidation、Elimination 和历史汇率。
- Journal Import、Web ADI、FSG、Smart View、月结、年结、表结构、SQL 与排错。

### Subledger Accounting

- Transaction Entity、Event、Event Class、Event Type 和 Accounting Event 生命周期。
- Accounting Method Builder、AAD、JLD、JLT、ADR、Mapping Set、Description Rule。
- Supporting Reference、Multiperiod Accounting、Rounding、Gain/Loss 和冲销。
- Create Accounting 的 Draft、Final、Final Post、Transfer to GL 和 Journal Import。
- AP、AR、FA、Projects、Costing 到 XLA/GL 的 Drilldown、诊断、性能和对账。

### Financials Accounting Hub 与 AGIS

- 🧩 外部子账交易、Accounting Hub 事件模型、来源、接口、会计规则和 GL 传递。
- AGIS Organization、交易类型、账户规则、Inbound/Outbound Batch、审批与安全。
- 跨 Ledger/Legal Entity 交易、AP/AR 发票生成、Netting、Elimination 和差异对账。

## Procure to Pay 规划

### Supplier、Purchasing 与 Receiving

- Supplier/TCA、Supplier Site、税务、银行、分类、审批和重复检查。
- Requisition、RFQ、Quotation、PO、Blanket、Contract、Release、Change Order。
- 2-way/3-way/4-way Match、Receipt、Return、Correction、Inspection、ERS。
- Receipt Accrual、Period-End Accrual、Accrual Reconciliation 和采购会计边界。
- iProcurement、iSupplier、Supplier Management 和 Services Procurement。

### Payables

- AP Options、Financial Options、Payment Terms、Tolerance 和 Distribution Set。
- Standard/Credit/Debit/Prepayment/Expense/Payment Request/Retainage Invoice。
- Invoice Line/Distribution、Validation、Hold、Approval、Matching 和税务。
- Prepayment Apply/Unapply、Interest、Recurring Invoice、Withholding、Tax Recovery。
- SLA、Trial Balance、Liability Reconciliation、Period Close、Interface/API、表和 SQL。

### Oracle Payments 与 Internet Expenses

- IBY Funds Disbursement/Funds Capture、Payment Method、Process Profile、Format、Document。
- Payment Process Request、Proposed Payment、Instruction、File、Transmission、ACK、Positive Pay。
- EFT、ACH、SEPA、Check、Direct Debit、Credit Card、Stop/Void/Reissue/Refund。
- SFTP、PGP、签名、证书、密钥、银行回执、支付安全和职责分离。
- iExpenses Policy、Expense Type、Per Diem、Mileage、Cash Advance、Corporate Card。
- Expense Approval/Audit、AP Import、员工付款、项目费用、移动端和接口。

## Credit to Cash 规划

### Receivables 与 TCA

- Customer、Account、Site、Contact、Profile、Collector、Credit Classification。
- Transaction Source/Type、AutoAccounting、Invoice/Credit/Debit Memo/Chargeback。
- Accounting/Invoicing Rule、Revenue Recognition、Payment Terms 和 Installment。
- Receipt Class/Method/Source、Receipt、Application、Unapplication、Reversal、Remittance。
- AutoInvoice、AutoLockbox、AutoCash、Bills Receivable、Refund、Adjustment 和 Write-off。
- SLA、Aging、Trial Balance、Period Close、接口、数据模型、SQL 和排错。

### Credit、Collections 与客户自助

- 🧩 Credit Review、Case Folder、Credit Check、Credit Limit 和 OM 集成。
- 🧩 Delinquency、Strategy、Work Item、Collector Queue、Promise to Pay、Dunning。
- Dispute、Deductions、Late Charge、Bad Debt、Write-off 和 Recovery。
- 🧩 iReceivables、Bill Presentment、Online Payment 和客户自助服务。
- 🧩 Oracle Loans 的 Origination、Servicing、Billing、Interest、Receipt 和 Accounting。

## Assets、Projects 与资本化规划

### Fixed Assets

- Book、Category、Asset Key、Location、Distribution、Corporate/Tax Book。
- Manual/Mass Addition、AP/Projects/CIP 来源、Capitalization 和 Unitization。
- Adjustment、Transfer、Reclassification、Impairment、Revaluation 和 Physical Inventory。
- Depreciation Method、Convention、Prorate、Bonus、Group 和 Tax Depreciation。
- Retirement、Reinstatement、SLA、GL、Mass Additions、迁移、月结和资产对账。
- 🧩 iAssets、Asset Tracking、eAM 设备与成本资本化。

### Oracle Projects

- Project、Task、Organization、Classification、安全、角色和项目模板。
- Project Costing：Labor、Expense、Supplier、Usage、Miscellaneous、Burden、Cost Distribution。
- Project Billing：Agreement、Funding、Revenue、Invoice、Billing Method、Rate 和 AR Interface。
- Budget、Forecast、Commitment、Budgetary Control、Planning、Resource 和 Portfolio。
- Capital Project、Project Asset、CIP、Interface Assets、Mass Additions 和 FA 对账。
- 🧩 Project Contracts、Grants Accounting、Project Manufacturing 和跨组织项目。

### Property 与 Lease

- 🧩 Property、Space、Lease、Term、Milestone、Payment/Receipt 和 AP/AR 集成。
- 🧩 Lease and Finance Management 的 Contract、Asset、Booking、Billing、Investor、Termination。
- Property/Lease 与 FA、Projects、AP、AR、Payments、GL 的业务与会计衔接。

## Cash、Treasury 与 E-Business Tax 规划

### Cash Management

- Bank/Branch/Account、Owner/Use、权限、Statement Header/Line 和 Transaction Code。
- Manual/Electronic Statement、BAI2、MT940、CAMT、Bank Statement Import。
- AutoReconciliation、匹配规则、AP/AR/Payroll/Treasury/GL 对账和 Cash Clearing。
- Cash Position、Forecast、Pool、Zero Balance、Bank Transfer 和 External Transaction。
- Reconciliation Open Interface、未达账项、银行余额、接口、SQL 和排错。

### Treasury

- 🧩 Company、Counterparty、Dealer、Limit、Settlement Account 和授权。
- 🧩 Money Market、Foreign Exchange、Securities、Derivatives、Exposure 和 Hedge。
- 🧩 Market Data、Yield Curve、Settlement、Confirmation、Accrual、Revaluation 和 Fair Value。
- Treasury 与 CE、Payments、AP、AR、GL 的接口、会计和对账。

### E-Business Tax

- Configuration Owner、Tax Regime、Tax、Status、Rate、Jurisdiction 和 Zone。
- Party Tax Profile、Registration、Fiscal Classification、Determining Factor 和 Condition Set。
- Applicability、Place of Supply、Rate Determination、Recovery、Offset、Exemption、Exception。
- Inclusive/Exclusive、Compound、Self-Assessed、Reverse Charge 和 Withholding Tax。
- AP、AR、PO、OM、Projects、Assets 税务处理、ZX 数据模型和诊断。
- Tax Reporting Ledger、法定申报、外部税引擎、本地化、接口和性能。

## 供应链财务与成本规划

- Purchasing/Receiving Accrual、Invoice Price/Exchange Rate Variance 和 Accrual Reconciliation。
- Inventory Organization、Material Transaction、Subinventory、Locator、Lot/Serial 和库存估值。
- Standard、Average、FIFO/LIFO、Cost Type、Element、Subelement、Rollup 和 Cost Update。
- WIP Job、Material、Resource、OSP、Overhead、Completion、Scrap、Close 和 Variance。
- 🧩 OPM Process Costing、Landed Cost Management 和 In-Memory Cost Management。
- COGS Recognition、Revenue Matching、Drop Ship、Consigned Inventory、Internal Transfer。
- PO/RCV/INV/WIP/CST/OM 到 SLA/GL 的接口、数据链、月结、对账和排错。

## 报表、关账、治理与全球合规规划

| 子域 | 计划覆盖内容 |
| --- | --- |
| Close & Reconciliation | AP、AR、CE、FA、PA、CST、SLA、GL 关账顺序、依赖、对账和签字 |
| Financial Reporting | FSG、Trial Balance、Account Analysis、RXi、标准财务与法定报表 |
| Reporting Technology | BI Publisher、Web ADI、Smart View、Oracle Reports、Discoverer 迁移 |
| Analytics | Enterprise Command Center、KPI、Dashboard、Drilldown、数据加载和安全 |
| Internal Controls | SOX、SoD、审批、重复付款、异常日记账、配置审计和证据留存 |
| Audit & Retention | AuditTrail、Sign-On Audit、WHO、归档、Purge、数据保留和隐私 |
| Localizations | 中国金税、亚太、印度 GST、欧洲 VAT/SEPA、Americas、1099、拉美 |
| Public Sector | 🧩 Fund/Commitment Accounting、Federal Financials、公共部门预算与报告 |

## 端到端流程规划

每条端到端流程统一覆盖：业务状态、单据链、主键链、配置依赖、会计分录、接口点、批处理、控制点、数量/金额对账、期间边界和故障定位。

| 流程 | 关键模块 |
| --- | --- |
| Record to Report | 子账、SLA、GL、合并、报表和关账 |
| Procure to Pay | Supplier、PO、Receiving、AP、IBY、CE、SLA、GL |
| Order/Credit to Cash | OM、Shipping、AR、Credit、Collections、IBY、CE、GL |
| Acquire to Retire | PO/AP、Projects/CIP、FA、Depreciation、Retirement、GL |
| Expense to Reimbursement | HR、iExpenses、Corporate Card、AP、IBY、Projects |
| Project to Cash | Project Costing、Billing、AR、Receipt、Revenue、GL |
| Inventory/WIP to GL | RCV、INV、WIP、CST、SLA、GL |
| Bank to Reconciliation | Bank File、CE、AP/AR/Treasury、Clearing、GL |
| Tax to Report | Transaction Tax、EBTax、SLA、TRL、法定申报 |
| Intercompany to Elimination | AGIS、AP/AR、SLA、GL、Consolidation |
| Payroll to GL | Payroll、Costing、SLA/GL、付款和银行对账 |
| External Subledger to FAH | 外部交易、FAH、SLA、Journal Import、GL |

## 技术架构与开发规划

### `architecture/`

- R12.2 总体架构、请求链路、数据库层、应用层、客户端和外部服务。
- Run/Patch/Non-Editioned File System、WebLogic、OHS、Forms、OAF、Concurrent、Workflow。
- AutoConfig、Context、Environment、Service、Port、Log、Multi-node、HA 和 Load Balancer。

### `data-model/` 与 `database-development/`

- APPS/产品 Schema、Synonym、Editioning View、表/视图后缀、WHO 和主外键规范。
- FND、XLA、TCA、IBY、ZX 共享模型、Flexfield、Lookup、Profile 和数据血缘。
- PL/SQL 标准、FND API、Message Stack、Context、事务、锁、异常、日志和批处理性能。
- Custom Schema、Grant、Synonym、只读边界、受支持数据变更和 NLS/精度处理。

### `integration/`

- Open Interface、Public PL/SQL API、Integration Repository、ISG REST/SOAP。
- Concurrent Program Service、Workflow Business Event、XML Gateway、EDI、Web ADI。
- SOA/OIC/EBS Adapter、文件/SFTP/PGP、MQ、Webhook 和外部服务调用。
- Staging、幂等、重试、补偿、相关号、Outbox、对账、监控和 Dead Letter。
- OAuth/Token、TLS、证书、密钥、最小权限、脱敏和接口审计。

### `concurrent-processing/` 与 `workflow-and-ame/`

- Executable、Program、Parameter、Value Set、Request Set、Stage、不兼容和专业化规则。
- Submit/Wait/Cancel API、Manager、Queue、Priority、Schedule、Capacity、OPP 和 ADZDPATCH。
- Workflow Item/Process/Activity、Builder、Function、Notification、Mailer、Background Engine。
- AME、Business Event、Deferred Agent Listener、部署、监控、Retry、Purge 和排错。

### `oaf/`、`forms/` 与 `reporting-development/`

- OAF MVC、EO/VO/AM/CO、Page/Region/Item、Personalization、Extension、Security 和 Debug。
- Forms Template、PLL、Personalization、Function/Menu、Flexfield/LOV、Compile、Deploy 和 Trace。
- BI Publisher Data Definition/Template、RTF/PDF/XSL-FO/eText、Bursting、Delivery 和 API。
- FSG、Smart View、Web ADI、Oracle Reports、字体、NLS、打印和 OPP 性能。

### `configuration-migration/`、`adop-and-ebr/` 与 `release-engineering/`

- FNDLOAD、WFLOAD、XDOLoader、BNE Loader、OAF/Forms Personalization 迁移。
- Editioned/Noneditioned Object、Editioning View、Cross-Edition Trigger 和 EBR 合规。
- ADOP Prepare/Apply/Finalize/Cutover/Cleanup、fs_clone、Delta Sync、Abort 和恢复。
- Git、代码评审、依赖、静态检查、自定义 Patch Driver、环境晋级、回退和变更证据。

### `database-platform/`、`middleware/` 与 `performance/`

- Schema、Tablespace、Storage、参数、Statistics、Session、Lock、Undo/Temp/Redo 和 Invalid Object。
- RMAN、RAC、Data Guard、Rapid Clone、Refresh、Purge、Archive 和 Capacity。
- OHS/WebLogic/Node Manager/Managed Server、JDBC、JVM、GC、Heap/Thread Dump、TLS/Wallet。
- AWR/ASH/SQL Monitor 许可证边界、Trace/TKPROF、SQL Plan、Concurrent/Workflow/OAF/OPP 调优。

### `security/`、`audit-and-compliance/` 与 `testing-and-automation/`

- User、Responsibility、Menu、Function、Request Group、RBAC/UMX、MOAC 和 Data Security。
- TLS、SSO、DMZ、Password、Database Grant、Secure Configuration Console 和 CPU。
- AuditTrail、Sign-On Audit、FGA、Document Sequence、SoD、隐私、保留和脱敏。
- PL/SQL 单测、接口契约、OATS/OpenScript/Flow Builder、财务回归、Patch Smoke Test。
- 对账断言、会计断言、性能/安全测试、测试数据和自动化发布流水线。

## 实施与运维生命周期规划

| 阶段 | 计划覆盖内容 |
| --- | --- |
| Assessment | 现状评估、范围、许可证、版本、组织与数据量、CEMLI 清单 |
| Blueprint | 业务蓝图、产品边界、企业结构、COA、会计、集成、报表和控制设计 |
| Configuration | 配置顺序、依赖、工作簿、基线、迁移工具和环境对比 |
| Data Migration | Profile、清洗、映射、主数据、余额、未结交易、历史和多轮 Mock |
| Integration Delivery | 契约、开发、单测、联调、容量、安全、Cutover 和 Support Model |
| Testing | CRP、SIT、UAT、Regression、Performance、Security 和 Accounting Validation |
| Cutover | Freeze、Delta、Reconciliation、Go/No-Go、Rollback 和业务签字 |
| Hypercare | 监控、缺陷分级、Daily Reconciliation、知识转移和支持交接 |
| Operations | 日/周/月巡检、批处理、接口、关账、日志、容量、证书和归档 |
| Incident/Problem/Change | 事件、问题、变更、RCA、MOS SR 证据和已知错误库 |
| Patching/Upgrade | RUP、AD/TXK、CPU、One-off、ETCC、冲突、升级演练和回归 |
| Resilience | Clone、Refresh、Masking、Backup、Restore、Data Guard 和 DR Drill |

## 产品子模块统一文档骨架

后续有实际内容时再创建对应文件；不是每个模块都必须机械创建全部空文档。

```text
<product-module>/
├── README.md
├── overview.md
├── business-process.md
├── setup.md
├── transactions.md
├── accounting.md
├── integration.md
├── reports.md
├── reconciliation.md
├── period-close.md
├── data-model.md
├── sql.md
├── troubleshooting.md
├── implementation-cases.md
└── official-references.md
```

统一阅读顺序：

```text
概念 → 业务流程 → 配置 → 交易 → 会计/SLA → 关账/对账 → 报表/内控
     → 数据模型 → SQL → 接口/API → 排错 → 实施案例 → 官方依据
```

## 内容权威归属与去重规则

| 内容 | 唯一权威位置 | 其他模块处理方式 |
| --- | --- | --- |
| 企业结构、MOAC、TCA、银行公共模型 | `01-foundation` | 业务模块只写自身用法并链接公共原理 |
| SLA 通用事件和 AMB | `02-record-to-report/subledger-accounting` | AP/AR/FA/PA/CST 只写模块事件与分录 |
| 模块接口表、公开 API 和错误 | 对应产品的 `integration.md` | 技术目录只维护接口选型与通用治理 |
| 跨系统编排、相关号和补偿 | `09-end-to-end` | 模块只描述自身入口、出口和状态 |
| 模块表结构与 SQL | 模块 `data-model.md`、`sql.md` | `90-reference` 只建索引，不复制正文 |
| 模块结账步骤 | 模块 `period-close.md` | E2E 只维护顺序、依赖和控制点 |
| 报表工具原理 | `08-reporting-governance` 或 `10-technical/reporting-development` | 模块只维护业务报表目录和解释 |
| Concurrent、Workflow、OAF、Forms、EBR | `10-technical` | 模块仅记录产品特有实现或故障 |
| 实施、迁移、测试和切换 | `11-implementation-operations` | 产品模块链接对应生命周期方法 |
| 状态字典、API、Profile、Lookup、错误码 | `90-reference` | 只做入口索引，正文留在权威模块 |

## 现有目录迁移映射

本轮不执行下列迁移，只在后续整理具体内容时逐步完成。

| 当前目录 | 目标归属 |
| --- | --- |
| `docs/01-common/` | `docs/01-foundation/`；其中 SLA 移至 `02-record-to-report/subledger-accounting/` |
| `docs/02-ap/` | `docs/03-procure-to-pay/payables/` |
| `docs/03-ar/` | `docs/04-credit-to-cash/receivables/` |
| `docs/04-gl/` | `docs/02-record-to-report/general-ledger/` |
| `docs/05-fa/` | `docs/05-assets-projects/fixed-assets/` |
| `docs/06-cost/` | 拆分至 `07-cost-accounting/` 下 Receiving、INV、WIP、CST 等子域 |
| `docs/07-ce-tax/` | 拆分至 `06-cash-tax/` 下 Cash Management、Treasury、EBTax |
| `docs/08-e2e/` | `docs/09-end-to-end/` |
| `docs/09-technical/` | 拆分至 `docs/10-technical/` 与 `docs/11-implementation-operations/` |
| 各模块 `tables.md` | 对应产品 `data-model.md`；`90-reference` 仅保留中央索引 |
| 各模块 `interfaces.md` | 对应产品 `integration.md`；跨模块编排归入 `09-end-to-end/` |

## 命名与维护规范

- 仅一级知识域使用两位编号；`90` 固定为索引，`99` 固定为归档。
- 产品子目录不编号，文件名统一使用小写英文 `kebab-case`。
- 中文名称写在 Markdown 标题和导航中，不把作者、日期、版本和状态放入文件名。
- 每个目录的 `README.md` 是导航 Hub，不承载大段专题正文。
- 不预建空目录和空文档；新增内容时同时更新直接父级 `README.md`。
- 拆分或移动文件时保留迁移记录，并一次性修正根目录、知识图谱和交叉链接。
- SQL 默认使用绑定变量；大表查询必须限定组织、账簿、日期或主键范围。
- AWR、ASH、SQL Monitor、Partitioning 等能力必须注明许可证边界。
- 可选产品、国家本地化和 MOS 专属资料必须注明适用范围和访问权限。

## Oracle R12.2 官方资料基线

- [Oracle E-Business Suite Release 12.2 Documentation Library](https://docs.oracle.com/cd/E26401_01/index.htm)
- [Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
- [Oracle Financials Concepts Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48836/toc.htm)
- [Oracle Financials Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48783/toc.htm)
- [Oracle Projects Documentation](https://docs.oracle.com/cd/E26401_01/nav/projects.htm)
- [Oracle Procurement Documentation](https://docs.oracle.com/cd/E26401_01/nav/procurement.htm)
- [Oracle Supply Chain Management Documentation](https://docs.oracle.com/cd/E26401_01/nav/scm.htm)
- [Oracle E-Business Suite Technology Documentation](https://docs.oracle.com/cd/E26401_01/nav/technology.htm)
- [Oracle E-Business Suite Concepts](https://docs.oracle.com/cd/E26401_01/doc.122/e22949/toc.htm)
- [Oracle E-Business Suite Setup Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22953/toc.htm)
- [Oracle E-Business Suite Security Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22952/toc.htm)
- [Oracle E-Business Suite Maintenance Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22954/toc.htm)
- [Oracle E-Business Suite Integrated SOA Gateway Developer's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e20927/toc.htm)
- [Oracle E-Business Suite Electronic Technical Reference Manual User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/)
