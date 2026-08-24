# 记录到报告（R2R）

> 总账、子账会计、FAH、AGIS、预算、重估、合并、报表与关账。本文件由原目录中的 19 份资料合并而成；各章节保留原来源标记，便于审计与后续去重。

## 本模块章节导航

- [Record to Report](#src-docs-02-record-to-report-readme)（原 `docs/02-record-to-report/README.md`）
- [Record to Report： agis-intercompany](#src-docs-02-record-to-report-agis-intercompany-readme)（原 `docs/02-record-to-report/agis-intercompany/README.md`）
- [Record to Report： budgetary-control](#src-docs-02-record-to-report-budgetary-control-readme)（原 `docs/02-record-to-report/budgetary-control/README.md`）
- [Record to Report： consolidation-and-elimination](#src-docs-02-record-to-report-consolidation-and-elimination-readme)（原 `docs/02-record-to-report/consolidation-and-elimination/README.md`）
- [Record to Report： financials-accounting-hub](#src-docs-02-record-to-report-financials-accounting-hub-readme)（原 `docs/02-record-to-report/financials-accounting-hub/README.md`）
- [Record to Report： general-ledger](#src-docs-02-record-to-report-general-ledger-readme)（原 `docs/02-record-to-report/general-ledger/README.md`）
- [Record to Report： record-to-report-close](#src-docs-02-record-to-report-record-to-report-close-readme)（原 `docs/02-record-to-report/record-to-report-close/README.md`）
- [Record to Report： secondary-ledger-reporting-currency](#src-docs-02-record-to-report-secondary-ledger-reporting-currency-readme)（原 `docs/02-record-to-report/secondary-ledger-reporting-currency/README.md`）
- [Record to Report： subledger-accounting](#src-docs-02-record-to-report-subledger-accounting-readme)（原 `docs/02-record-to-report/subledger-accounting/README.md`）
- [Oracle General Ledger（GL / Record to Report）](#src-docs-04-gl-readme)（原 `docs/04-gl/README.md`）
- [预算、预算控制与资金可用性](#src-docs-04-gl-budgetary-control)（原 `docs/04-gl/budgetary-control.md`）
- [GL 期间开关、月结、年结与报表](#src-docs-04-gl-close-reports)（原 `docs/04-gl/close-reports.md`）
- [合并、重估、折算与重复日记账](#src-docs-04-gl-consolidation-revaluation)（原 `docs/04-gl/consolidation-revaluation.md`）
- [Oracle General Ledger 接口实现案例](#src-docs-04-gl-interfaces)（原 `docs/04-gl/interfaces.md`）
- [GL 日记账来源、类别、审批与自动过账](#src-docs-04-gl-journals)（原 `docs/04-gl/journals.md`）
- [General Ledger 账簿、日记账与过账流程](#src-docs-04-gl-process)（原 `docs/04-gl/process.md`）
- [FSG、Smart View、Web ADI 与日记账导入](#src-docs-04-gl-reporting-interfaces)（原 `docs/04-gl/reporting-interfaces.md`）
- [SLA、Financials Accounting Hub 与 AGIS](#src-docs-04-gl-sla-fah-agis)（原 `docs/04-gl/sla-fah-agis.md`）
- [Oracle General Ledger 常用表结构](#src-docs-04-gl-tables)（原 `docs/04-gl/tables.md`）

---

<!-- source: docs/02-record-to-report/README.md -->
<a id="src-docs-02-record-to-report-readme"></a>
## Record to Report


<a id="src-docs-02-record-to-report-readme--范围与目标"></a>
### 范围与目标
覆盖总账、SLA、FAH、AGIS、预算、合并、二级账簿和月结，目标是使每个业务事件可从来源交易追溯至已过账日记账和报告。

<a id="src-docs-02-record-to-report-readme--运行与实施控制"></a>
### 运行与实施控制
先确认 Ledger、期间、Data Access、会计方法和来源系统；再验证事件、分录、传输、Journal Import、Posting、重估/折算/合并和报告。

<a id="src-docs-02-record-to-report-readme--核心数据对象"></a>
### 核心数据对象
GL_LEDGERS、GL_JE_BATCHES、GL_JE_HEADERS、GL_JE_LINES、GL_BALANCES、XLA_TRANSACTION_ENTITIES、XLA_EVENTS、XLA_AE_HEADERS、XLA_AE_LINES。对象、列、状态和 API 签名须在目标实例 eTRM、Integration Repository 与数据字典复核。

<a id="src-docs-02-record-to-report-readme--与既有知识的关系"></a>
### 与既有知识的关系
本目标目录新增详细入口；已有专题保留在 [04-gl/README](#src-docs-04-gl-readme) 并逐步迁移链接，不复制历史内容。

<a id="src-docs-02-record-to-report-readme--官方依据"></a>
### 官方依据
[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/02-record-to-report/agis-intercompany/README.md -->
<a id="src-docs-02-record-to-report-agis-intercompany-readme"></a>
## Record to Report： agis-intercompany


<a id="src-docs-02-record-to-report-agis-intercompany-readme--业务定位"></a>
### 业务定位
本专题是 Record to Report 中的 agis-intercompany 子域。覆盖总账、SLA、FAH、AGIS、预算、合并、二级账簿和月结，目标是使每个业务事件可从来源交易追溯至已过账日记账和报告。

<a id="src-docs-02-record-to-report-agis-intercompany-readme--设计与配置"></a>
### 设计与配置
先确认 Ledger、期间、Data Access、会计方法和来源系统；再验证事件、分录、传输、Journal Import、Posting、重估/折算/合并和报告。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-02-record-to-report-agis-intercompany-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
GL_LEDGERS、GL_JE_BATCHES、GL_JE_HEADERS、GL_JE_LINES、GL_BALANCES、XLA_TRANSACTION_ENTITIES、XLA_EVENTS、XLA_AE_HEADERS、XLA_AE_LINES。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-02-record-to-report-agis-intercompany-readme--常见问题与排查"></a>
### 常见问题与排查
子账差异被手工 GL 分录掩盖；未区分 Create Accounting、Transfer、Import 与 Posting 断点；跨 Ledger 汇率/转换规则未对账。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-02-record-to-report-agis-intercompany-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-02-record-to-report-agis-intercompany-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-02-record-to-report-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/02-record-to-report/budgetary-control/README.md -->
<a id="src-docs-02-record-to-report-budgetary-control-readme"></a>
## Record to Report： budgetary-control


<a id="src-docs-02-record-to-report-budgetary-control-readme--业务定位"></a>
### 业务定位
本专题是 Record to Report 中的 budgetary-control 子域。覆盖总账、SLA、FAH、AGIS、预算、合并、二级账簿和月结，目标是使每个业务事件可从来源交易追溯至已过账日记账和报告。

<a id="src-docs-02-record-to-report-budgetary-control-readme--设计与配置"></a>
### 设计与配置
先确认 Ledger、期间、Data Access、会计方法和来源系统；再验证事件、分录、传输、Journal Import、Posting、重估/折算/合并和报告。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-02-record-to-report-budgetary-control-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
GL_LEDGERS、GL_JE_BATCHES、GL_JE_HEADERS、GL_JE_LINES、GL_BALANCES、XLA_TRANSACTION_ENTITIES、XLA_EVENTS、XLA_AE_HEADERS、XLA_AE_LINES。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-02-record-to-report-budgetary-control-readme--常见问题与排查"></a>
### 常见问题与排查
子账差异被手工 GL 分录掩盖；未区分 Create Accounting、Transfer、Import 与 Posting 断点；跨 Ledger 汇率/转换规则未对账。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-02-record-to-report-budgetary-control-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-02-record-to-report-budgetary-control-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-02-record-to-report-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/02-record-to-report/consolidation-and-elimination/README.md -->
<a id="src-docs-02-record-to-report-consolidation-and-elimination-readme"></a>
## Record to Report： consolidation-and-elimination


<a id="src-docs-02-record-to-report-consolidation-and-elimination-readme--业务定位"></a>
### 业务定位
本专题是 Record to Report 中的 consolidation-and-elimination 子域。覆盖总账、SLA、FAH、AGIS、预算、合并、二级账簿和月结，目标是使每个业务事件可从来源交易追溯至已过账日记账和报告。

<a id="src-docs-02-record-to-report-consolidation-and-elimination-readme--设计与配置"></a>
### 设计与配置
先确认 Ledger、期间、Data Access、会计方法和来源系统；再验证事件、分录、传输、Journal Import、Posting、重估/折算/合并和报告。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-02-record-to-report-consolidation-and-elimination-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
GL_LEDGERS、GL_JE_BATCHES、GL_JE_HEADERS、GL_JE_LINES、GL_BALANCES、XLA_TRANSACTION_ENTITIES、XLA_EVENTS、XLA_AE_HEADERS、XLA_AE_LINES。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-02-record-to-report-consolidation-and-elimination-readme--常见问题与排查"></a>
### 常见问题与排查
子账差异被手工 GL 分录掩盖；未区分 Create Accounting、Transfer、Import 与 Posting 断点；跨 Ledger 汇率/转换规则未对账。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-02-record-to-report-consolidation-and-elimination-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-02-record-to-report-consolidation-and-elimination-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-02-record-to-report-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/02-record-to-report/financials-accounting-hub/README.md -->
<a id="src-docs-02-record-to-report-financials-accounting-hub-readme"></a>
## Record to Report： financials-accounting-hub


<a id="src-docs-02-record-to-report-financials-accounting-hub-readme--业务定位"></a>
### 业务定位
本专题是 Record to Report 中的 financials-accounting-hub 子域。覆盖总账、SLA、FAH、AGIS、预算、合并、二级账簿和月结，目标是使每个业务事件可从来源交易追溯至已过账日记账和报告。

<a id="src-docs-02-record-to-report-financials-accounting-hub-readme--设计与配置"></a>
### 设计与配置
先确认 Ledger、期间、Data Access、会计方法和来源系统；再验证事件、分录、传输、Journal Import、Posting、重估/折算/合并和报告。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-02-record-to-report-financials-accounting-hub-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
GL_LEDGERS、GL_JE_BATCHES、GL_JE_HEADERS、GL_JE_LINES、GL_BALANCES、XLA_TRANSACTION_ENTITIES、XLA_EVENTS、XLA_AE_HEADERS、XLA_AE_LINES。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-02-record-to-report-financials-accounting-hub-readme--常见问题与排查"></a>
### 常见问题与排查
子账差异被手工 GL 分录掩盖；未区分 Create Accounting、Transfer、Import 与 Posting 断点；跨 Ledger 汇率/转换规则未对账。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-02-record-to-report-financials-accounting-hub-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-02-record-to-report-financials-accounting-hub-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-02-record-to-report-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/02-record-to-report/general-ledger/README.md -->
<a id="src-docs-02-record-to-report-general-ledger-readme"></a>
## Record to Report： general-ledger


<a id="src-docs-02-record-to-report-general-ledger-readme--业务定位"></a>
### 业务定位
本专题是 Record to Report 中的 general-ledger 子域。覆盖总账、SLA、FAH、AGIS、预算、合并、二级账簿和月结，目标是使每个业务事件可从来源交易追溯至已过账日记账和报告。

<a id="src-docs-02-record-to-report-general-ledger-readme--设计与配置"></a>
### 设计与配置
先确认 Ledger、期间、Data Access、会计方法和来源系统；再验证事件、分录、传输、Journal Import、Posting、重估/折算/合并和报告。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-02-record-to-report-general-ledger-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
GL_LEDGERS、GL_JE_BATCHES、GL_JE_HEADERS、GL_JE_LINES、GL_BALANCES、XLA_TRANSACTION_ENTITIES、XLA_EVENTS、XLA_AE_HEADERS、XLA_AE_LINES。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-02-record-to-report-general-ledger-readme--常见问题与排查"></a>
### 常见问题与排查
子账差异被手工 GL 分录掩盖；未区分 Create Accounting、Transfer、Import 与 Posting 断点；跨 Ledger 汇率/转换规则未对账。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-02-record-to-report-general-ledger-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-02-record-to-report-general-ledger-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-02-record-to-report-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/02-record-to-report/record-to-report-close/README.md -->
<a id="src-docs-02-record-to-report-record-to-report-close-readme"></a>
## Record to Report： record-to-report-close


<a id="src-docs-02-record-to-report-record-to-report-close-readme--业务定位"></a>
### 业务定位
本专题是 Record to Report 中的 record-to-report-close 子域。覆盖总账、SLA、FAH、AGIS、预算、合并、二级账簿和月结，目标是使每个业务事件可从来源交易追溯至已过账日记账和报告。

<a id="src-docs-02-record-to-report-record-to-report-close-readme--设计与配置"></a>
### 设计与配置
先确认 Ledger、期间、Data Access、会计方法和来源系统；再验证事件、分录、传输、Journal Import、Posting、重估/折算/合并和报告。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-02-record-to-report-record-to-report-close-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
GL_LEDGERS、GL_JE_BATCHES、GL_JE_HEADERS、GL_JE_LINES、GL_BALANCES、XLA_TRANSACTION_ENTITIES、XLA_EVENTS、XLA_AE_HEADERS、XLA_AE_LINES。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-02-record-to-report-record-to-report-close-readme--常见问题与排查"></a>
### 常见问题与排查
子账差异被手工 GL 分录掩盖；未区分 Create Accounting、Transfer、Import 与 Posting 断点；跨 Ledger 汇率/转换规则未对账。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-02-record-to-report-record-to-report-close-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-02-record-to-report-record-to-report-close-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-02-record-to-report-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/02-record-to-report/secondary-ledger-reporting-currency/README.md -->
<a id="src-docs-02-record-to-report-secondary-ledger-reporting-currency-readme"></a>
## Record to Report： secondary-ledger-reporting-currency


<a id="src-docs-02-record-to-report-secondary-ledger-reporting-currency-readme--业务定位"></a>
### 业务定位
本专题是 Record to Report 中的 secondary-ledger-reporting-currency 子域。覆盖总账、SLA、FAH、AGIS、预算、合并、二级账簿和月结，目标是使每个业务事件可从来源交易追溯至已过账日记账和报告。

<a id="src-docs-02-record-to-report-secondary-ledger-reporting-currency-readme--设计与配置"></a>
### 设计与配置
先确认 Ledger、期间、Data Access、会计方法和来源系统；再验证事件、分录、传输、Journal Import、Posting、重估/折算/合并和报告。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-02-record-to-report-secondary-ledger-reporting-currency-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
GL_LEDGERS、GL_JE_BATCHES、GL_JE_HEADERS、GL_JE_LINES、GL_BALANCES、XLA_TRANSACTION_ENTITIES、XLA_EVENTS、XLA_AE_HEADERS、XLA_AE_LINES。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-02-record-to-report-secondary-ledger-reporting-currency-readme--常见问题与排查"></a>
### 常见问题与排查
子账差异被手工 GL 分录掩盖；未区分 Create Accounting、Transfer、Import 与 Posting 断点；跨 Ledger 汇率/转换规则未对账。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-02-record-to-report-secondary-ledger-reporting-currency-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-02-record-to-report-secondary-ledger-reporting-currency-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-02-record-to-report-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/02-record-to-report/subledger-accounting/README.md -->
<a id="src-docs-02-record-to-report-subledger-accounting-readme"></a>
## Record to Report： subledger-accounting


<a id="src-docs-02-record-to-report-subledger-accounting-readme--业务定位"></a>
### 业务定位
本专题是 Record to Report 中的 subledger-accounting 子域。覆盖总账、SLA、FAH、AGIS、预算、合并、二级账簿和月结，目标是使每个业务事件可从来源交易追溯至已过账日记账和报告。

<a id="src-docs-02-record-to-report-subledger-accounting-readme--设计与配置"></a>
### 设计与配置
先确认 Ledger、期间、Data Access、会计方法和来源系统；再验证事件、分录、传输、Journal Import、Posting、重估/折算/合并和报告。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-02-record-to-report-subledger-accounting-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
GL_LEDGERS、GL_JE_BATCHES、GL_JE_HEADERS、GL_JE_LINES、GL_BALANCES、XLA_TRANSACTION_ENTITIES、XLA_EVENTS、XLA_AE_HEADERS、XLA_AE_LINES。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-02-record-to-report-subledger-accounting-readme--常见问题与排查"></a>
### 常见问题与排查
子账差异被手工 GL 分录掩盖；未区分 Create Accounting、Transfer、Import 与 Posting 断点；跨 Ledger 汇率/转换规则未对账。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-02-record-to-report-subledger-accounting-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-02-record-to-report-subledger-accounting-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-02-record-to-report-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/04-gl/README.md -->
<a id="src-docs-04-gl-readme"></a>
## Oracle General Ledger（GL / Record to Report）


GL 是法人/账簿层面的最终会计记录与报告层。本目录覆盖日记账、预算控制、重估/折算/合并、报表、Journal Import 及期间关闭；SLA 规则的权威正文见 `01-common/sla.md`。

<a id="src-docs-04-gl-readme--专题导航"></a>
### 专题导航

- [账簿、日记账与过账流程](#src-docs-04-gl-process)
- [日记账来源、类别、审批与自动过账](#src-docs-04-gl-journals)
- [预算与资金控制](#src-docs-04-gl-budgetary-control)
- [合并、重估、折算与重复日记账](#src-docs-04-gl-consolidation-revaluation)
- [月结、年结与报表](#src-docs-04-gl-close-reports)
- [FSG、Smart View、Web ADI 与导入](#src-docs-04-gl-reporting-interfaces)
- [SLA、FAH 与 AGIS](#src-docs-04-gl-sla-fah-agis)
- [表结构](#src-docs-04-gl-tables)
- [`GL_INTERFACE` 实现](#src-docs-04-gl-interfaces)

<a id="src-docs-04-gl-readme--设计与关账原则"></a>
### 设计与关账原则

1. 先锁定 COA、日历、币种、会计方法和 Ledger，再配置 Ledger Set、Data Access Set、二级账簿及 Reporting Currency。
2. 任何子账余额差异先在子账/SLA 排除，确认已生成、传输和导入后再检查 GL；不要以手工总账分录掩盖子账差异。
3. 月结采用“子账关闭 → SLA/GL 传输 → Journal Import/Posting → 重估/折算/合并 → 报表与签字”的受控顺序。
4. 预算控制、自动平衡、悬挂账户、公司间与 Journal Approval 的例外须进入持续监控和审批流程。

<a id="src-docs-04-gl-readme--官方依据"></a>
### 官方依据

- [Oracle General Ledger Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
- [Oracle Financials Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48783/toc.htm)


<!-- source: docs/04-gl/budgetary-control.md -->
<a id="src-docs-04-gl-budgetary-control"></a>
## 预算、预算控制与资金可用性


<a id="src-docs-04-gl-budgetary-control--概念"></a>
### 概念

GL Budget 保存预算余额；Encumbrance 表示承诺/义务；Budgetary Control/Funds Check 按预算组织、账户范围、期间和边界判断可用资金。在不同实施中，可能使用传统 GL Encumbrance/Budgetary Control 或公共部门相关功能，须以已安装产品为准。

```text
Budget - Actual - Encumbrance = Funds Available
```

<a id="src-docs-04-gl-budgetary-control--配置"></a>
### 配置

1. 定义 Budget、Budget Organization、Account Ranges、Calendar/Periods。
2. 定义 Encumbrance Types 和采购阶段（Requisition/PO/Invoice）会计。
3. 定义 Funds Check Level（None/Advisory/Absolute）、Tolerance、Boundary 和 Override Authority。
4. 加载/过账预算，测试预算转移、补充、采购取消/反冲和期末结转。

<a id="src-docs-04-gl-budgetary-control--sql"></a>
### SQL

```sql
SELECT gb.budget_name, gb.status, gb.first_valid_period_name,
       gb.last_valid_period_name
  FROM gl_budgets gb
 WHERE gb.budget_name = :p_budget_name;

SELECT gb.ledger_id, gb.code_combination_id, gb.currency_code,
       gb.period_name, gb.actual_flag, gb.encumbrance_type_id,
       gb.period_net_dr, gb.period_net_cr,
       gb.begin_balance_dr, gb.begin_balance_cr
  FROM gl_balances gb
 WHERE gb.ledger_id = :p_ledger_id
   AND gb.period_name = :p_period_name
   AND gb.code_combination_id = :p_ccid
   AND gb.actual_flag IN ('A','B','E');
```

<a id="src-docs-04-gl-budgetary-control--排查"></a>
### 排查

- Funds Check 失败：检查预算组织账户范围、Budget Period/Amount、Actual/Encumbrance、Boundary、Currency 和控制级别。
- 可用金额不对：区分 Requisition/PO/Invoice 保留、未过账 Journal、取消/退货释放和期间跨度。
- Override 不可用：查用户限额/权限、Funds Check 级别、单据状态和审批链。
- 预算导入错：检查 Budget Name、Ledger/CCID、Currency、Period、Debit/Credit 方向和接口错误代码。

<a id="src-docs-04-gl-budgetary-control--关联"></a>
### 关联

- [COA](01-foundation.md#src-docs-01-common-coa)
- [P2P](09-end-to-end.md#src-docs-08-e2e-procure-to-pay)


<!-- source: docs/04-gl/close-reports.md -->
<a id="src-docs-04-gl-close-reports"></a>
## GL 期间开关、月结、年结与报表


<a id="src-docs-04-gl-close-reports--关账依赖"></a>
### 关账依赖

```text
Operational Freeze
→ AP/AR/CE/FA/INV/WIP/Cost/Projects Close & Reconcile
→ SLA Final + Transfer → GL Import/Post
→ Intercompany/Allocation/Revaluation/Translation
→ Trial Balance & Financial Statements
→ GL Close → Year-end Carry Forward
```

<a id="src-docs-04-gl-close-reports--月结清单"></a>
### 月结清单

1. 发布结账日历和截止时间，确认本期/下期输入规则。
2. 所有子账完成业务处理、库存/接口、会计、转 GL 和对账。
3. 处理 GL Interface、Suspense、未审批/未过账 Journal、Intercompany 不平。
4. 运行 Allocation/Accrual/Revaluation/Translation/Elimination，过账并复核。
5. 运行 Trial Balance、Account Analysis、FSG/BI 财务报表，保留参数和请求 ID。
6. 关闭 GL 期间。需重开时走授权与审计流程。

<a id="src-docs-04-gl-close-reports--年结"></a>
### 年结

- 确认 Natural Account Type，收入/费用结转 Retained Earnings，资产/负债/权益结转期初。
- 完成最终重估/折算、法人税务调整、抵销与审计分录。
- 开放新年期间前验证 Calendar 和期间，保存年末 Trial Balance/财务报表快照。

<a id="src-docs-04-gl-close-reports--sql"></a>
### SQL

```sql
SELECT fa.application_short_name, gps.period_name,
       gps.closing_status, gps.start_date, gps.end_date
  FROM gl_period_statuses gps
  JOIN fnd_application fa ON fa.application_id = gps.application_id
 WHERE gps.set_of_books_id = :p_ledger_id
   AND gps.period_name = :p_period_name
 ORDER BY fa.application_short_name;

SELECT gjh.je_source, gjh.je_category, gjh.status,
       COUNT(*) journal_count
  FROM gl_je_headers gjh
 WHERE gjh.ledger_id = :p_ledger_id
   AND gjh.period_name = :p_period_name
 GROUP BY gjh.je_source, gjh.je_category, gjh.status
 ORDER BY gjh.je_source, gjh.je_category, gjh.status;

SELECT status, user_je_source_name, COUNT(*) row_count
  FROM gl_interface
 WHERE ledger_id = :p_ledger_id
 GROUP BY status, user_je_source_name;
```

<a id="src-docs-04-gl-close-reports--排查"></a>
### 排查

- 无法关期：读取关期页面/报表指出的未完成项，定位子账、SLA、Interface 或 Journal 断点。
- Trial Balance 不平：分析 Ledger/Currency/Actual Flag/Translated Flag，检查 Suspense、Intercompany、异常 Journal 和数据完整性。
- 报表数字变动：比较两次请求之间的过账/反冲/重开期间记录，固定报表参数和数据截止时间。
- 期间误关：不更新 Period Status；评估报表/披露影响后使用标准 Reopen。

<a id="src-docs-04-gl-close-reports--关联"></a>
### 关联

- [Calendar/Period](01-foundation.md#src-docs-01-common-calendar-currency-period)
- [AP Close](03-procure-to-pay.md#src-docs-02-ap-accounting-close-reports)
- [AR Close](04-credit-to-cash.md#src-docs-03-ar-accounting-close-reports)


<!-- source: docs/04-gl/consolidation-revaluation.md -->
<a id="src-docs-04-gl-consolidation-revaluation"></a>
## 合并、重估、折算与重复日记账


<a id="src-docs-04-gl-consolidation-revaluation--原理"></a>
### 原理

- **Revaluation**：将外币账户余额按期末汇率重新计量，差额进入未实现损益。
- **Translation**：将 Ledger 余额从本位币折算为报告币种，资产负债/损益通常使用不同汇率规则。
- **Consolidation**：通过账户映射、数据转移和抵销将多账簿汇总；也可根据架构使用 Ledger Set/Secondary Ledger/Reporting 工具。
- **Recurring Journal/MassAllocation**：定期生成固定、公式或分配分录，生成后仍需审批与过账。

<a id="src-docs-04-gl-consolidation-revaluation--配置与执行"></a>
### 配置与执行

1. 定义 Period/Balance 汇率，定义 Revaluation 账户范围、Rate Type、Unrealized Gain/Loss。
2. 为 Translation 配置 Cumulative Translation Adjustment 账户和历史汇率。
3. 为 Consolidation 定义 COA Mapping、币种处理、子/母账簿期间映射、抵销与少数股东。
4. 对 Recurring/Allocation 使用独立 Source/Category，审查公式、统计量、分配基础和反冲。

<a id="src-docs-04-gl-consolidation-revaluation--sql"></a>
### SQL

```sql
-- 某期外币余额
SELECT ledger_id, code_combination_id, currency_code,
       period_name, actual_flag,
       begin_balance_dr, begin_balance_cr,
       period_net_dr, period_net_cr,
       begin_balance_dr_beq, begin_balance_cr_beq,
       period_net_dr_beq, period_net_cr_beq
  FROM gl_balances
 WHERE ledger_id = :p_ledger_id
   AND period_name = :p_period_name
   AND currency_code <> :p_ledger_currency
   AND actual_flag = 'A';

-- 期末汇率
SELECT from_currency, to_currency, conversion_date,
       conversion_type, conversion_rate, status_code
  FROM gl_daily_rates
 WHERE conversion_date = :p_rate_date
   AND to_currency = :p_ledger_currency
   AND conversion_type = :p_rate_type
 ORDER BY from_currency;
```

> 日汇率只是重估输入之一；实际执行还受 Revaluation Definition 的账户范围、币种、汇率类型和损益账户影响。生产重估以标准程序日志和报表输出为准。

<a id="src-docs-04-gl-consolidation-revaluation--排查"></a>
### 排查

- Revaluation 遗漏账户：查账户范围、Currency、Balance、Period Rate 和余额是否已为零。
- Translation 不平：查 Historical/Average/Period-end Rate、CTA Account、期间顺序和当期日记账是否全部过账。
- Consolidation 差异：查 COA Mapping、期间/币种、增量/全量转移、重复转移和抵销分录。
- Allocation 异常：保存生成批次，输出分配基础与公式中间值，不直接修改已过账行。

<a id="src-docs-04-gl-consolidation-revaluation--关联"></a>
### 关联

- [Currency/Rate](01-foundation.md#src-docs-01-common-calendar-currency-period)
- [GL 结账](#src-docs-04-gl-close-reports)


<!-- source: docs/04-gl/interfaces.md -->
<a id="src-docs-04-gl-interfaces"></a>
## Oracle General Ledger 接口实现案例


<a id="src-docs-04-gl-interfaces--1-业界常用场景"></a>
### 1. 业界常用场景

| 场景 | 推荐接口 | 实施要点 |
| --- | --- | --- |
| 薪资、资金、费用系统生成总账凭证 | `GL_INTERFACE` + Journal Import | 每批使用独立 `GROUP_ID`，源系统单号写入 `REFERENCE*` |
| 多 ERP/海外系统汇总凭证 | 汇总层暂存表 + `GL_INTERFACE` | 同时传批次控制总额、币种和账簿，导入前验证借贷平衡 |
| 大批量人工调整 | Web ADI | 使用受控 Layout、List of Values 和职责权限 |
| 子账会计传总账 | SLA Transfer to GL | AP/AR/FA 等子账不应绕过 SLA 直接拼装 GL 分录 |
| 外部系统实时记账 | ISG 暴露受控并发程序或自定义服务 | 接口服务只入暂存/接口表，后台异步 Journal Import |

<a id="src-docs-04-gl-interfaces--2-导入前主数据与期间校验"></a>
### 2. 导入前主数据与期间校验

```sql
SELECT gl.ledger_id,
       gl.name ledger_name,
       gl.currency_code,
       gps.period_name,
       gps.closing_status
  FROM gl_ledgers gl
  JOIN gl_period_statuses gps
    ON gps.set_of_books_id = gl.ledger_id
   AND gps.application_id = 101             -- General Ledger
 WHERE gl.ledger_id = :p_ledger_id
   AND gps.period_name = :p_period_name;

SELECT gcc.code_combination_id,
       gcc.enabled_flag,
       gcc.detail_posting_allowed_flag,
       gcc.start_date_active,
       gcc.end_date_active
  FROM gl_code_combinations gcc
 WHERE gcc.code_combination_id = :p_ccid
   AND gcc.chart_of_accounts_id = :p_coa_id;
```

`CLOSING_STATUS='O'` 通常表示 Open；实际允许状态还应结合 Open Future Enterable Periods、Data Access Set 和账户有效期判断。

<a id="src-docs-04-gl-interfaces--3-glinterface-具体实现"></a>
### 3. `GL_INTERFACE` 具体实现

<a id="src-docs-04-gl-interfaces--31-生成批次号并写入借贷行"></a>
#### 3.1 生成批次号并写入借贷行

```sql
DECLARE
  l_group_id NUMBER := gl_interface_control_s.NEXTVAL;
  l_user_id  NUMBER := fnd_global.user_id;
BEGIN
  -- 借方行
  INSERT INTO gl_interface (
    status,
    ledger_id,
    accounting_date,
    currency_code,
    date_created,
    created_by,
    actual_flag,
    user_je_source_name,
    user_je_category_name,
    group_id,
    code_combination_id,
    entered_dr,
    entered_cr,
    reference1,
    reference4,
    reference10
  ) VALUES (
    'NEW',
    :p_ledger_id,
    :p_accounting_date,
    :p_currency_code,
    SYSDATE,
    l_user_id,
    'A',
    'XX EXTERNAL',
    'Adjustment',
    l_group_id,
    :p_debit_ccid,
    :p_amount,
    NULL,
    :p_external_batch_id,
    :p_external_document_id,
    'External integration debit'
  );

  -- 贷方行
  INSERT INTO gl_interface (
    status,
    ledger_id,
    accounting_date,
    currency_code,
    date_created,
    created_by,
    actual_flag,
    user_je_source_name,
    user_je_category_name,
    group_id,
    code_combination_id,
    entered_dr,
    entered_cr,
    reference1,
    reference4,
    reference10
  ) VALUES (
    'NEW',
    :p_ledger_id,
    :p_accounting_date,
    :p_currency_code,
    SYSDATE,
    l_user_id,
    'A',
    'XX EXTERNAL',
    'Adjustment',
    l_group_id,
    :p_credit_ccid,
    NULL,
    :p_amount,
    :p_external_batch_id,
    :p_external_document_id,
    'External integration credit'
  );

  COMMIT;
  dbms_output.put_line('GROUP_ID=' || l_group_id);
END;
/
```

生产实现应先在自定义暂存表保存原始消息和幂等键，再由单一工作进程写 `GL_INTERFACE`。不要直接写 `GL_JE_HEADERS`、`GL_JE_LINES` 或 `GL_BALANCES`。

<a id="src-docs-04-gl-interfaces--32-外币分录"></a>
#### 3.2 外币分录

外币日记账必须按实例规则提供 `CURRENCY_CONVERSION_TYPE`、`CURRENCY_CONVERSION_DATE` 和汇率，或者确保 GL Daily Rates 能派生汇率：

```sql
UPDATE gl_interface
   SET currency_conversion_type = :p_conversion_type,
       currency_conversion_date = :p_conversion_date,
       currency_conversion_rate = :p_conversion_rate
 WHERE group_id = :p_group_id
   AND currency_code <> :p_ledger_currency
   AND status = 'NEW';
```

该更新只能作为同一接口工作单元在 Journal Import 前执行，不应在 Import 已运行后修补数据。

<a id="src-docs-04-gl-interfaces--4-批次控制与幂等校验"></a>
### 4. 批次控制与幂等校验

```sql
-- 每个 Ledger、Currency、Group 必须借贷平衡
SELECT ledger_id,
       currency_code,
       group_id,
       SUM(NVL(entered_dr, 0)) total_dr,
       SUM(NVL(entered_cr, 0)) total_cr,
       SUM(NVL(entered_dr, 0) - NVL(entered_cr, 0)) difference
  FROM gl_interface
 WHERE group_id = :p_group_id
 GROUP BY ledger_id, currency_code, group_id
HAVING ABS(SUM(NVL(entered_dr, 0) - NVL(entered_cr, 0))) > 0.00001;

-- 提交前防止同一外部单据再次入接口
SELECT COUNT(*) duplicate_count
  FROM gl_interface
 WHERE user_je_source_name = 'XX EXTERNAL'
   AND reference1 = :p_external_batch_id
   AND reference4 = :p_external_document_id;
```

只查 `GL_INTERFACE` 不能覆盖已导入数据。可靠幂等应以自定义消息表唯一约束为主，并在成功表中保存 `JE_BATCH_ID/JE_HEADER_ID`。

<a id="src-docs-04-gl-interfaces--5-提交-journal-import"></a>
### 5. 提交 Journal Import

先在目标实例确认并发程序 `GLLEZL` 的参数顺序；补丁、Ledger/Data Access Set 设置可能使参数定义不同。

```sql
SELECT cp.concurrent_program_name,
       dfa.column_seq_num,
       dfa.end_user_column_name,
       dfa.required_flag
  FROM fnd_concurrent_programs cp
  JOIN fnd_descr_flex_column_usages dfa
    ON dfa.application_id = cp.application_id
   AND dfa.descriptive_flexfield_name = '$SRS$.' || cp.concurrent_program_name
 WHERE cp.concurrent_program_name = 'GLLEZL'
   AND dfa.enabled_flag = 'Y'
 ORDER BY dfa.column_seq_num;
```

确认参数后，可封装为受控程序提交：

```sql
DECLARE
  l_request_id NUMBER;
BEGIN
  fnd_global.apps_initialize(:p_user_id, :p_resp_id, :p_resp_appl_id);

  l_request_id := fnd_request.submit_request(
    application => 'SQLGL',
    program     => 'GLLEZL',
    description => NULL,
    start_time  => NULL,
    sub_request => FALSE,
    argument1   => TO_CHAR(:p_interface_run_id),
    argument2   => TO_CHAR(:p_access_set_id),
    argument3   => 'XX EXTERNAL',
    argument4   => TO_CHAR(:p_ledger_id),
    argument5   => TO_CHAR(:p_group_id),
    argument6   => 'N',
    argument7   => 'N'
  );

  IF l_request_id = 0 THEN
    raise_application_error(-20040, fnd_message.get);
  END IF;
  COMMIT;
  dbms_output.put_line('REQUEST_ID=' || l_request_id);
END;
/
```

上例是常见参数骨架，不是可跳过目标实例核对的固定签名。若实例要求 `GL_INTERFACE_CONTROL`，应通过标准 Journal Import 提交流程创建 Interface Run，而不是猜测参数或自行更新控制状态。

<a id="src-docs-04-gl-interfaces--6-错误排查与成功对账"></a>
### 6. 错误排查与成功对账

```sql
-- 接口状态及错误代码分布
SELECT status, COUNT(*) line_count
  FROM gl_interface
 WHERE group_id = :p_group_id
 GROUP BY status
 ORDER BY status;

-- 成功导入后按来源和外部批次定位 Journal
SELECT gjb.je_batch_id,
       gjb.name batch_name,
       gjh.je_header_id,
       gjh.name journal_name,
       gjh.status,
       gjh.period_name,
       SUM(NVL(gjl.entered_dr, 0)) entered_dr,
       SUM(NVL(gjl.entered_cr, 0)) entered_cr
  FROM gl_je_batches gjb
  JOIN gl_je_headers gjh ON gjh.je_batch_id = gjb.je_batch_id
  JOIN gl_je_lines gjl ON gjl.je_header_id = gjh.je_header_id
 WHERE gjh.je_source = 'XX EXTERNAL'
   AND EXISTS (
         SELECT 1
           FROM gl_import_references gir
          WHERE gir.je_header_id = gjl.je_header_id
            AND gir.je_line_num = gjl.je_line_num
            AND gir.reference_1 = :p_external_batch_id
       )
 GROUP BY gjb.je_batch_id, gjb.name, gjh.je_header_id,
          gjh.name, gjh.status, gjh.period_name;
```

Journal Import Execution Report 是错误代码的首要解释来源。常见问题包括期间未开、Source/Category 未定义、CCID 无效、外币汇率缺失、借贷不平和 Data Access Set 无权限。

<a id="src-docs-04-gl-interfaces--7-实施控制清单"></a>
### 7. 实施控制清单

- 为外部来源建立独立 Journal Source/Category，并明确是否允许冻结、审批和保留 Import Reference。
- 每个消息保存 `CORRELATION_ID`、源单号、`GROUP_ID`、Request ID、Journal ID 和批次控制总额。
- 重试只重试暂存层失败消息；不要复制仍在处理或结果未知的 `GL_INTERFACE` 行。
- 把“导入成功”“审批完成”“已过账”作为不同业务状态，分别对账。
- 日记账成功导入后仍需根据公司政策执行审批、过账和反冲。

<a id="src-docs-04-gl-interfaces--8-关联文档"></a>
### 8. 关联文档

- [GL 日记账、审批与过账](#src-docs-04-gl-journals)
- [GL 业务流程](#src-docs-04-gl-process)
- [公共 SLA](01-foundation.md#src-docs-01-common-sla)
- [GL 常用表](#src-docs-04-gl-tables)
- [通用接口治理](01-foundation.md#src-docs-01-common-interfaces)

<a id="src-docs-04-gl-interfaces--9-官方参考"></a>
### 9. 官方参考

- [Oracle General Ledger Implementation Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48747/)
- [Oracle E-Business Suite Integrated SOA Gateway Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e20925/)
- [Oracle E-Business Suite eTRM User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/)


<!-- source: docs/04-gl/journals.md -->
<a id="src-docs-04-gl-journals"></a>
## GL 日记账来源、类别、审批与自动过账


<a id="src-docs-04-gl-journals--原理与控制"></a>
### 原理与控制

Journal Source 标识产生系统，控制 Import、Freeze Source、Journal References 等；Category 标识业务性质。Batch 是审批/过账单位，Header 定义 Ledger/Period/Currency/Source/Category，Line 保存账户和借贷。

- Journal Approval 通常依据 Ledger、Source/Category、Amount 和职权配置 Workflow/AME。
- AutoPost Criteria Set 按 Ledger、Source、Category、Balance Type、Period 筛选批次。
- Reversal 可按 Category 默认 Method/Period；Switch Dr/Cr 与 Change Sign 的结果不同。
- Source Freeze 可防止在 GL 修改来自子账的日记账，保持审计链。

<a id="src-docs-04-gl-journals--sql"></a>
### SQL

```sql
SELECT gjh.je_header_id, gjh.je_batch_id, gjh.name,
       gjh.je_source, gjh.je_category, gjh.status,
       gjh.period_name, gjh.currency_code,
       gjh.running_total_dr, gjh.running_total_cr,
       gjh.running_total_accounted_dr,
       gjh.running_total_accounted_cr,
       gjh.reversed_je_header_id, gjh.accrual_rev_period_name
  FROM gl_je_headers gjh
 WHERE gjh.je_header_id = :p_je_header_id;

SELECT gjb.je_batch_id, gjb.name, gjb.status,
       gjb.approval_status_code, gjb.posted_date,
       gjb.posting_run_id, gjb.request_id
  FROM gl_je_batches gjb
 WHERE gjb.je_batch_id = :p_je_batch_id;

SELECT je_source_name, user_je_source_name,
       journal_approval_flag, override_edits_flag
  FROM gl_je_sources
 ORDER BY user_je_source_name;
```

<a id="src-docs-04-gl-journals--排查"></a>
### 排查

- 审批人找不到：查员工/用户关联、职位/审批限额、Workflow/AME 规则和通知状态。
- AutoPost 没选中：比较 Criteria Set 与 Batch 的 Ledger/Source/Category/Balance Type/Period/Status。
- Reversal 不正确：检查 Reversal Method、Period、Effective Date、原日记账状态和是否已反冲。
- 子账 Journal 被修改：检查 Source Freeze、职责和审计数据；应在子账反冲并重建会计。

<a id="src-docs-04-gl-journals--关联"></a>
### 关联

- [GL 主流程](#src-docs-04-gl-process)
- [Web ADI/Import](#src-docs-04-gl-reporting-interfaces)


<!-- source: docs/04-gl/process.md -->
<a id="src-docs-04-gl-process"></a>
## General Ledger 账簿、日记账与过账流程


<a id="src-docs-04-gl-process--架构"></a>
### 架构

```text
Subledger/SLA → GL_INTERFACE → Journal Import
Manual/Web ADI/Recurring → Journal Batch/Header/Lines
→ Approval → Post → GL_BALANCES → Reporting/Close
```

Ledger 由 COA、Currency、Calendar 和 SLA Method 组成。Primary/Secondary Ledger 表示不同会计表述，Reporting Currency 表示币种表述，Ledger Set 用于对多账簿统一开关期和报表。Data Access Set 决定职责对 Ledger/平衡段的读写权限。

<a id="src-docs-04-gl-process--配置"></a>
### 配置

1. 定义 COA、Calendar、Currency/Rate，在 Accounting Setup Manager 建立 Ledger。
2. 配置 Legal Entity/Balancing Segment、Secondary Ledger/Reporting Currency、SLA、Intercompany/Intracompany。
3. 定义 Journal Source/Category、Suspense/Rounding/Retained Earnings、Document Sequence、Approval/AutoPost。
4. 定义 Data Access Set、Ledger Set、账户安全与 FSG/BI 报表。

<a id="src-docs-04-gl-process--sql"></a>
### SQL

```sql
SELECT gjb.je_batch_id, gjb.name batch_name, gjb.status batch_status,
       gjh.je_header_id, gjh.name journal_name, gjh.status,
       gjh.ledger_id, gjh.period_name, gjh.je_source,
       gjh.je_category, gjh.currency_code, gjh.actual_flag
  FROM gl_je_batches gjb
  JOIN gl_je_headers gjh ON gjh.je_batch_id = gjb.je_batch_id
 WHERE gjh.je_header_id = :p_je_header_id;

SELECT gjl.je_line_num, gjl.code_combination_id,
       gjl.entered_dr, gjl.entered_cr,
       gjl.accounted_dr, gjl.accounted_cr,
       gjl.description, gjl.status
  FROM gl_je_lines gjl
 WHERE gjl.je_header_id = :p_je_header_id
 ORDER BY gjl.je_line_num;
```

<a id="src-docs-04-gl-process--排查"></a>
### 排查

- Import 失败：查 `GL_INTERFACE.STATUS/STATUS_DESCRIPTION`、Ledger/Period/Currency/CCID、Source/Group ID。
- Journal 不平：分别检查 Entered/Accounted 借贷、Currency/Rate、Suspense/Rounding 设置和平衡段。
- 不能 Post：检查 Batch/Header Status、Approval、Period、Data Access Set Write 权限、账户有效性。
- 余额不更新：查 Posting 请求日志、Journal Status、Actual Flag、Currency 和查询的 Balance Type。

<a id="src-docs-04-gl-process--关联"></a>
### 关联

- [GL 常用表结构与字段含义](#src-docs-04-gl-tables)
- [Journal 控制](#src-docs-04-gl-journals)
- [SLA](01-foundation.md#src-docs-01-common-sla)
- [GL 结账](#src-docs-04-gl-close-reports)


<!-- source: docs/04-gl/reporting-interfaces.md -->
<a id="src-docs-04-gl-reporting-interfaces"></a>
## FSG、Smart View、Web ADI 与日记账导入


> `GL_INTERFACE`、批次平衡、Journal Import 提交和结果对账代码见 [GL 接口实现案例](#src-docs-04-gl-interfaces)。

<a id="src-docs-04-gl-reporting-interfaces--报表与接口"></a>
### 报表与接口

- **FSG**：Row Set 定义账户/计算行，Column Set 定义期间/金额/计算列，Content Set 按段拆分，Row Order 定义排序，Display Set 控制显示。
- **Smart View**：通过已配置数据源在 Excel 查询/钻取 GL 余额，权限仍受 EBS/GL 数据访问控制。
- **Web ADI**：Integrator + Interface + Content + Layout + Mapping 将 Excel 数据验证并上传，GL Journal 最终进入 GL Interface/Import。
- **Journal Import**：按 Source/Group ID 从 `GL_INTERFACE` 生成 Batch/Header/Lines，错误行留在接口表并带 Status。

<a id="src-docs-04-gl-reporting-interfaces--sql"></a>
### SQL

```sql
SELECT status, ledger_id, user_je_source_name,
       user_je_category_name, accounting_date,
       currency_code, code_combination_id,
       entered_dr, entered_cr, accounted_dr, accounted_cr,
       group_id, reference1, reference4
  FROM gl_interface
 WHERE group_id = :p_group_id
 ORDER BY accounting_date;

SELECT gir.je_header_id, gir.je_line_num,
       gir.reference_1, gir.reference_2, gir.reference_3,
       gir.reference_4, gir.gl_sl_link_id, gir.gl_sl_link_table
  FROM gl_import_references gir
 WHERE gir.je_header_id = :p_je_header_id
 ORDER BY gir.je_line_num;

SELECT row_set_id, name, description
  FROM rg_report_axis_sets
 ORDER BY name;
```

<a id="src-docs-04-gl-reporting-interfaces--排查"></a>
### 排查

- FSG 金额不对：检查 Ledger/Currency/Amount Type、Period Offset、Row Account Assignment、Summary/Detail、Sign 和报表显示舍入。
- FSG 空白：查 Data Access Set、行列显示条件、Zero Suppression、Content Set 和账户范围。
- Web ADI 上传失败：查 Desktop Integration 配置、Excel 信任/宏、Integrator/Layout/Mapping、职责、日期格式和服务器日志。
- Journal Import 错：按 `STATUS` 解码，检查 Ledger/Period/CCID/Currency/Balance/Source；修正上游或接口，不直接改已生成 Journal。
- 子账 Drilldown 丢失：检查 Journal Source Import References 选项和 `GL_IMPORT_REFERENCES`/SLA Link。

<a id="src-docs-04-gl-reporting-interfaces--关联"></a>
### 关联

- [GL Process](#src-docs-04-gl-process)
- [COA](01-foundation.md#src-docs-01-common-coa)
- [Integration](10-technical.md#src-docs-09-technical-integration)

<a id="src-docs-04-gl-reporting-interfaces--官方参考"></a>
### 官方参考

- [Oracle General Ledger Implementation Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48747/toc.htm)


<!-- source: docs/04-gl/sla-fah-agis.md -->
<a id="src-docs-04-gl-sla-fah-agis"></a>
## SLA、Financials Accounting Hub 与 AGIS


<a id="src-docs-04-gl-sla-fah-agis--适用范围"></a>
### 适用范围

SLA 是 EBS 子账会计的通用引擎；GL 接收其传输的日记账。Financials Accounting Hub（FAH）用于将外部业务系统的事件转换为受控的子账会计；Advanced Global Intercompany System（AGIS）处理跨法人/跨组织内部交易。两者均为独立可选产品/功能边界，须确认许可证、安装和实施范围。

<a id="src-docs-04-gl-sla-fah-agis--会计链路"></a>
### 会计链路

```text
业务交易
  → Transaction Entity / Event / Event Type
  → Accounting Method Builder（AAD/JLD/JLT/ADR/Mapping Set）
  → XLA AE Header / Line
  → Transfer to GL
  → Journal Import / GL Journal / Post
```

<a id="src-docs-04-gl-sla-fah-agis--设计要点"></a>
### 设计要点

| 主题 | 实施决定 | 控制要求 |
| --- | --- | --- |
| SLA | 会计方法、事件类型、账户规则、辅助参考、说明规则 | 不直接改已完成历史会计；规则改动须版本化、测试和审批 |
| FAH | 外部来源、事件模型、接口字段、映射、异常/重放 | 外部业务键必须唯一，可从来源交易追溯至 GL |
| AGIS | 交易类型、组织关系、内部交易账户、审批与接收规则 | 发出/接收、AP/AR、公司间与消除差异分别对账 |
| Balancing | Intercompany/Intracompany 规则、舍入、悬挂账户 | 配置变化先在隔离 Ledger/测试数据验证分录平衡 |

<a id="src-docs-04-gl-sla-fah-agis--只读诊断-sql"></a>
### 只读诊断 SQL

```sql
-- 从会计事件追踪已生成的子账分录；按事件或实体键收缩范围。
select xte.entity_code,
       xte.source_id_int_1,
       xte.transaction_number,
       xah.event_id,
       xah.ae_header_id,
       xah.accounting_entry_status_code,
       xah.gl_transfer_status_code
  from xla_transaction_entities xte
  join xla_events xev
    on xev.entity_id = xte.entity_id
  join xla_ae_headers xah
    on xah.event_id = xev.event_id
 where xte.ledger_id = :p_ledger_id
   and xte.transaction_number = :p_transaction_number;

-- 分录行到 GL 的关联应通过受支持的 XLA/GL 链审查，字段以目标 eTRM 为准。
select xal.ae_header_id,
       xal.ae_line_num,
       xal.accounting_class_code,
       xal.accounted_dr,
       xal.accounted_cr,
       xal.gl_sl_link_id
  from xla_ae_lines xal
 where xal.ae_header_id = :p_ae_header_id
 order by xal.ae_line_num;
```

<a id="src-docs-04-gl-sla-fah-agis--排错顺序"></a>
### 排错顺序

1. 确认源交易、会计事件及其状态，再检查会计定义是否对该事件类型生效。
2. 区分“未创建会计”“Draft/Final 状态”“未传输 GL”“Journal Import/过账失败”四个断点。
3. 对 FAH/AGIS 先检查来源业务键、批次控制和接收方状态；不要把跨系统部分成功当作单一数据库事务回滚。

<a id="src-docs-04-gl-sla-fah-agis--官方参考"></a>
### 官方参考

- [Oracle Subledger Accounting Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48771/title.htm)
- [Oracle Financials Accounting Hub Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
- [Oracle Advanced Global Intercompany System Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/04-gl/tables.md -->
<a id="src-docs-04-gl-tables"></a>
## Oracle General Ledger 常用表结构


<a id="src-docs-04-gl-tables--业务说明"></a>
### 业务说明

GL 的业务层级是 Ledger → Batch → Journal Header → Journal Line → Balance。子账数据经 SLA 进入 `GL_INTERFACE`，Journal Import 生成日记账，Posting 更新 `GL_BALANCES`。日记账行是交易流量，Balance 是 Ledger+CCID+Currency+Period+Balance Type 的累计/期间快照。

<a id="src-docs-04-gl-tables--表级速查"></a>
### 表级速查

| 表 | 中文名 | 业务粒度 | 关键字段 |
| --- | --- | --- | --- |
| `GL_LEDGERS` | 账簿 | 每个 Ledger | `LEDGER_ID`, `CHART_OF_ACCOUNTS_ID` |
| `GL_LEDGER_SETS` | 账簿集 | 每个 Ledger Set | `LEDGER_SET_ID`, `NAME` |
| `GL_ACCESS_SETS` | 数据访问集 | 每个 Data Access Set | `ACCESS_SET_ID`, `SECURITY_SEGMENT_CODE` |
| `GL_CODE_COMBINATIONS` | 会计科目组合 | 每个 CCID | `CODE_COMBINATION_ID`, `SEGMENT1..N` |
| `GL_JE_BATCHES` | 日记账批 | 每个 Batch | `JE_BATCH_ID`, `STATUS`, `APPROVAL_STATUS_CODE` |
| `GL_JE_HEADERS` | 日记账头 | 每个 Journal | `JE_HEADER_ID`, `LEDGER_ID`, `PERIOD_NAME`, `STATUS` |
| `GL_JE_LINES` | 日记账行 | Journal+行号 | `JE_HEADER_ID`, `JE_LINE_NUM`, `CODE_COMBINATION_ID` |
| `GL_INTERFACE` | GL 日记账接口 | 待 Import 分录行 | `STATUS`, `LEDGER_ID`, `GROUP_ID`, `USER_JE_SOURCE_NAME` |
| `GL_IMPORT_REFERENCES` | GL 导入参考 | GL Journal Line 与子账链接 | `JE_HEADER_ID`, `JE_LINE_NUM`, `GL_SL_LINK_ID/TABLE` |
| `GL_BALANCES` | GL 科目余额 | Ledger+CCID+Currency+Period+Flag | `LEDGER_ID`, `CODE_COMBINATION_ID`, `PERIOD_NAME`, `ACTUAL_FLAG` |
| `GL_PERIOD_STATUSES` | GL/子账期间状态 | Application+Ledger+Period | `APPLICATION_ID`, `CLOSING_STATUS` |
| `GL_DAILY_RATES` | 日汇率 | 币种对+日期+类型 | `CONVERSION_DATE`, `CONVERSION_TYPE`, `CONVERSION_RATE` |

<a id="src-docs-04-gl-tables--gljebatches-日记账批"></a>
### `GL_JE_BATCHES` — 日记账批

| 字段 | 中文名 | 业务含义 |
| --- | --- | --- |
| `JE_BATCH_ID` | 日记账批 ID | Header 的外键，审批/过账常以 Batch 为单位 |
| `NAME` | 批名称 | 可包含 Source/Period/系统生成信息，不应作为稳定唯一集成键 |
| `STATUS` | 批状态 | 与 Header Status 共同判断是否可审批/过账 |
| `APPROVAL_STATUS_CODE` | 审批状态 | Required/In Process/Approved/Rejected 等，请用 GL Lookup 解码 |
| `POSTED_DATE` | 过账日期 | 已过账批的实际过账时间 |
| `POSTING_RUN_ID` | 过账运行 ID | 跟踪 Posting 程序批次 |

<a id="src-docs-04-gl-tables--gljeheaders-日记账头"></a>
### `GL_JE_HEADERS` — 日记账头

| 字段 | 中文名 | 业务含义/值 |
| --- | --- | --- |
| `LEDGER_ID` | 账簿 ID | 决定 COA、日历、本位币和 Data Access Set |
| `JE_SOURCE` | 日记账来源 | Payables/Receivables/Assets/Manual 等，应与 `GL_JE_SOURCES` 解码 |
| `JE_CATEGORY` | 日记账类别 | Invoices/Payments/Receipts/Adjustment 等业务性质 |
| `STATUS` | Journal 状态 | `U` 常表示 Unposted，`P` 常表示 Posted；其他值可为错误/导入状态，必须通过 GL 标准解码 |
| `ACTUAL_FLAG` | 余额类型 | `A`实际、`B`预算、`E`保留/承诺 |
| `CURRENCY_CODE` | Journal 币种 | 交易币；`ACCOUNTED_*` 为 Ledger Currency |
| `PERIOD_NAME` | 会计期间 | 需与 `DEFAULT_EFFECTIVE_DATE` 及 Ledger Calendar 一致 |
| `CONVERSION_TYPE/DATE/RATE` | 汇率属性 | 非本位币 Journal 的折算依据 |
| `REVERSED_JE_HEADER_ID` | 被反冲 Journal | 用于反冲链跟踪，同时查 Reversal Period/Method |

<a id="src-docs-04-gl-tables--gljelines-日记账行"></a>
### `GL_JE_LINES` — 日记账行

| 字段 | 中文名 | 说明 |
| --- | --- | --- |
| `ENTERED_DR/ENTERED_CR` | 交易币借/贷 | 同一行通常仅一侧有值 |
| `ACCOUNTED_DR/ACCOUNTED_CR` | 本位币借/贷 | 外币 Journal 经汇率折算后金额 |
| `CODE_COMBINATION_ID` | 会计科目 CCID | 必须属于 Ledger COA，在有效日可过账 |
| `EFFECTIVE_DATE` | 有效/过账日期 | 决定 Period，不等于 Creation Date |
| `REFERENCE_1..10` | 导入参考 | 含义由 Source/Interface 决定，不应跨 Source 固定解读 |

<a id="src-docs-04-gl-tables--glbalances-余额"></a>
### `GL_BALANCES` — 余额

| 字段 | 中文名 | 说明 |
| --- | --- | --- |
| `ACTUAL_FLAG` | 实际/预算/保留 | `A/B/E` |
| `CURRENCY_CODE` | 余额币种 | Ledger Currency、Foreign Currency、Statistical Currency 需按报表参数区分 |
| `TRANSLATED_FLAG` | 折算标志 | 用于外币/折算余额识别，NULL 不一定是错误 |
| `BEGIN_BALANCE_DR/CR` | 期初借/贷余额 | 净额通常用 Dr-Cr 计算，显示符号受账户类型影响 |
| `PERIOD_NET_DR/CR` | 本期借/贷发生 | 与期初共同计算期末 |
| `*_BEQ` | 本位币等值 | 外币余额的 Ledger Currency Equivalent |

<a id="src-docs-04-gl-tables--glinterface-常见状态原则"></a>
### `GL_INTERFACE` 常见状态原则

- `STATUS='NEW'` 通常表示等待 Journal Import。
- Import 失败后 `STATUS` 可变为具体错误代码，应用 Journal Import Execution Report/GL Lookup 解码，不建立不完整的自制代码表。
- `GROUP_ID` 隔离一次导入批次；`REFERENCE*` 应保存可追溯源单据的值。
- 已成功 Import 的数据不再以 `GL_INTERFACE` 为完整审计源，应跟踪 Journal 和 `GL_IMPORT_REFERENCES`。

<a id="src-docs-04-gl-tables--官方参考"></a>
### 官方参考

- [Oracle General Ledger Implementation Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48747/)
- [Oracle E-Business Suite eTRM User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/)
