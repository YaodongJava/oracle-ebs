# 现金、资金与税务（Cash, Treasury and Tax）

> 本模块覆盖银行账户、银行流水、现金头寸、对账、资金交易以及 E-Business Tax（电子商务税务，EBTax/ZX）。现金事实以银行回执和对账为准，税务事实以税制、税状态、税率、辖区和产品税属性为准。

## 阅读导航

- [范围](#1-学习目标与边界) · [现金主链](#2-银行与现金主链) · [银行对账](#3-银行流水接口与对账) · [资金管理](#4-现金头寸与资金管理) · [税务确定](#5-e-business-tax-税务确定) · [会计技术](#6-会计和对账) · [专题详解](#9-专题详解)

## 1. 学习目标与边界

应能设计集中式银行账户、AP/AR 与 CE 的清算链、银行流水接口和自动对账；理解现金预测、内部转账和资金产品边界；能够解释 ZX 税务确定流程、恢复税和税务报告。

## 2. 银行与现金主链

```text
AP 付款 / AR 收款 / 资金交易
→ Cash Clearing（现金清算）
→ 银行流水导入
→ 自动/人工匹配
→ Reconciliation（银行对账）
→ 清算会计、现金头寸和差异处理
```

银行账户应由 Legal Entity 或组织按授权使用。账户、分行、签字权限、用途、币种和支付/收款方法变更都属于高风险主数据变更。

## 3. 银行流水接口与对账

接口契约至少包含账户标识、流水号、价值日、交易日、币种、金额、借贷标识、银行交易码、客户参考和文件控制总额。重复文件识别应组合文件 ID、账户、日期和哈希/控制总额。

自动对账规则要定义匹配字段、日期窗口、金额容差、优先级和一对多/多对一场景。无法可靠匹配的流水进入异常队列；人工匹配需要原因和审批证据。

## 4. 现金头寸与资金管理

Cash Position（现金头寸）强调已发生或高确定性现金；Cash Forecast（现金预测）结合 AP、AR、采购、订单、工资和外部预测。必须标记来源、日期口径、置信度和重复消除规则。

Treasury（资金管理）可涉及借款、投资、外汇、利率和套期。实施前确认产品安装与许可证，并明确前台交易、风险控制、后台结算和会计职责分离。

## 5. E-Business Tax 税务确定

典型决定链：Tax Regime（税制）→ Tax（税种）→ Tax Status（税状态）→ Tax Rate（税率）→ Tax Jurisdiction（税务辖区）→ Tax Rule（税规则）。交易方税档案、产品税分类、地点、交易类型和日期共同影响结果。

Recoverable Tax（可抵扣税）与 Nonrecoverable Tax（不可抵扣税）会进入不同账户或成本。税额人工覆盖必须受权限、原因和审计控制，不能用来绕过错误主数据或规则。

## 6. 会计和对账

- 付款/收款：先区分已生成、已确认、已汇款、已清算和已对账。
- 银行：账面余额、银行余额、在途项目和调节项必须可解释。
- 税务：交易税行、SLA 税务会计、税务登记簿和 GL 税账户按期间与法人对账。
- 汇率：交易换算、银行实际结算和期末重估差异应分别记录。

## 7. 技术视角

CE、IBY、AP、AR 与 ZX 跨模块关联时，以银行账户、支付/收款标识、流水编号和会计事件逐层追踪。银行文件和税务接口属于敏感数据；日志不得输出完整账号、税号、凭据或支付报文。

接口需要幂等、文件级与行级状态、控制总额、错误重放和回执相关号。目标实例可能使用不同银行格式或外部税引擎，必须用真实契约验证。

## 8. 高频问题与练习

- 银行流水未匹配：检查账户、交易码、日期窗口、金额、参考号和规则优先级。
- 税未计算/税率错误：检查配置有效期、税注册、地点、产品税分类、规则和人工覆盖。
- CE 与 GL 差异：检查未清算项目、会计日期、未会计交易、手工日记账和期间。

建议完成一笔 AP 付款至银行清算、一笔 AR Lockbox 至对账，以及一笔含可抵扣/不可抵扣税的采购交易。

## 9. 专题详解


<!-- source: docs/07-ce-tax/README.md -->
<a id="src-docs-07-ce-tax-readme"></a>
### 现金管理、付款与税务（CE / IBY / EBTax）


本目录覆盖银行账户与用途、银行对账单、自动核对、现金预测、Oracle Payments 支付链和 E-Business Tax 税务确定。银行账户是跨 AP、AR、CE、IBY、Treasury 和 GL 的公共主数据，权限、加密、审批、回执与对账不可分割。

<a id="src-docs-07-ce-tax-readme--专题导航"></a>
#### 专题导航

- [银行、账户、对账单与自动核对](#src-docs-07-ce-tax-cash-management)
- [现金预测、清算与银行接口](#src-docs-07-ce-tax-cash-forecast-interfaces)
- [Treasury、现金头寸与银行主数据治理](#src-docs-07-ce-tax-treasury-bank-governance)
- [税种、税率、规则与排错](#src-docs-07-ce-tax-ebtax)
- [税务报告、本地化与合规控制](#src-docs-07-ce-tax-tax-reporting-localization)
- [表结构](#src-docs-07-ce-tax-tables)
- [银行对账单、支付、税务接口实现](#src-docs-07-ce-tax-interfaces)

<a id="src-docs-07-ce-tax-readme--运行控制"></a>
#### 运行控制

| 领域 | 必须对账的对象 | 关键例外 |
| --- | --- | --- |
| CE | 银行对账单余额、已核对/未核对 AP/AR/Treasury/GL 交易 | 重复导入、日期错位、银行交易代码映射错误 |
| IBY | PPR、付款指令、支付文件、传输、ACK、作废/重发 | 文件已发出但状态未回写、回执与付款状态不一致 |
| EBTax | 交易税行、税务登记、税率、Recoverability、税务报告 | 税务决定因素缺失、Tax Regime/Status/Rate 不适用、反向计税错误 |

<a id="src-docs-07-ce-tax-readme--安全边界"></a>
#### 安全边界

银行账号、证书、密钥、支付文件和税务身份信息不得写入示例或日志。接口只保存必要的掩码、哈希、外部业务键和审计相关号；敏感明细的查询权限应受职责和数据库最小授权约束。

<a id="src-docs-07-ce-tax-readme--官方依据"></a>
#### 官方依据

- [Oracle Cash Management Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
- [Oracle E-Business Tax Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/07-ce-tax/cash-forecast-interfaces.md -->
<a id="src-docs-07-ce-tax-cash-forecast-interfaces"></a>
### 付款/收款清算、现金预测与银行接口


> 银行对账单两张接口表、IBY 付款/回执和 EBTax 传参代码见 [CE/IBY/EBTax 接口实现案例](#src-docs-07-ce-tax-interfaces)。

<a id="src-docs-07-ce-tax-cash-forecast-interfaces--现金预测"></a>
#### 现金预测

Cash Forecast Template 定义来源、时间桶、币种和包含条件。来源可包括 AP Invoices/Payments、AR Transactions/Receipts、PO/Requisitions、Payroll、Cashflows 和手工预测。Forecast 是预期流动性，不是 GL 实际现金余额。

<a id="src-docs-07-ce-tax-cash-forecast-interfaces--银行接口设计"></a>
#### 银行接口设计

- 入站：Bank Statement/Lockbox/Acknowledgement，保存原始文件、Checksum、Bank/Account、Sequence、批次和重复键。
- 出站：Payment File/Positive Pay/Remittance，使用 IBY Format/PPP/Transmission，实施加密、签名、安全传输和回执。
- 不将银行账号、个人信息或私钥写入并发日志；设置文件保留/脱敏/归档策略。

<a id="src-docs-07-ce-tax-cash-forecast-interfaces--sql"></a>
#### SQL

```sql
-- 未对账银行流水汇总
SELECT csh.bank_account_id, csl.trx_type, csl.status,
       COUNT(*) line_count, SUM(csl.amount) amount
  FROM ce_statement_headers csh
  JOIN ce_statement_lines csl
    ON csl.statement_header_id = csh.statement_header_id
 WHERE csh.statement_date BETWEEN :p_start_date AND :p_end_date
 GROUP BY csh.bank_account_id, csl.trx_type, csl.status;

-- 请求参数和日志线索
SELECT request_id, phase_code, status_code,
       actual_start_date, actual_completion_date,
       argument_text, logfile_name, outfile_name
  FROM fnd_concurrent_requests
 WHERE request_id = :p_request_id;
```

<a id="src-docs-07-ce-tax-cash-forecast-interfaces--排查"></a>
#### 排查

- Forecast 为空：检查 Template Source、Cutoff Date/Bucket、OU/Ledger/Bank Account、Currency 和源单据状态。
- Forecast 重复：检查多来源是否同时包含 Invoice+Payment/Order+Invoice，理解预测选择规则。
- Bank File 重复：使用 Bank Account + Statement/File ID + Sequence + Checksum 幂等，不仅依赖文件名。
- 传输失败：查证书/密钥有效期、SFTP/HTTPS 连通、目录权限、文件编码、银行回执和 IBY/OPP 日志。

<a id="src-docs-07-ce-tax-cash-forecast-interfaces--关联"></a>
#### 关联

- [Cash Management](#src-docs-07-ce-tax-cash-management)
- [Integration](10-technical.md#src-docs-09-technical-integration)


<!-- source: docs/07-ce-tax/cash-management.md -->
<a id="src-docs-07-ce-tax-cash-management"></a>
### 现金管理：银行、银行账户、对账单与自动核对


<a id="src-docs-07-ce-tax-cash-management--模型与流程"></a>
#### 模型与流程

R12 银行/分行基于 TCA，内部银行账户由 CE 管理，账户所有者/用途决定 AP/AR/Payroll/Treasury 在哪个 Legal Entity/OU 下可用。

```text
Bank Statement Import/Manual Entry
→ Header/Lines → AutoReconciliation Matching
→ Reconciled/Unreconciled/Errors
→ Cash Position + Accounting/Close
```

AutoReconciliation 根据 Transaction Code Mapping、Reference、Amount/Date Tolerance、Receipt/Payment Number 等规则匹配 AP Payment、AR Receipt、Cashflow、Bank Transfer 和手工现金交易。

<a id="src-docs-07-ce-tax-cash-management--sql"></a>
#### SQL

```sql
SELECT cba.bank_account_id, cba.bank_account_name,
       cba.bank_account_num, cba.currency_code,
       cba.start_date, cba.end_date,
       cba.account_owner_org_id
  FROM ce_bank_accounts cba
 WHERE cba.bank_account_id = :p_bank_account_id;

SELECT csh.statement_header_id, csh.bank_account_id,
       csh.statement_number, csh.statement_date,
       csh.currency_code, csh.control_begin_balance,
       csh.control_end_balance
  FROM ce_statement_headers csh
 WHERE csh.bank_account_id = :p_bank_account_id
 ORDER BY csh.statement_date DESC;

SELECT csl.statement_line_id, csl.line_number,
       csl.trx_date, csl.trx_type, csl.trx_code,
       csl.amount, csl.status, csl.bank_trx_number,
       csl.invoice_text
  FROM ce_statement_lines csl
 WHERE csl.statement_header_id = :p_statement_header_id
 ORDER BY csl.line_number;
```

<a id="src-docs-07-ce-tax-cash-management--排查"></a>
#### 排查

- 银行账户不可选：查 Owner Legal Entity、OU Use、Application Use、Currency、有效期和用户权限。
- Statement Import 错：检查 Bank Account/Number、Currency、Statement Number 唯一性、Control Balance、Transaction Code 和文件格式。
- AutoReconciliation 匹配不到：比较 Transaction Type/Code、Reference、Amount/Date/Currency、原交易状态和容差。
- 对账后 GL 不对：跟踪 AP/AR/CE 原交易、Clearing Event、SLA 和 GL Post，区分对账状态与会计状态。

<a id="src-docs-07-ce-tax-cash-management--关联"></a>
#### 关联

- [CE/IBY/EBTax 常用表结构与字段含义](#src-docs-07-ce-tax-tables)
- [AP Payments](03-procure-to-pay.md#src-docs-02-ap-payments)
- [AR Receipts](04-credit-to-cash.md#src-docs-03-ar-receipts)


<!-- source: docs/07-ce-tax/ebtax.md -->
<a id="src-docs-07-ce-tax-ebtax"></a>
### Oracle E-Business Tax（EBTax）税种、税率、规则与排错


<a id="src-docs-07-ce-tax-ebtax--税务确定模型"></a>
#### 税务确定模型

```text
Configuration Owner / Legal Entity / OU
→ Tax Regime → Tax → Tax Status → Tax Rate / Jurisdiction
→ Party Tax Profile / Registration / Exemption
→ Determining Factors + Rules
→ Tax Lines / Recoverability / Accounting
```

EBTax 是一套中央税引擎。应用产品（AP/AR/PO/OM）传入交易日期、法人/OU、交易业务类别、产品财政分类、交易方税务分类、Ship From/To/Bill From/To 等确定因素；规则按优先级确定 Applicable Tax、Place of Supply、Status、Rate、Taxable Basis、Recovery 等。

<a id="src-docs-07-ce-tax-ebtax--配置"></a>
#### 配置

1. 确定 Configuration Owner Tax Options 和 Party Tax Profile/Registrations。
2. 定义 Regime/Tax/Status/Rate/Jurisdiction，检查有效期与地理范围。
3. 定义 Fiscal Classifications、Tax Zones、Determining Factor Sets/Conditions/Rules。
4. 定义 Tax Accounts、Recovery Rates、Exemptions、Thresholds、Inclusive/Compound Tax。
5. 按 AP/AR/PO/OM、手工/接口、Credit/Return、预付、外币、不同地址组合建立测试矩阵。

<a id="src-docs-07-ce-tax-ebtax--sql"></a>
#### SQL

```sql
-- 税行与源单据；ENTITY/EVENT_CLASS 按产品解读
SELECT zxl.tax_line_id, zxl.application_id,
       zxl.entity_code, zxl.event_class_code,
       zxl.trx_id, zxl.trx_line_id, zxl.trx_level_type,
       zxl.tax_regime_code, zxl.tax, zxl.tax_status_code,
       zxl.tax_rate_code, zxl.tax_rate, zxl.taxable_amt,
       zxl.tax_amt, zxl.tax_amt_funcl_curr,
       zxl.cancel_flag, zxl.delete_flag
  FROM zx_lines zxl
 WHERE zxl.application_id = :p_application_id
   AND zxl.trx_id = :p_trx_id
 ORDER BY zxl.tax_line_id;

SELECT tax_regime_code, tax, tax_status_code,
       tax_rate_code, percentage_rate,
       effective_from, effective_to, active_flag
  FROM zx_rates_b
 WHERE tax_regime_code = :p_tax_regime_code
   AND tax = :p_tax
 ORDER BY effective_from;
```

<a id="src-docs-07-ce-tax-ebtax--排查方法"></a>
#### 排查方法

1. 锁定 Application/Entity/Event Class/Trx ID/Line ID，不只用发票号查 ZX。
2. 重建税确定因素快照：Owner/LE/OU、Date、Party Registrations、Locations、Product/Party Fiscal Classification。
3. 按 Applicable Tax → Place of Supply → Status → Rate → Basis → Recovery 的顺序查规则、优先级和默认。
4. 比较正常/异常单据的所有确定因素，而不是直接比较最终 Tax Rate。

常见问题：税不计算通常是 Owner/Regime Applicability/Date/Place of Supply 不匹配；税率错通常是 Status/Rate Rule 优先级或 Registration/Class 不同；进项税不可抵扣要检查 Recovery Rule/Rate/Account；会计错要区分 ZX Tax Line 正确但 SLA 账户派生错的情况。

<a id="src-docs-07-ce-tax-ebtax--关联"></a>
#### 关联

- [AP Invoices](03-procure-to-pay.md#src-docs-02-ap-invoices)
- [AR Transactions](04-credit-to-cash.md#src-docs-03-ar-transactions)
- [SLA](01-foundation.md#src-docs-01-common-sla)

<a id="src-docs-07-ce-tax-ebtax--官方参考"></a>
#### 官方参考

- [Oracle Financials Implementation Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48783/T348488T348491.htm)


<!-- source: docs/07-ce-tax/interfaces.md -->
<a id="src-docs-07-ce-tax-interfaces"></a>
### Oracle Cash Management、Payments 与 E-Business Tax 接口实现案例


<a id="src-docs-07-ce-tax-interfaces--1-业界常用场景"></a>
#### 1. 业界常用场景

| 场景 | 推荐接口 | 实施方法 |
| --- | --- | --- |
| 银行对账单 MT940/BAI2/CAMT.053 | Bank Statement Loader/Open Interface | 原始文件归档 → 映射 → Header/Line Interface → Import/Reconcile |
| 银行日内流水 CAMT.052 | Intra-Day Bank Statement Loader | 使用 Statement Timestamp/Sequence 区分同日多批 |
| 银行代收/虚拟账号 | AR AutoLockbox | 银行文件进入 `AR_PAYMENTS_INTERFACE_ALL`，再 Validate/Post |
| 付款文件与银行回执 | Oracle Payments（IBY）Format/Transmission | 使用 Payment Process Profile、BI Publisher 格式、SFTP/HTTPS 和 ACK |
| 外部资金系统交易参与对账 | Reconciliation Open Interface | 实现 `CE_999_INTERFACE_V`，供 CE 自动/手工匹配 |
| AP/AR 交易计税 | EBTax Tax Determining Factors | 从 AP/AR 标准接口传税务分类和地点，不直接 DML `ZX_*` |

<a id="src-docs-07-ce-tax-interfaces--2-银行文件落地与幂等"></a>
#### 2. 银行文件落地与幂等

银行原始文件应保存为不可变归档，并记录哈希：

```sql
CREATE TABLE xxce_bank_files (
  bank_file_id       NUMBER        NOT NULL,
  bank_account_key   VARCHAR2(100) NOT NULL,
  statement_number   VARCHAR2(100),
  file_name          VARCHAR2(255) NOT NULL,
  file_sha256        VARCHAR2(64)  NOT NULL,
  file_date          DATE,
  status             VARCHAR2(30)  NOT NULL,
  request_id         NUMBER,
  error_message      VARCHAR2(2000),
  creation_date      DATE          DEFAULT SYSDATE NOT NULL,
  last_update_date   DATE          DEFAULT SYSDATE NOT NULL,
  CONSTRAINT xxce_bank_files_pk PRIMARY KEY (bank_file_id),
  CONSTRAINT xxce_bank_files_u1 UNIQUE (bank_account_key, file_sha256)
);
```

文件名不是可靠重复键。常用重复判断是 Bank Account + Statement Number/Sequence + File Hash；同一银行重新发送修正版时应保留版本关系，不覆盖原文件。

<a id="src-docs-07-ce-tax-interfaces--3-bank-statement-open-interface"></a>
#### 3. Bank Statement Open Interface

Oracle 官方接口由 `CE_STATEMENT_HEADERS_INT` 和 `CE_STATEMENT_LINES_INTERFACE` 组成。官方说明 Header 必填项包括 Statement Number、Bank Account Number、Statement Date 和 Org ID；使用 Loader 时 `ORG_ID` 可由程序填充。

<a id="src-docs-07-ce-tax-interfaces--31-写入对账单头"></a>
##### 3.1 写入对账单头

```sql
INSERT INTO ce_statement_headers_int (
  statement_number,
  bank_account_num,
  statement_date,
  currency_code,
  control_begin_balance,
  control_end_balance,
  control_total_dr,
  control_total_cr,
  record_status_flag,
  org_id
) VALUES (
  :p_statement_number,
  :p_bank_account_number,
  :p_statement_date,
  :p_currency_code,
  :p_opening_balance,
  :p_closing_balance,
  :p_total_debit,
  :p_total_credit,
  'N',
  :p_org_id
);
```

<a id="src-docs-07-ce-tax-interfaces--32-写入对账单行"></a>
##### 3.2 写入对账单行

```sql
INSERT INTO ce_statement_lines_interface (
  statement_number,
  bank_account_num,
  line_number,
  trx_date,
  effective_date,
  trx_code,
  bank_trx_number,
  amount,
  currency_code,
  trx_text
) VALUES (
  :p_statement_number,
  :p_bank_account_number,
  :p_line_number,
  :p_transaction_date,
  :p_value_date,
  :p_bank_transaction_code,
  :p_bank_reference,
  :p_signed_amount,
  :p_currency_code,
  :p_remittance_information
);
```

`CE_STATEMENT_LINES_INTERFACE` 不包含 Header 的 `ORG_ID` 和 `RECORD_STATUS_FLAG`；组织由 Header/Bank Account 确定。不同文件格式对金额正负、Debit/Credit 和交易码的定义不同。必须通过 Bank Transaction Codes 映射验证，不能直接用银行原始符号猜测 EBS 金额方向。

<a id="src-docs-07-ce-tax-interfaces--33-上线前校准字段"></a>
##### 3.3 上线前校准字段

```sql
SELECT table_name,
       column_id,
       column_name,
       data_type,
       nullable
  FROM all_tab_columns
 WHERE owner = 'CE'
   AND table_name IN ('CE_STATEMENT_HEADERS_INT',
                      'CE_STATEMENT_LINES_INTERFACE')
 ORDER BY table_name, column_id;
```

若使用 Oracle 提供的 BAI2、SWIFT940、CAMT Loader，应优先复用标准 Mapping/Loader。只有银行专有格式才开发 Custom Loader；直接插表代码也应封装在 Loader 内，并接受目标实例列校验。

<a id="src-docs-07-ce-tax-interfaces--4-importreconcile-与错误对账"></a>
#### 4. Import、Reconcile 与错误对账

```text
Interface N
→ Bank Statement Import validation
→ CE_STATEMENT_HEADERS / CE_STATEMENT_LINES
→ AutoReconciliation
→ Matched/Cleared/Reconciled
```

```sql
-- 接口状态
SELECT h.statement_number,
       h.bank_account_num,
       h.statement_date,
       h.record_status_flag,
       COUNT(l.line_number) line_count,
       SUM(l.amount) line_amount
  FROM ce_statement_headers_int h
  LEFT JOIN ce_statement_lines_interface l
    ON l.statement_number = h.statement_number
   AND l.bank_account_num = h.bank_account_num
 WHERE h.statement_number = :p_statement_number
   AND h.bank_account_num = :p_bank_account_number
 GROUP BY h.statement_number, h.bank_account_num,
          h.statement_date, h.record_status_flag;

-- 导入后的 Statement
SELECT csh.statement_header_id,
       csh.statement_number,
       csh.statement_date,
       csh.control_begin_balance,
       csh.control_end_balance,
       COUNT(csl.statement_line_id) line_count,
       SUM(csl.amount) line_amount
  FROM ce_statement_headers csh
  LEFT JOIN ce_statement_lines csl
    ON csl.statement_header_id = csh.statement_header_id
 WHERE csh.statement_number = :p_statement_number
   AND csh.bank_account_id = :p_bank_account_id
 GROUP BY csh.statement_header_id, csh.statement_number,
          csh.statement_date, csh.control_begin_balance,
          csh.control_end_balance;
```

Bank Statement Import 以整张 Statement 验证；任一行失败可能导致该 Statement 不导入。错误应从 Bank Statement Interface 页面、AutoReconciliation Execution Report 和请求日志联合定位。

<a id="src-docs-07-ce-tax-interfaces--5-外部资金交易参与-ce-对账"></a>
#### 5. 外部资金交易参与 CE 对账

当交易仍保存在外部 Treasury/支付平台、但需要和 EBS Bank Statement 匹配时，实现 `CE_999_INTERFACE_V`：

```sql
CREATE OR REPLACE VIEW ce_999_interface_v AS
SELECT x.external_transaction_id row_id,
       x.bank_account_id,
       x.transaction_number,
       x.transaction_date,
       x.currency_code,
       x.amount,
       x.status
  FROM xxtreasury_open_transactions x
 WHERE x.status IN ('AVAILABLE', 'CLEARED', 'RECONCILED');
```

上例只说明 View 封装方式，真实 View 必须严格实现 Oracle Cash Management User Guide 列出的全部列、数据类型和状态语义。目标银行账户还需启用 Use Reconciliation Open Interfaces 并配置 Matching Criteria、Float Status、Clear Status。

<a id="src-docs-07-ce-tax-interfaces--6-oracle-payments-出站文件"></a>
#### 6. Oracle Payments 出站文件

生产付款链路应使用 IBY 标准对象：

```text
AP Selected Invoices
→ Payment Process Request
→ Payment Instruction
→ BI Publisher Payment Format
→ Transmission Configuration
→ Bank/SWIFT
→ Technical ACK + Business ACK
→ Payment/Clearing/Reconciliation
```

不要直接插入或更新 `IBY_*` 业务表生成付款。常用实施控制：

- 每个 Payment Instruction 只生成一个版本化文件，重传不重新付款；
- 文件使用 PGP/银行签名，SFTP/HTTPS 凭据进入 Wallet/Secrets，不写源码；
- 保存 File Hash、Instruction ID、Transmission ID、银行 Message ID 和 ACK 状态；
- 区分“文件传输成功”“银行接收”“银行受理”“资金结算”四种状态；
- Positive Pay、付款回执和退票文件独立建消息类型及幂等键。

```sql
SELECT ppr.payment_service_request_id,
       ppr.calling_app_doc_unique_ref1,
       ppr.payment_service_request_status,
       pi.payment_instruction_id,
       pi.payment_instruction_status,
       pi.generate_sep_remit_advice_flag
  FROM iby_pay_service_requests ppr
  LEFT JOIN iby_pay_instructions_all pi
    ON pi.payment_service_request_id = ppr.payment_service_request_id
 WHERE ppr.payment_service_request_id = :p_payment_service_request_id;
```

具体列应在目标实例 eTRM 复核；排查时还要查 Payments/Format/Transmission 并发日志和银行 ACK。

<a id="src-docs-07-ce-tax-interfaces--7-ebtax-接口实现"></a>
#### 7. EBTax 接口实现

税由 EBTax 根据 Regime、Tax、Status、Rate、Party/Place、Product、Fiscal Classification 和税务日期判定。外部系统通常只传 Tax Determining Factors：

```sql
-- AP 发票接口行示例片段
INSERT INTO ap_invoice_lines_interface (
  invoice_id,
  invoice_line_id,
  line_number,
  line_type_lookup_code,
  amount,
  tax_classification_code,
  ship_to_location_id,
  org_id
) VALUES (
  :p_invoice_id,
  :p_invoice_line_id,
  :p_line_number,
  'ITEM',
  :p_amount,
  :p_tax_classification_code,
  :p_ship_to_location_id,
  :p_org_id
);

-- AR AutoInvoice 行可传相同思想的税分类
UPDATE ra_interface_lines_all
   SET tax_classification_code = :p_tax_classification_code
 WHERE interface_line_id = :p_interface_line_id;
```

是否允许源系统传税额、税率或手工税行取决于产品规则、Batch Source、Configuration Owner Tax Options 和法规。不要直接写 `ZX_LINES`、`ZX_DETAIL_TAX_LINES_GT` 等 EBTax 业务/临时表。

<a id="src-docs-07-ce-tax-interfaces--8-常见问题"></a>
#### 8. 常见问题

| 症状 | 常见原因 | 排查方法 |
| --- | --- | --- |
| Bank Statement Import Error | Bank Account 不唯一/未定义、币种/交易码/金额无效 | 查 Interface 页面和 Import Validation Report |
| 同日对账单重复 | 未使用 Statement Number/Timestamp/File Hash | 在 Landing 层做银行账户范围唯一约束 |
| 自动对账率低 | Transaction Code、匹配规则、Reference/日期容差不合理 | 按银行交易类型分组分析未匹配原因 |
| 付款文件已传但状态未知 | 只处理技术 ACK、缺业务 ACK | 建四级状态并用银行 Message ID 查询 |
| AP/AR 税未计算或错误 | Tax Classification、Party/Place、日期、Registration 缺失 | 查 EBTax Determining Factors，不手改 ZX 行 |

<a id="src-docs-07-ce-tax-interfaces--9-关联文档"></a>
#### 9. 关联文档

- [现金管理和银行接口](#src-docs-07-ce-tax-cash-forecast-interfaces)
- [EBTax](#src-docs-07-ce-tax-ebtax)
- [CE/EBTax 常用表](#src-docs-07-ce-tax-tables)
- [AR AutoLockbox](04-credit-to-cash.md#src-docs-03-ar-interfaces)

<a id="src-docs-07-ce-tax-interfaces--10-官方参考"></a>
#### 10. 官方参考

- [Oracle Cash Management User Guide: Bank Statement Open Interface](https://docs.oracle.com/cd/E26401_01/doc.122/e48900/T359831T359835.htm)
- [Oracle Cash Management User Guide: Bank Statement Validation](https://docs.oracle.com/cd/E26401_01/doc.122/e48900/T359831T359836.htm)
- [Oracle Cash Management User Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48900/)
- [Oracle E-Business Tax User Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48750/)


<!-- source: docs/07-ce-tax/tables.md -->
<a id="src-docs-07-ce-tax-tables"></a>
### Cash Management / Payments / EBTax 常用表结构


<a id="src-docs-07-ce-tax-tables--业务说明"></a>
#### 业务说明

R12 银行与分行是 TCA Party，内部银行账户在 CE，账户所有者与 OU/应用用途决定 AP/AR 可见性，支付指令/文件在 IBY，银行对账单在 CE。EBTax 以 Application+Entity+Event Class+Transaction/Line 标识税务交易，不能仅按 `TRX_ID` 跨模块查税行。

<a id="src-docs-07-ce-tax-tables--表级速查"></a>
#### 表级速查

| 表 | 中文名 | 粒度/用途 | 关键字段 |
| --- | --- | --- | --- |
| `CE_BANK_ACCOUNTS` | 内部银行账户 | 每个银行账户 | `BANK_ACCOUNT_ID`, `BANK_ACCOUNT_NUM`, `CURRENCY_CODE` |
| `CE_BANK_ACCT_USES_ALL` | 银行账户 OU 用途 | Account+OU+Application Use | `BANK_ACCT_USE_ID`, `BANK_ACCOUNT_ID`, `ORG_ID` |
| `CE_STATEMENT_HEADERS` | 银行对账单头 | Account+Statement | `STATEMENT_HEADER_ID`, `BANK_ACCOUNT_ID`, `STATEMENT_NUMBER` |
| `CE_STATEMENT_LINES` | 银行对账单行 | 每条银行流水 | `STATEMENT_LINE_ID`, `TRX_TYPE`, `TRX_CODE`, `STATUS` |
| `CE_STATEMENT_HEADERS_INT` | 银行对账单头接口 | 待导入 Statement | 银行账户/对账单标识与控制金额 |
| `CE_STATEMENT_LINES_INTERFACE` | 银行对账单行接口 | 待导入流水 | 交易编码、日期、金额、参考 |
| `IBY_PAY_SERVICE_REQUESTS` | 付款服务请求/PPR | 每个 Payment Process Request | `PAYMENT_SERVICE_REQUEST_ID`, `CALL_APP_PAY_SERVICE_REQ_CODE`, `PAYMENT_SERVICE_REQUEST_STATUS` |
| `IBY_PAY_INSTRUCTIONS_ALL` | 付款指令 | 每个银行/格式/支付分组 | `PAYMENT_INSTRUCTION_ID`, `PAYMENT_STATUS` |
| `IBY_PAYMENTS_ALL` | IBY 付款 | 每笔支付 | `PAYMENT_ID`, `PAYMENT_INSTRUCTION_ID`, `PAYMENT_STATUS` |
| `IBY_EXT_BANK_ACCOUNTS` | 外部收款人银行账户 | 每个外部账户 | `EXT_BANK_ACCOUNT_ID`, Country/Bank/Branch/Account |
| `ZX_LINES` | EBTax 税行 | 源交易行的每个税 | `TAX_LINE_ID`, `APPLICATION_ID`, `TRX_ID`, `TRX_LINE_ID` |
| `ZX_LINES_DET_FACTORS` | 税确定因素 | 交易/行的税务输入快照 | Entity/Event/Trx/Line 组合 |
| `ZX_RATES_B` | 税率 | Regime+Tax+Status+Rate+有效期 | `TAX_RATE_ID`, `TAX_RATE_CODE`, `PERCENTAGE_RATE` |
| `ZX_PARTY_TAX_PROFILE` | 交易方税务档案 | Party/Party Site/LE/OU 税务属性 | `PARTY_TAX_PROFILE_ID`, `PARTY_ID`, `PARTY_TYPE_CODE` |
| `ZX_REGISTRATIONS` | 税务登记 | Party Tax Profile+Regime/Tax/Jurisdiction | `REGISTRATION_ID`, `REGISTRATION_NUMBER`, Effective Dates |

<a id="src-docs-07-ce-tax-tables--ce-银行账户与对账单"></a>
#### CE 银行账户与对账单

<a id="src-docs-07-ce-tax-tables--cebankaccounts"></a>
##### `CE_BANK_ACCOUNTS`

| 字段 | 中文名 | 说明 |
| --- | --- | --- |
| `BANK_ACCOUNT_ID` | 内部银行账户 ID | AP/AR/CE/IBY 关联的核心键 |
| `BANK_ACCOUNT_NUM` | 银行账号 | 敏感数据，报表/日志应脱敏 |
| `CURRENCY_CODE` | 账户币种 | 多币种能力还受 Account Setup 影响 |
| `ACCOUNT_OWNER_ORG_ID` | 账户所有者组织 | 通常为 Legal Entity/Legal Context，不等于具体 OU Use |
| `START_DATE/END_DATE` | 有效期 | 应与付款/收款/对账业务日期比较 |

<a id="src-docs-07-ce-tax-tables--cestatementlines"></a>
##### `CE_STATEMENT_LINES`

| 字段 | 中文名 | 业务含义 |
| --- | --- | --- |
| `TRX_TYPE` | 银行交易类型 | Debit/Credit、Payment/Receipt 等高层类型，以 CE Lookup 为准 |
| `TRX_CODE` | 银行交易代码 | 应映射 CE Bank Transaction Code，决定 AutoReconciliation 行为 |
| `AMOUNT` | 流水金额 | 正负号与 Debit/Credit 规则由文件格式/交易编码定义 |
| `STATUS` | 对账状态 | Unreconciled/Reconciled/Error 等含义，用 CE Lookup 解码 |
| `BANK_TRX_NUMBER` | 银行交易号 | 自动匹配和重复检查的重要参考，但不应单独作为全局唯一键 |

<a id="src-docs-07-ce-tax-tables--iby-支付状态"></a>
#### IBY 支付状态

PPR → Proposed Payments → Payment Instruction → Payment File/Transmission 是分层模型。`PAYMENT_SERVICE_REQUEST_STATUS`、`PAYMENT_STATUS` 会出现 Submitted、Assigning/Validation、Formatting、Formatted、Transmitted、Confirmed、Failed/Terminated 等业务含义，必须使用 IBY Lookup 和 PPR 日志解码。上层 PPR 完成不代表每笔 Payment 都已被银行接受。

<a id="src-docs-07-ce-tax-tables--zxlines-税行"></a>
#### `ZX_LINES` — 税行

| 字段 | 中文名 | 业务说明 |
| --- | --- | --- |
| `APPLICATION_ID` | 源应用 ID | AP、AR、PO 等，决定 `TRX_ID` 指向哪个产品对象 |
| `ENTITY_CODE/EVENT_CLASS_CODE` | 实体/事件类 | 如 AP Invoice/AR Transaction 中的具体事件模型 |
| `TRX_ID/TRX_LINE_ID/TRX_LEVEL_TYPE` | 源交易/行/层级 | 必须结合 Application/Entity 解读 |
| `TAX_REGIME_CODE/TAX` | 税制/税 | 税务设置主线 |
| `TAX_STATUS_CODE/TAX_RATE_CODE` | 税状态/税率码 | 规则确定结果，受有效期和 Owner 影响 |
| `TAX_RATE` | 税率数值 | 不等于 Tax Rate Code；税额还受 Basis/Rounding/Inclusive 影响 |
| `TAXABLE_AMT/TAX_AMT` | 计税基础/税额 | 注意交易币、税币与本位币列的区别 |
| `CANCEL_FLAG/DELETE_FLAG` | 取消/删除标志 | 历史行可保留，当前税查询应正确过滤 |

<a id="src-docs-07-ce-tax-tables--官方参考"></a>
#### 官方参考

- [Oracle Cash Management User Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48900/)
- [Oracle Financials Implementation Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48783/)
- [Oracle E-Business Suite eTRM User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/)


<!-- source: docs/07-ce-tax/tax-reporting-localization.md -->
<a id="src-docs-07-ce-tax-tax-reporting-localization"></a>
### EBTax 税务报告、本地化与合规控制


<a id="src-docs-07-ce-tax-tax-reporting-localization--原则"></a>
#### 原则

EBTax 负责交易税务确定，不等于自动满足某一国家/地区的全部申报、电子发票或归档要求。税务报告、本地化功能和外部税引擎均应按部署国家、法规版本、许可证、补丁和法定顾问意见实施。

<a id="src-docs-07-ce-tax-tax-reporting-localization--从交易到申报的控制链"></a>
#### 从交易到申报的控制链

```text
Legal Entity / Registration / Party Tax Profile
  → Tax Regime / Tax / Status / Rate / Jurisdiction
  → Determining Factors / Applicability / Recovery
  → ZX Tax Line / AP-AR-PO-OM 交易
  → SLA / GL / Tax Reporting Ledger（如适用）
  → 法定报表、电子申报/外部系统、归档与对账
```

<a id="src-docs-07-ce-tax-tax-reporting-localization--实施清单"></a>
#### 实施清单

- 每个法人/登记主体明确税号、注册地址、税务管辖、有效期、开票主体和报表责任人。
- 对标准、免税、零税率、反向计税、自行计税、代扣税、可抵扣/不可抵扣和复合税建立可测试的业务矩阵。
- 外部税引擎或本地化适配器应设计超时/不可用时的业务策略、版本控制、审计请求/响应摘要和日终对账。
- 税务申报前按交易、税行、税率、登记、会计期间和 GL 口径交叉核对；由税务负责人签字确认。

<a id="src-docs-07-ce-tax-tax-reporting-localization--sql税务行范围校验"></a>
#### SQL：税务行范围校验

```sql
-- ZX_LINES 的对象/字段需按目标补丁级别复核；查询必须按来源交易或日期收缩。
select zl.trx_id,
       zl.trx_line_id,
       zl.tax_regime_code,
       zl.tax,
       zl.tax_status_code,
       zl.tax_rate_code,
       zl.tax_amt,
       zl.taxable_amt
  from zx_lines zl
 where zl.trx_id = :p_trx_id
 order by zl.trx_line_id, zl.tax_line_id;
```

<a id="src-docs-07-ce-tax-tax-reporting-localization--常见问题"></a>
#### 常见问题

- 税率未命中：按交易日期、注册、地点、产品/税分类、客户/供应商税务档案和确定因素逐层检查。
- 税务金额正确但申报不一致：检查报告口径、会计/申报期间、取消/冲销交易、税务登记有效期和外部提取批次。
- 国家本地化需求：不要以通用 EBTax 设置替代法定评估；需同步核对 Oracle 本地化文档、MOS 补丁和当地法规。

<a id="src-docs-07-ce-tax-tax-reporting-localization--官方参考"></a>
#### 官方参考

- [Oracle E-Business Tax Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
- [Oracle Financials Localizations Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/07-ce-tax/treasury-bank-governance.md -->
<a id="src-docs-07-ce-tax-treasury-bank-governance"></a>
### Treasury、现金头寸与银行主数据治理


<a id="src-docs-07-ce-tax-treasury-bank-governance--适用范围"></a>
#### 适用范围

Oracle Treasury 是可选产品，处理资金交易、交易对手、限额、结算、估值和风险暴露；Cash Management 处理银行对账、现金头寸及预测。即使未部署 Treasury，内部银行账户、账户用途、签字权限、支付文件和外部对账单也必须作为跨财务模块的受控主数据。

<a id="src-docs-07-ce-tax-treasury-bank-governance--治理模型"></a>
#### 治理模型

| 对象 | 数据所有者 | 变更控制 | 对账责任 |
| --- | --- | --- | --- |
| Bank / Branch | 财资主数据团队 | 银行证明、双人复核、有效期 | 银行档案与账户清单 |
| Internal Account / Use | 法人财务与资金 | 所有者、用途、币种、模块授权 | AP/AR/CE/IBY 使用范围 |
| Counterparty / Limit | 财资风险团队 | KYC/信用审批、额度与有效期 | 敞口与限额报表 |
| Statement / Transaction Code | 资金运营 | 文件格式、代码映射、测试回放 | 银行余额、未达项、自动核对率 |

<a id="src-docs-07-ce-tax-treasury-bank-governance--现金头寸与预测"></a>
#### 现金头寸与预测

- 区分已确认银行余额、预计收款/付款、Treasury 结算、未达项和内部资金归集；不可将预测值当作已核对可用余额。
- 预测数据应携带来源、日期、币种、账户、置信状态和相关业务键，支持按账户/法人/币种/日期重算。
- 银行文件接口需要文件级哈希或唯一标识、防重复导入、原始文件受控留存、解析错误隔离和可审计重放。

<a id="src-docs-07-ce-tax-treasury-bank-governance--只读诊断-sql"></a>
#### 只读诊断 SQL

```sql
-- 银行账户对象的可用列在不同补丁级别可能不同；先校验数据字典。
select cba.bank_account_id,
       cba.bank_account_name,
       cba.currency_code,
       cba.start_date,
       cba.end_date
  from ce_bank_accounts cba
 where cba.bank_account_id = :p_bank_account_id;

-- 对账单行诊断必须缩小到账户、日期或对账单，避免全表扫描。
select csl.statement_line_id,
       csl.trx_date,
       csl.amount,
       csl.trx_code,
       csl.reconciliation_status
  from ce_statement_lines csl
 where csl.statement_header_id = :p_statement_header_id
 order by csl.line_number;
```

<a id="src-docs-07-ce-tax-treasury-bank-governance--官方参考"></a>
#### 官方参考

- [Oracle Treasury Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
- [Oracle Cash Management Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)

<!-- 兼容旧版目录与学习材料的定位锚点；正文已按主题重编。 -->
<a id="src-docs-06-cash-tax-bank-account-transfer-and-pooling-readme"></a>
<a id="src-docs-06-cash-tax-bank-account-transfer-and-pooling-readme--业务定位"></a>
<a id="src-docs-06-cash-tax-bank-account-transfer-and-pooling-readme--关联与官方依据"></a>
<a id="src-docs-06-cash-tax-bank-account-transfer-and-pooling-readme--实施边界"></a>
<a id="src-docs-06-cash-tax-bank-account-transfer-and-pooling-readme--常见问题与排查"></a>
<a id="src-docs-06-cash-tax-bank-account-transfer-and-pooling-readme--数据接口与会计追溯"></a>
<a id="src-docs-06-cash-tax-bank-account-transfer-and-pooling-readme--设计与配置"></a>
<a id="src-docs-06-cash-tax-bank-statement-integration-readme"></a>
<a id="src-docs-06-cash-tax-bank-statement-integration-readme--业务定位"></a>
<a id="src-docs-06-cash-tax-bank-statement-integration-readme--关联与官方依据"></a>
<a id="src-docs-06-cash-tax-bank-statement-integration-readme--实施边界"></a>
<a id="src-docs-06-cash-tax-bank-statement-integration-readme--常见问题与排查"></a>
<a id="src-docs-06-cash-tax-bank-statement-integration-readme--数据接口与会计追溯"></a>
<a id="src-docs-06-cash-tax-bank-statement-integration-readme--设计与配置"></a>
<a id="src-docs-06-cash-tax-cash-management-readme"></a>
<a id="src-docs-06-cash-tax-cash-management-readme--业务定位"></a>
<a id="src-docs-06-cash-tax-cash-management-readme--关联与官方依据"></a>
<a id="src-docs-06-cash-tax-cash-management-readme--实施边界"></a>
<a id="src-docs-06-cash-tax-cash-management-readme--常见问题与排查"></a>
<a id="src-docs-06-cash-tax-cash-management-readme--数据接口与会计追溯"></a>
<a id="src-docs-06-cash-tax-cash-management-readme--设计与配置"></a>
<a id="src-docs-06-cash-tax-cash-position-and-forecast-readme"></a>
<a id="src-docs-06-cash-tax-cash-position-and-forecast-readme--业务定位"></a>
<a id="src-docs-06-cash-tax-cash-position-and-forecast-readme--关联与官方依据"></a>
<a id="src-docs-06-cash-tax-cash-position-and-forecast-readme--实施边界"></a>
<a id="src-docs-06-cash-tax-cash-position-and-forecast-readme--常见问题与排查"></a>
<a id="src-docs-06-cash-tax-cash-position-and-forecast-readme--数据接口与会计追溯"></a>
<a id="src-docs-06-cash-tax-cash-position-and-forecast-readme--设计与配置"></a>
<a id="src-docs-06-cash-tax-e-business-tax-readme"></a>
<a id="src-docs-06-cash-tax-e-business-tax-readme--业务定位"></a>
<a id="src-docs-06-cash-tax-e-business-tax-readme--关联与官方依据"></a>
<a id="src-docs-06-cash-tax-e-business-tax-readme--实施边界"></a>
<a id="src-docs-06-cash-tax-e-business-tax-readme--常见问题与排查"></a>
<a id="src-docs-06-cash-tax-e-business-tax-readme--数据接口与会计追溯"></a>
<a id="src-docs-06-cash-tax-e-business-tax-readme--设计与配置"></a>
<a id="src-docs-06-cash-tax-readme"></a>
<a id="src-docs-06-cash-tax-readme--与既有知识的关系"></a>
<a id="src-docs-06-cash-tax-readme--官方依据"></a>
<a id="src-docs-06-cash-tax-readme--核心数据对象"></a>
<a id="src-docs-06-cash-tax-readme--范围与目标"></a>
<a id="src-docs-06-cash-tax-readme--运行与实施控制"></a>
<a id="src-docs-06-cash-tax-tax-engines-and-external-services-readme"></a>
<a id="src-docs-06-cash-tax-tax-engines-and-external-services-readme--业务定位"></a>
<a id="src-docs-06-cash-tax-tax-engines-and-external-services-readme--关联与官方依据"></a>
<a id="src-docs-06-cash-tax-tax-engines-and-external-services-readme--实施边界"></a>
<a id="src-docs-06-cash-tax-tax-engines-and-external-services-readme--常见问题与排查"></a>
<a id="src-docs-06-cash-tax-tax-engines-and-external-services-readme--数据接口与会计追溯"></a>
<a id="src-docs-06-cash-tax-tax-engines-and-external-services-readme--设计与配置"></a>
<a id="src-docs-06-cash-tax-tax-reporting-ledger-readme"></a>
<a id="src-docs-06-cash-tax-tax-reporting-ledger-readme--业务定位"></a>
<a id="src-docs-06-cash-tax-tax-reporting-ledger-readme--关联与官方依据"></a>
<a id="src-docs-06-cash-tax-tax-reporting-ledger-readme--实施边界"></a>
<a id="src-docs-06-cash-tax-tax-reporting-ledger-readme--常见问题与排查"></a>
<a id="src-docs-06-cash-tax-tax-reporting-ledger-readme--数据接口与会计追溯"></a>
<a id="src-docs-06-cash-tax-tax-reporting-ledger-readme--设计与配置"></a>
<a id="src-docs-06-cash-tax-treasury-readme"></a>
<a id="src-docs-06-cash-tax-treasury-readme--业务定位"></a>
<a id="src-docs-06-cash-tax-treasury-readme--关联与官方依据"></a>
<a id="src-docs-06-cash-tax-treasury-readme--实施边界"></a>
<a id="src-docs-06-cash-tax-treasury-readme--常见问题与排查"></a>
<a id="src-docs-06-cash-tax-treasury-readme--数据接口与会计追溯"></a>
<a id="src-docs-06-cash-tax-treasury-readme--设计与配置"></a>
