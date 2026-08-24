# 财务产品地图与边界

本页按业务能力说明 Oracle E-Business Suite（EBS）财务及相邻产品的职责。产品是否可用取决于许可证、安装状态、国家本地化和补丁级别；标记为“可选”的能力不得仅凭菜单或表存在就认定可使用。

## 一图理解产品关系

```text
供应商/TCA → PO/Receiving → AP ──→ IBY ──→ Bank
                    │          │        │        │
                    │          └→ SLA ──┼→ GL ←─┤ CE Reconciliation
                    │                   │
客户/TCA → OM/Shipping → AR ───────────┘
                    │
             Inventory/WIP/Cost ─→ SLA/GL
                    │
Projects/CIP ─→ Fixed Assets ─────→ SLA/GL

EBTax 为 PO/AP/AR/OM/Projects/Assets 等交易提供税务确定
SLA 为各子账提供会计规则与分录生成
GL 汇总最终会计、余额与财务报告
```

## 共享基础产品

| 产品/能力 | 英文全称与中文说明 | 主要职责 | 不负责什么 |
| --- | --- | --- | --- |
| FND | Application Object Library，应用对象库 | 用户、职责、菜单、Profile、Lookup、并发程序和公共元数据 | 不定义具体子账交易规则 |
| TCA | Trading Community Architecture，贸易社区架构 | Party、地点、账户、关系和客户/供应商共享身份模型 | 不承担 AP 发票或 AR 收款处理 |
| SLA/XLA | Subledger Accounting，子账会计 | 会计事件、规则、分录、传送 GL 和追溯 | 不替代业务子账余额，也不完成 GL 过账 |
| EBTax/ZX | E-Business Tax，电子商务税 | 交易税适用性、税率、税额、抵扣和税务明细 | 不替代法定申报流程和所有国家本地化 |
| MOAC | Multi-Org Access Control，多组织访问控制 | 基于安全配置文件访问多个经营单位 | 不代替 Ledger/Data Access Set 的总账权限 |

## Record to Report（R2R，记录到报告）

| 产品/能力 | 中文说明 | 核心对象/流程 | 主要上下游 |
| --- | --- | --- | --- |
| GL | General Ledger，总账 | Ledger、Journal、Posting、Balance、Revaluation、Translation、Reporting | 上游为 SLA/Journal Import；下游为报表、合并 |
| SLA | Subledger Accounting，子账会计 | Event、Accounting Rule、Journal Entry、GL Transfer | 上游为各子账；下游为 GL |
| FAH（可选） | Financials Accounting Hub，财务会计中心 | 为外部系统或扩展来源建立会计事件与会计规则 | 外部子账、SLA、GL |
| AGIS（可选） | Advanced Global Intercompany System，高级全球公司间系统 | 公司间交易、审批、AR/AP/GL 衔接和对账 | 法人、AP、AR、GL |
| Budgetary Control | 预算控制 | Funds Check、Reservation、Encumbrance、Budget Balance | Purchasing、AP、Projects、GL；具体模式需按产品确认 |
| Consolidation | 合并 | 多账簿余额映射、传输、调整和抵销 | Ledger、汇率、报表 |

边界提示：Ledger Set（账簿集）是集合访问/处理能力，不等于合并；Secondary Ledger（辅助账簿）是另一会计表示，不等于 Reporting Currency（报告币种）。

入口：[Record to Report](../02-record-to-report/README.md)｜[既有 GL 详文](../04-gl/README.md)

## Procure to Pay（P2P，采购到付款）

| 产品/能力 | 中文说明 | 核心对象/流程 | 边界 |
| --- | --- | --- | --- |
| Supplier/TCA | 供应商与参与方主数据 | Supplier、Site、Contact、税务和银行关系 | 主数据身份由 TCA 支撑，采购/AP 使用业务站点 |
| Purchasing（PO） | 采购 | Requisition、RFQ、Quotation、Purchase Order、Agreement | 创建采购承诺，不形成 AP 负债 |
| iProcurement（可选） | 互联网采购 | 员工自助请购、目录、购物车和审批 | 前端采购体验，后续仍由 PO/Receiving/AP 处理 |
| Receiving（RCV） | 接收 | Receipt、Inspection、Delivery、Return、Correction、Accrual | 接收与入库不同；应计方式受目的地和设置影响 |
| AP | Payables，应付账款 | Invoice、Matching、Validation、Hold、Approval、Liability、Payment Schedule | 确认负债和付款资格，不负责最终银行文件编排 |
| IBY | Oracle Payments，Oracle 支付 | Payment Process Request、Instruction、Format、Transmission、ACK | 编排支付，不替代 AP 负债和 CE 银行对账 |
| iExpenses（可选） | Internet Expenses，互联网费用报销 | Expense Report、Policy、Card、Audit、AP Import | 报销前端，付款通常进入 AP/IBY |
| iSupplier（可选） | 供应商门户 | 供应商协同、订单/发运/发票可见性 | 权限和数据范围需独立治理 |

入口：[Procure to Pay](../03-procure-to-pay/README.md)｜[既有 AP 详文](../02-ap/README.md)

## Order/Credit to Cash（O2C/C2C，订单/信用到收款）

| 产品/能力 | 中文说明 | 核心对象/流程 | 边界 |
| --- | --- | --- | --- |
| TCA Customer | 客户主数据 | Party、Customer Account、Site Use、Contact、Profile | Party 与 Account 是不同层级 |
| Order Management（OM） | 订单管理 | Sales Order、Price、Credit Check、Fulfillment | 订单履行不等于 AR 已开票 |
| Shipping Execution（WSH） | 发运执行 | Pick、Ship Confirm、Delivery | 发运状态与 AutoInvoice 状态需分别追踪 |
| AR | Receivables，应收账款 | Transaction、AutoInvoice、Receipt、Application、Adjustment、Revenue | 管理应收和收款应用，不替代银行对账 |
| Credit Management（可选） | 信用管理 | Credit Review、Case Folder、Credit Limit | 与 OM 信用检查、客户 Profile 配合 |
| Advanced Collections（可选） | 高级催收 | Delinquency、Strategy、Promise to Pay、Dunning | 与 AR 余额、账龄和争议衔接 |
| iReceivables（可选） | 互联网应收 | 客户自助查看账单、付款和争议 | 需评估外部访问和支付安全 |
| Loans（可选） | 贷款 | Origination、Servicing、Billing、Interest | 独立产品，不应与普通 AR 交易混为一谈 |

入口：[Credit to Cash](../04-credit-to-cash/README.md)｜[既有 AR 详文](../03-ar/README.md)

## Acquire to Retire 与 Projects（资产取得到退出、项目）

| 产品/能力 | 中文说明 | 核心对象/流程 | 边界 |
| --- | --- | --- | --- |
| FA | Fixed Assets，固定资产 | Asset、Book、Category、Depreciation、Transfer、Retirement | 管理资产价值与折旧，不承担采购审批 |
| Mass Additions | 成批增加 | 从 AP/Projects/外部来源准备并过账资产 | 接口成功不等于资产已资本化 |
| Projects Foundation | 项目基础 | Project、Task、Organization、Role、Classification | 为 Costing/Billing 等项目产品提供基础 |
| Project Costing | 项目成本 | Expenditure、Cost Distribution、Burden、Commitment | 成本分配与 GL/FA 接口是后续步骤 |
| Project Billing | 项目开票 | Agreement、Funding、Revenue、Invoice、AR Interface | 项目收入/发票进入 AR 后仍需 AR 处理 |
| Project to Asset | 项目转资产 | CIP、Project Asset、Capitalization、FA Interface | 需对账项目资本成本、FA 成本和 GL |
| Grants（可选） | 赠款会计 | Award、Funding、Compliance | 依赖项目基础并受资助方规则影响 |
| iAssets（可选） | 互联网资产 | 员工资产查询/转移请求 | 不替代 FA 核心资产账簿 |
| Property Manager（可选） | 物业管理 | Property、Space、Lease、Payment/Receipt | 与 AP/AR/FA/Projects 的接口需明确 |

入口：[资产与项目](../05-assets-projects/README.md)｜[既有 FA 详文](../05-fa/README.md)

## Cash、Treasury 与 Tax（现金、资金与税务）

| 产品/能力 | 中文说明 | 核心对象/流程 | 边界 |
| --- | --- | --- | --- |
| CE | Cash Management，现金管理 | Bank Account、Statement、Reconciliation、Cash Position、Forecast | 对账和现金可视性，不负责 AP 付款选择 |
| IBY | Oracle Payments，Oracle 支付 | 付款/收款执行框架、格式、传输和回执 | 与 AP/AR 业务义务及 CE 对账分层 |
| Treasury（可选） | 资金管理 | Deal、Counterparty、Limit、Settlement、Exposure、Revaluation | 需要单独确认产品许可和会计边界 |
| EBTax | 电子商务税 | Tax Regime、Tax、Rate、Rule、Recovery、Tax Line | 税务确定与法定申报不能简单等同 |
| Tax Reporting Ledger | 税务报告台账 | 提取、分组、法定报告数据准备 | 国家本地化和申报工具因地区而异 |

入口：[现金与税务](../06-cash-tax/README.md)｜[既有 CE/Tax 详文](../07-ce-tax/README.md)

## Supply Chain Financials and Costing（供应链财务与成本）

| 产品/能力 | 中文说明 | 核心对象/流程 | 边界 |
| --- | --- | --- | --- |
| Inventory（INV） | 库存 | Item、On-hand、Material Transaction、Lot/Serial、Valuation | 库存数量状态与成本处理状态需分别检查 |
| Cost Management（CST） | 成本管理 | Cost Type、Element、Rollup、Cost Update、Distribution | 计算/分配成本，不替代业务交易处理 |
| WIP | Work in Process，在制品 | Job、Material、Resource、Completion、Close、Variance | 关闭工单前需完成交易与成本处理 |
| OPM Costing（可选） | 流程制造成本 | Process Organization、Recipe/Batch 相关成本 | 数据模型和成本方法与离散制造不同 |
| LCM（可选） | Landed Cost Management，到岸成本管理 | Charge、Allocation、Actual Landed Cost | 需与采购、接收、库存和 AP 差异对账 |
| COGS Recognition | 销售成本确认 | Revenue/COGS Matching | 与 OM/Shipping/AR 收入事件协同 |

入口：[供应链财务与成本](../07-cost-accounting/README.md)｜[既有 Cost 详文](../06-cost/README.md)

## Reporting and Governance（报表与治理）

| 工具/能力 | 中文说明 | 适用范围 | 注意事项 |
| --- | --- | --- | --- |
| FSG | Financial Statement Generator，财务报表生成器 | 基于 GL 余额的财务报表 | 行集、列集、内容集和显示集需版本管理 |
| BI Publisher / XML Publisher | BI 发布工具/XML 发布工具 | 模板化报表、eText、分发 | 数据定义、模板、字体、OPP 和安全共同影响结果 |
| RXi | Report eXchange，报表交换 | 可配置财务报表输出 | 与普通 BI Publisher 模板治理不同 |
| Web ADI | Web Applications Desktop Integrator，Web 应用桌面集成器 | Excel 录入/上传和部分配置迁移 | 模板、Integrator、权限和版本需控制 |
| Smart View | 智能视图 | Office 中的查询和财务分析 | 可用性取决于部署组件与配置 |
| ECC（可选） | Enterprise Command Center，企业指挥中心 | 运营看板和数据发现 | 数据加载、职责安全和产品版本需确认 |

入口：[报表与治理](../08-reporting-governance/README.md)

## 技术产品边界

| 能力 | 负责 | 不应承担 |
| --- | --- | --- |
| Open Interface | 标准批量导入与错误处理入口 | 绕过标准导入程序直接写业务表 |
| Public API | 受支持的程序化业务操作 | 根据网络样例猜 API 签名和 Commit 行为 |
| Integration Repository / ISG | 服务发现、部署和调用治理 | 把所有内部 Package 都视为公开服务 |
| Concurrent Processing | 后台调度、批处理、日志和输出 | 单凭请求 Completed/Normal 判断业务成功 |
| Workflow / AME | 流程活动、通知和规则审批 | 代替业务对象本身的状态和会计校验 |
| ADOP / EBR | R12.2 在线补丁和版本化发布 | 允许覆盖标准文件或跳过发布审批 |

## 项目范围确认清单

每个产品进入方案前确认：

1. 产品是否已安装、共享还是完全安装，是否有使用许可证。
2. 目标 EBS、AD/TXK、数据库和中间件补丁级别。
3. 适用法人、账簿、OU、库存组织、国家/地区和币种。
4. 业务所有者、主数据所有者、会计所有者、接口所有者和关账责任人。
5. 标准功能、CEMLI、本地化和外部系统之间的边界。
6. 交易量、历史数据、月结窗口、批处理容量和保留要求。
7. 安全、职责分离、审计、隐私、银行/税务合规要求。
8. 配置、接口、会计、报表、迁移和运维的验收证据。

## 官方依据

- [Oracle E-Business Suite R12.2 Documentation Library](https://docs.oracle.com/cd/E26401_01/index.htm)
- [Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
- [Oracle Projects Documentation](https://docs.oracle.com/cd/E26401_01/nav/projects.htm)
- [Oracle Procurement Documentation](https://docs.oracle.com/cd/E26401_01/nav/procurement.htm)
- [Oracle Supply Chain Management Documentation](https://docs.oracle.com/cd/E26401_01/nav/scm.htm)
