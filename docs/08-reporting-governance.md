# 报表、关账与治理

> 财务/管理报表、BI Publisher、FSG、ECC、内控、审计与本地化。本文件由原目录中的 11 份资料合并而成；各章节保留原来源标记，便于审计与后续去重。

## 本模块章节导航

- [Reporting and Governance](#src-docs-08-reporting-governance-readme)（原 `docs/08-reporting-governance/README.md`）
- [Reporting and Governance： audit-and-data-retention](#src-docs-08-reporting-governance-audit-and-data-retention-readme)（原 `docs/08-reporting-governance/audit-and-data-retention/README.md`）
- [Reporting and Governance： bi-publisher-and-rxi](#src-docs-08-reporting-governance-bi-publisher-and-rxi-readme)（原 `docs/08-reporting-governance/bi-publisher-and-rxi/README.md`）
- [Reporting and Governance： enterprise-command-center](#src-docs-08-reporting-governance-enterprise-command-center-readme)（原 `docs/08-reporting-governance/enterprise-command-center/README.md`）
- [Reporting and Governance： financial-reporting](#src-docs-08-reporting-governance-financial-reporting-readme)（原 `docs/08-reporting-governance/financial-reporting/README.md`）
- [Reporting and Governance： fsg-smart-view-webadi](#src-docs-08-reporting-governance-fsg-smart-view-webadi-readme)（原 `docs/08-reporting-governance/fsg-smart-view-webadi/README.md`）
- [Reporting and Governance： internal-controls-and-sox](#src-docs-08-reporting-governance-internal-controls-and-sox-readme)（原 `docs/08-reporting-governance/internal-controls-and-sox/README.md`）
- [Reporting and Governance： localizations-and-regulatory](#src-docs-08-reporting-governance-localizations-and-regulatory-readme)（原 `docs/08-reporting-governance/localizations-and-regulatory/README.md`）
- [Reporting and Governance： management-reporting](#src-docs-08-reporting-governance-management-reporting-readme)（原 `docs/08-reporting-governance/management-reporting/README.md`）
- [Reporting and Governance： period-close-and-reconciliation](#src-docs-08-reporting-governance-period-close-and-reconciliation-readme)（原 `docs/08-reporting-governance/period-close-and-reconciliation/README.md`）
- [Reporting and Governance： public-sector-and-federal](#src-docs-08-reporting-governance-public-sector-and-federal-readme)（原 `docs/08-reporting-governance/public-sector-and-federal/README.md`）

---

<!-- source: docs/08-reporting-governance/README.md -->
<a id="src-docs-08-reporting-governance-readme"></a>
## Reporting and Governance


<a id="src-docs-08-reporting-governance-readme--范围与目标"></a>
### 范围与目标
覆盖关账与对账、财务/管理报告、BI Publisher/RXi、FSG/Smart View/Web ADI、ECC、内控、审计、保留、本地化和公共部门。

<a id="src-docs-08-reporting-governance-readme--运行与实施控制"></a>
### 运行与实施控制
先明确报告口径、数据截止、Ledger/币种/组织范围和签字责任；报告、对账、审计和数据保留使用一致的业务键与期间。

<a id="src-docs-08-reporting-governance-readme--核心数据对象"></a>
### 核心数据对象
GL_BALANCES、GL_JE_HEADERS、XLA_AE_HEADERS、FND_CONCURRENT_REQUESTS、FND_AUDIT_TABLES、FND_LOGINS。对象、列、状态和 API 签名须在目标实例 eTRM、Integration Repository 与数据字典复核。

<a id="src-docs-08-reporting-governance-readme--与既有知识的关系"></a>
### 与既有知识的关系
本目标目录新增详细入口；已有专题保留在 [04-gl/README](02-record-to-report.md#src-docs-04-gl-readme) 并逐步迁移链接，不复制历史内容。

<a id="src-docs-08-reporting-governance-readme--官方依据"></a>
### 官方依据
[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/08-reporting-governance/audit-and-data-retention/README.md -->
<a id="src-docs-08-reporting-governance-audit-and-data-retention-readme"></a>
## Reporting and Governance： audit-and-data-retention


<a id="src-docs-08-reporting-governance-audit-and-data-retention-readme--业务定位"></a>
### 业务定位
本专题是 Reporting and Governance 中的 audit-and-data-retention 子域。覆盖关账与对账、财务/管理报告、BI Publisher/RXi、FSG/Smart View/Web ADI、ECC、内控、审计、保留、本地化和公共部门。

<a id="src-docs-08-reporting-governance-audit-and-data-retention-readme--设计与配置"></a>
### 设计与配置
先明确报告口径、数据截止、Ledger/币种/组织范围和签字责任；报告、对账、审计和数据保留使用一致的业务键与期间。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-08-reporting-governance-audit-and-data-retention-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
GL_BALANCES、GL_JE_HEADERS、XLA_AE_HEADERS、FND_CONCURRENT_REQUESTS、FND_AUDIT_TABLES、FND_LOGINS。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-08-reporting-governance-audit-and-data-retention-readme--常见问题与排查"></a>
### 常见问题与排查
报表只按显示金额判断正确；将业务修复留给报表层；未确认 ECC、公共部门、本地化和诊断工具的许可证/适用范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-08-reporting-governance-audit-and-data-retention-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-08-reporting-governance-audit-and-data-retention-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-08-reporting-governance-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/08-reporting-governance/bi-publisher-and-rxi/README.md -->
<a id="src-docs-08-reporting-governance-bi-publisher-and-rxi-readme"></a>
## Reporting and Governance： bi-publisher-and-rxi


<a id="src-docs-08-reporting-governance-bi-publisher-and-rxi-readme--业务定位"></a>
### 业务定位
本专题是 Reporting and Governance 中的 bi-publisher-and-rxi 子域。覆盖关账与对账、财务/管理报告、BI Publisher/RXi、FSG/Smart View/Web ADI、ECC、内控、审计、保留、本地化和公共部门。

<a id="src-docs-08-reporting-governance-bi-publisher-and-rxi-readme--设计与配置"></a>
### 设计与配置
先明确报告口径、数据截止、Ledger/币种/组织范围和签字责任；报告、对账、审计和数据保留使用一致的业务键与期间。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-08-reporting-governance-bi-publisher-and-rxi-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
GL_BALANCES、GL_JE_HEADERS、XLA_AE_HEADERS、FND_CONCURRENT_REQUESTS、FND_AUDIT_TABLES、FND_LOGINS。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-08-reporting-governance-bi-publisher-and-rxi-readme--常见问题与排查"></a>
### 常见问题与排查
报表只按显示金额判断正确；将业务修复留给报表层；未确认 ECC、公共部门、本地化和诊断工具的许可证/适用范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-08-reporting-governance-bi-publisher-and-rxi-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-08-reporting-governance-bi-publisher-and-rxi-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-08-reporting-governance-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/08-reporting-governance/enterprise-command-center/README.md -->
<a id="src-docs-08-reporting-governance-enterprise-command-center-readme"></a>
## Reporting and Governance： enterprise-command-center


<a id="src-docs-08-reporting-governance-enterprise-command-center-readme--业务定位"></a>
### 业务定位
本专题是 Reporting and Governance 中的 enterprise-command-center 子域。覆盖关账与对账、财务/管理报告、BI Publisher/RXi、FSG/Smart View/Web ADI、ECC、内控、审计、保留、本地化和公共部门。

<a id="src-docs-08-reporting-governance-enterprise-command-center-readme--设计与配置"></a>
### 设计与配置
先明确报告口径、数据截止、Ledger/币种/组织范围和签字责任；报告、对账、审计和数据保留使用一致的业务键与期间。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-08-reporting-governance-enterprise-command-center-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
GL_BALANCES、GL_JE_HEADERS、XLA_AE_HEADERS、FND_CONCURRENT_REQUESTS、FND_AUDIT_TABLES、FND_LOGINS。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-08-reporting-governance-enterprise-command-center-readme--常见问题与排查"></a>
### 常见问题与排查
报表只按显示金额判断正确；将业务修复留给报表层；未确认 ECC、公共部门、本地化和诊断工具的许可证/适用范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-08-reporting-governance-enterprise-command-center-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-08-reporting-governance-enterprise-command-center-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-08-reporting-governance-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/08-reporting-governance/financial-reporting/README.md -->
<a id="src-docs-08-reporting-governance-financial-reporting-readme"></a>
## Reporting and Governance： financial-reporting


<a id="src-docs-08-reporting-governance-financial-reporting-readme--业务定位"></a>
### 业务定位
本专题是 Reporting and Governance 中的 financial-reporting 子域。覆盖关账与对账、财务/管理报告、BI Publisher/RXi、FSG/Smart View/Web ADI、ECC、内控、审计、保留、本地化和公共部门。

<a id="src-docs-08-reporting-governance-financial-reporting-readme--设计与配置"></a>
### 设计与配置
先明确报告口径、数据截止、Ledger/币种/组织范围和签字责任；报告、对账、审计和数据保留使用一致的业务键与期间。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-08-reporting-governance-financial-reporting-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
GL_BALANCES、GL_JE_HEADERS、XLA_AE_HEADERS、FND_CONCURRENT_REQUESTS、FND_AUDIT_TABLES、FND_LOGINS。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-08-reporting-governance-financial-reporting-readme--常见问题与排查"></a>
### 常见问题与排查
报表只按显示金额判断正确；将业务修复留给报表层；未确认 ECC、公共部门、本地化和诊断工具的许可证/适用范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-08-reporting-governance-financial-reporting-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-08-reporting-governance-financial-reporting-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-08-reporting-governance-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/08-reporting-governance/fsg-smart-view-webadi/README.md -->
<a id="src-docs-08-reporting-governance-fsg-smart-view-webadi-readme"></a>
## Reporting and Governance： fsg-smart-view-webadi


<a id="src-docs-08-reporting-governance-fsg-smart-view-webadi-readme--业务定位"></a>
### 业务定位
本专题是 Reporting and Governance 中的 fsg-smart-view-webadi 子域。覆盖关账与对账、财务/管理报告、BI Publisher/RXi、FSG/Smart View/Web ADI、ECC、内控、审计、保留、本地化和公共部门。

<a id="src-docs-08-reporting-governance-fsg-smart-view-webadi-readme--设计与配置"></a>
### 设计与配置
先明确报告口径、数据截止、Ledger/币种/组织范围和签字责任；报告、对账、审计和数据保留使用一致的业务键与期间。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-08-reporting-governance-fsg-smart-view-webadi-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
GL_BALANCES、GL_JE_HEADERS、XLA_AE_HEADERS、FND_CONCURRENT_REQUESTS、FND_AUDIT_TABLES、FND_LOGINS。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-08-reporting-governance-fsg-smart-view-webadi-readme--常见问题与排查"></a>
### 常见问题与排查
报表只按显示金额判断正确；将业务修复留给报表层；未确认 ECC、公共部门、本地化和诊断工具的许可证/适用范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-08-reporting-governance-fsg-smart-view-webadi-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-08-reporting-governance-fsg-smart-view-webadi-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-08-reporting-governance-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/08-reporting-governance/internal-controls-and-sox/README.md -->
<a id="src-docs-08-reporting-governance-internal-controls-and-sox-readme"></a>
## Reporting and Governance： internal-controls-and-sox


<a id="src-docs-08-reporting-governance-internal-controls-and-sox-readme--业务定位"></a>
### 业务定位
本专题是 Reporting and Governance 中的 internal-controls-and-sox 子域。覆盖关账与对账、财务/管理报告、BI Publisher/RXi、FSG/Smart View/Web ADI、ECC、内控、审计、保留、本地化和公共部门。

<a id="src-docs-08-reporting-governance-internal-controls-and-sox-readme--设计与配置"></a>
### 设计与配置
先明确报告口径、数据截止、Ledger/币种/组织范围和签字责任；报告、对账、审计和数据保留使用一致的业务键与期间。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-08-reporting-governance-internal-controls-and-sox-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
GL_BALANCES、GL_JE_HEADERS、XLA_AE_HEADERS、FND_CONCURRENT_REQUESTS、FND_AUDIT_TABLES、FND_LOGINS。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-08-reporting-governance-internal-controls-and-sox-readme--常见问题与排查"></a>
### 常见问题与排查
报表只按显示金额判断正确；将业务修复留给报表层；未确认 ECC、公共部门、本地化和诊断工具的许可证/适用范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-08-reporting-governance-internal-controls-and-sox-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-08-reporting-governance-internal-controls-and-sox-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-08-reporting-governance-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/08-reporting-governance/localizations-and-regulatory/README.md -->
<a id="src-docs-08-reporting-governance-localizations-and-regulatory-readme"></a>
## Reporting and Governance： localizations-and-regulatory


<a id="src-docs-08-reporting-governance-localizations-and-regulatory-readme--业务定位"></a>
### 业务定位
本专题是 Reporting and Governance 中的 localizations-and-regulatory 子域。覆盖关账与对账、财务/管理报告、BI Publisher/RXi、FSG/Smart View/Web ADI、ECC、内控、审计、保留、本地化和公共部门。

<a id="src-docs-08-reporting-governance-localizations-and-regulatory-readme--设计与配置"></a>
### 设计与配置
先明确报告口径、数据截止、Ledger/币种/组织范围和签字责任；报告、对账、审计和数据保留使用一致的业务键与期间。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-08-reporting-governance-localizations-and-regulatory-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
GL_BALANCES、GL_JE_HEADERS、XLA_AE_HEADERS、FND_CONCURRENT_REQUESTS、FND_AUDIT_TABLES、FND_LOGINS。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-08-reporting-governance-localizations-and-regulatory-readme--常见问题与排查"></a>
### 常见问题与排查
报表只按显示金额判断正确；将业务修复留给报表层；未确认 ECC、公共部门、本地化和诊断工具的许可证/适用范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-08-reporting-governance-localizations-and-regulatory-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-08-reporting-governance-localizations-and-regulatory-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-08-reporting-governance-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/08-reporting-governance/management-reporting/README.md -->
<a id="src-docs-08-reporting-governance-management-reporting-readme"></a>
## Reporting and Governance： management-reporting


<a id="src-docs-08-reporting-governance-management-reporting-readme--业务定位"></a>
### 业务定位
本专题是 Reporting and Governance 中的 management-reporting 子域。覆盖关账与对账、财务/管理报告、BI Publisher/RXi、FSG/Smart View/Web ADI、ECC、内控、审计、保留、本地化和公共部门。

<a id="src-docs-08-reporting-governance-management-reporting-readme--设计与配置"></a>
### 设计与配置
先明确报告口径、数据截止、Ledger/币种/组织范围和签字责任；报告、对账、审计和数据保留使用一致的业务键与期间。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-08-reporting-governance-management-reporting-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
GL_BALANCES、GL_JE_HEADERS、XLA_AE_HEADERS、FND_CONCURRENT_REQUESTS、FND_AUDIT_TABLES、FND_LOGINS。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-08-reporting-governance-management-reporting-readme--常见问题与排查"></a>
### 常见问题与排查
报表只按显示金额判断正确；将业务修复留给报表层；未确认 ECC、公共部门、本地化和诊断工具的许可证/适用范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-08-reporting-governance-management-reporting-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-08-reporting-governance-management-reporting-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-08-reporting-governance-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/08-reporting-governance/period-close-and-reconciliation/README.md -->
<a id="src-docs-08-reporting-governance-period-close-and-reconciliation-readme"></a>
## Reporting and Governance： period-close-and-reconciliation


<a id="src-docs-08-reporting-governance-period-close-and-reconciliation-readme--业务定位"></a>
### 业务定位
本专题是 Reporting and Governance 中的 period-close-and-reconciliation 子域。覆盖关账与对账、财务/管理报告、BI Publisher/RXi、FSG/Smart View/Web ADI、ECC、内控、审计、保留、本地化和公共部门。

<a id="src-docs-08-reporting-governance-period-close-and-reconciliation-readme--设计与配置"></a>
### 设计与配置
先明确报告口径、数据截止、Ledger/币种/组织范围和签字责任；报告、对账、审计和数据保留使用一致的业务键与期间。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-08-reporting-governance-period-close-and-reconciliation-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
GL_BALANCES、GL_JE_HEADERS、XLA_AE_HEADERS、FND_CONCURRENT_REQUESTS、FND_AUDIT_TABLES、FND_LOGINS。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-08-reporting-governance-period-close-and-reconciliation-readme--常见问题与排查"></a>
### 常见问题与排查
报表只按显示金额判断正确；将业务修复留给报表层；未确认 ECC、公共部门、本地化和诊断工具的许可证/适用范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-08-reporting-governance-period-close-and-reconciliation-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-08-reporting-governance-period-close-and-reconciliation-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-08-reporting-governance-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/08-reporting-governance/public-sector-and-federal/README.md -->
<a id="src-docs-08-reporting-governance-public-sector-and-federal-readme"></a>
## Reporting and Governance： public-sector-and-federal


<a id="src-docs-08-reporting-governance-public-sector-and-federal-readme--业务定位"></a>
### 业务定位
本专题是 Reporting and Governance 中的 public-sector-and-federal 子域。覆盖关账与对账、财务/管理报告、BI Publisher/RXi、FSG/Smart View/Web ADI、ECC、内控、审计、保留、本地化和公共部门。

<a id="src-docs-08-reporting-governance-public-sector-and-federal-readme--设计与配置"></a>
### 设计与配置
先明确报告口径、数据截止、Ledger/币种/组织范围和签字责任；报告、对账、审计和数据保留使用一致的业务键与期间。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-08-reporting-governance-public-sector-and-federal-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
GL_BALANCES、GL_JE_HEADERS、XLA_AE_HEADERS、FND_CONCURRENT_REQUESTS、FND_AUDIT_TABLES、FND_LOGINS。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-08-reporting-governance-public-sector-and-federal-readme--常见问题与排查"></a>
### 常见问题与排查
报表只按显示金额判断正确；将业务修复留给报表层；未确认 ECC、公共部门、本地化和诊断工具的许可证/适用范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-08-reporting-governance-public-sector-and-federal-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-08-reporting-governance-public-sector-and-federal-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-08-reporting-governance-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
