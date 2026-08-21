# oracle-ebs
knowledge about  oracle ebs
# Oracle EBS 财务知识库

本仓库用于沉淀 Oracle E-Business Suite（EBS）R12 财务与相关供应链模块的业务流程、配置说明、常用 SQL、接口及运维排错经验。当前已包含一个[轻量 PL/SQL Web Service 网关](#通用-web-service轻量-plsql-网关)示例；以下目录可作为后续文档的统一入口。

## 知识目录

### 01. 财务公共基础（Financials Common）

- [企业结构与多组织（Ledger / Legal Entity / Operating Unit）](docs/01-common/organization.md)
- [会计科目与弹性域（Chart of Accounts / Key Flexfield）](docs/01-common/coa.md)
- [日历、币种、汇率与期间控制](docs/01-common/calendar-currency-period.md)
- [职责、数据访问权限与安全配置](docs/01-common/security.md)
- [子分类账会计（SLA）规则、事件与过账](docs/01-common/sla.md)
- [附件、说明性弹性域与文档序列](docs/01-common/attachments-dff.md)

### 02. 应付账款 AP（Payables）

- [业务流程：供应商、采购订单、收货、发票、付款](docs/02-ap/process.md)
- [供应商与供应商地点](docs/02-ap/suppliers.md)
- [发票录入、验证、暂估与税务](docs/02-ap/invoices.md)
- [付款、付款批次、银行账户与核销](docs/02-ap/payments.md)
- [预付款、员工报销、借项通知单与保留款](docs/02-ap/special-transactions.md)
- [AP 会计、过账、月结与常用报表](docs/02-ap/accounting-close-reports.md)
- [Open Interface、导入发票与常用排错 SQL](docs/02-ap/interfaces-troubleshooting.md)

### 03. 应收账款 AR（Receivables）

- [业务流程：客户、订单、开票、收款、核销](docs/03-ar/process.md)
- [客户、客户地点、付款条件与信用管理](docs/03-ar/customers-credit.md)
- [交易类型、发票、贷项通知单与调整](docs/03-ar/transactions.md)
- [收款、锁箱、自动核销与退款](docs/03-ar/receipts.md)
- [催收、账龄、坏账准备与核销](docs/03-ar/collections-aging.md)
- [AR 会计、过账、月结与报表](docs/03-ar/accounting-close-reports.md)
- [AutoInvoice、收款接口与常用排错 SQL](docs/03-ar/interfaces-troubleshooting.md)

### 04. 总账 GL（General Ledger）

- [账簿、会计科目、日记账与过账流程](docs/04-gl/process.md)
- [日记账来源、类别、审批与自动过账](docs/04-gl/journals.md)
- [预算、预算控制与资金可用性](docs/04-gl/budgetary-control.md)
- [合并、重估、折算与重复日记账](docs/04-gl/consolidation-revaluation.md)
- [期间打开/关闭、月结、年结与报表](docs/04-gl/close-reports.md)
- [FSG、Smart View、Web ADI 与日记账导入](docs/04-gl/reporting-interfaces.md)

### 05. 固定资产 FA（Assets）

- [资产全生命周期：资本化、折旧、处置](docs/05-fa/process.md)
- [资产账簿、资产类别、位置与关键配置](docs/05-fa/setup.md)
- [资产增加、调整、转移、重分类与盘点](docs/05-fa/asset-transactions.md)
- [折旧、税务折旧、资产处置与会计](docs/05-fa/depreciation-accounting.md)
- [FA 月结、报表、接口与排错](docs/05-fa/close-reports-interfaces.md)

### 06. 成本管理 Cost Management（CST）

- [成本组织、成本类型、成本要素与成本组](docs/06-cost/setup.md)
- [标准成本、平均成本与周期成本](docs/06-cost/costing-methods.md)
- [物料成本、资源成本、间接费用与成本更新](docs/06-cost/cost-elements.md)
- [接收、库存、WIP 与销售成本核算](docs/06-cost/accounting-flow.md)
- [成本分配、差异分析、结转、月结与报表](docs/06-cost/period-close-reports.md)
- [成本接口、Transaction Processor 与排错](docs/06-cost/interfaces-troubleshooting.md)

### 07. 现金管理与税务（CE / EBTax）

- [银行、银行账户、对账单与自动核对](docs/07-ce-tax/cash-management.md)
- [付款/收款清算、现金预测与银行接口](docs/07-ce-tax/cash-forecast-interfaces.md)
- [税种、税率、税务规则与税务报表](docs/07-ce-tax/ebtax.md)

### 08. 关联模块与端到端流程

- [采购到付款（PO → Receiving → AP → GL）](docs/08-e2e/procure-to-pay.md)
- [订单到收款（OM → Shipping → AR → GL）](docs/08-e2e/order-to-cash.md)
- [库存、WIP、成本与总账衔接](docs/08-e2e/inventory-wip-cost-gl.md)
- [项目、费用与资产资本化](docs/08-e2e/projects-assets.md)

### 09. 技术、接口与运维

- [EBS 数据模型与常用表](docs/09-technical/data-model.md)
- [Concurrent Program、请求集与日志排错](docs/09-technical/concurrent-programs.md)
- [PL/SQL、Forms、Personalization 与 OAF](docs/09-technical/customization.md)
- [开放接口、API、报表与数据迁移](docs/09-technical/integration.md)
- [性能调优、权限审计与生产运维](docs/09-technical/operations.md)

## 建议的文档约定

- 每篇文档优先说明适用版本、业务前提、菜单路径、关键配置、操作步骤、会计影响、常用表与排错方式。
- 涉及 SQL 时标注环境与只读/写入风险；生产环境执行更新脚本前须走变更与备份流程。
- 目录中的链接为后续文档预留路径；新增资料时请按模块归档，并同步更新本页。