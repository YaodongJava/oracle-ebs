# 资产、项目与资本化

> 固定资产、项目成本/开票、项目转资产及相关扩展产品。本文件由原目录中的 22 份资料合并而成；各章节保留原来源标记，便于审计与后续去重。

## 本模块章节导航

- [Assets and Projects](#src-docs-05-assets-projects-readme)（原 `docs/05-assets-projects/README.md`）
- [Assets and Projects： asset-tracking-and-eam](#src-docs-05-assets-projects-asset-tracking-and-eam-readme)（原 `docs/05-assets-projects/asset-tracking-and-eam/README.md`）
- [Assets and Projects： fixed-assets](#src-docs-05-assets-projects-fixed-assets-readme)（原 `docs/05-assets-projects/fixed-assets/README.md`）
- [Assets and Projects： grants-accounting](#src-docs-05-assets-projects-grants-accounting-readme)（原 `docs/05-assets-projects/grants-accounting/README.md`）
- [Assets and Projects： iassets](#src-docs-05-assets-projects-iassets-readme)（原 `docs/05-assets-projects/iassets/README.md`）
- [Assets and Projects： lease-and-finance-management](#src-docs-05-assets-projects-lease-and-finance-management-readme)（原 `docs/05-assets-projects/lease-and-finance-management/README.md`）
- [Assets and Projects： project-billing](#src-docs-05-assets-projects-project-billing-readme)（原 `docs/05-assets-projects/project-billing/README.md`）
- [Assets and Projects： project-contracts](#src-docs-05-assets-projects-project-contracts-readme)（原 `docs/05-assets-projects/project-contracts/README.md`）
- [Assets and Projects： project-costing](#src-docs-05-assets-projects-project-costing-readme)（原 `docs/05-assets-projects/project-costing/README.md`）
- [Assets and Projects： project-planning-control](#src-docs-05-assets-projects-project-planning-control-readme)（原 `docs/05-assets-projects/project-planning-control/README.md`）
- [Assets and Projects： project-to-asset](#src-docs-05-assets-projects-project-to-asset-readme)（原 `docs/05-assets-projects/project-to-asset/README.md`）
- [Assets and Projects： projects-foundation](#src-docs-05-assets-projects-projects-foundation-readme)（原 `docs/05-assets-projects/projects-foundation/README.md`）
- [Assets and Projects： property-manager](#src-docs-05-assets-projects-property-manager-readme)（原 `docs/05-assets-projects/property-manager/README.md`）
- [Oracle Assets（FA / Acquire to Retire）](#src-docs-05-fa-readme)（原 `docs/05-fa/README.md`）
- [FA 资产增加、调整、转移、重分类与盘点](#src-docs-05-fa-asset-transactions)（原 `docs/05-fa/asset-transactions.md`）
- [FA 月结、报表、Mass Additions 与排错](#src-docs-05-fa-close-reports-interfaces)（原 `docs/05-fa/close-reports-interfaces.md`）
- [FA 折旧、税务折旧、资产处置与会计](#src-docs-05-fa-depreciation-accounting)（原 `docs/05-fa/depreciation-accounting.md`）
- [Oracle Assets 接口实现案例](#src-docs-05-fa-interfaces)（原 `docs/05-fa/interfaces.md`）
- [Oracle Assets 资产全生命周期](#src-docs-05-fa-process)（原 `docs/05-fa/process.md`）
- [Projects 到 Assets：CIP 与资本化](#src-docs-05-fa-projects-capitalization)（原 `docs/05-fa/projects-capitalization.md`）
- [FA 资产账簿、类别、位置与关键配置](#src-docs-05-fa-setup)（原 `docs/05-fa/setup.md`）
- [Oracle Assets 常用表结构](#src-docs-05-fa-tables)（原 `docs/05-fa/tables.md`）

---

<!-- source: docs/05-assets-projects/README.md -->
<a id="src-docs-05-assets-projects-readme"></a>
## Assets and Projects


<a id="src-docs-05-assets-projects-readme--范围与目标"></a>
### 范围与目标
覆盖固定资产、iAssets、eAM、Projects、项目成本/开票/计划/合同/Grants、项目转资产、物业与租赁。

<a id="src-docs-05-assets-projects-readme--运行与实施控制"></a>
### 运行与实施控制
明确可资本化政策、项目/任务/资产类别/账簿、成本归集、资产行、Mass Additions、折旧和处置；项目成本、CIP、FA 和 GL 分别对账。

<a id="src-docs-05-assets-projects-readme--核心数据对象"></a>
### 核心数据对象
FA_ADDITIONS_B、FA_BOOKS、FA_DISTRIBUTION_HISTORY、FA_DEPRN_SUMMARY、FA_MASS_ADDITIONS、PA_PROJECTS_ALL、PA_TASKS、PA_EXPENDITURE_ITEMS_ALL。对象、列、状态和 API 签名须在目标实例 eTRM、Integration Repository 与数据字典复核。

<a id="src-docs-05-assets-projects-readme--与既有知识的关系"></a>
### 与既有知识的关系
本目标目录新增详细入口；已有专题保留在 [05-fa/README](#src-docs-05-fa-readme) 并逐步迁移链接，不复制历史内容。

<a id="src-docs-05-assets-projects-readme--官方依据"></a>
### 官方依据
[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/05-assets-projects/asset-tracking-and-eam/README.md -->
<a id="src-docs-05-assets-projects-asset-tracking-and-eam-readme"></a>
## Assets and Projects： asset-tracking-and-eam


<a id="src-docs-05-assets-projects-asset-tracking-and-eam-readme--业务定位"></a>
### 业务定位
本专题是 Assets and Projects 中的 asset-tracking-and-eam 子域。覆盖固定资产、iAssets、eAM、Projects、项目成本/开票/计划/合同/Grants、项目转资产、物业与租赁。

<a id="src-docs-05-assets-projects-asset-tracking-and-eam-readme--设计与配置"></a>
### 设计与配置
明确可资本化政策、项目/任务/资产类别/账簿、成本归集、资产行、Mass Additions、折旧和处置；项目成本、CIP、FA 和 GL 分别对账。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-05-assets-projects-asset-tracking-and-eam-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
FA_ADDITIONS_B、FA_BOOKS、FA_DISTRIBUTION_HISTORY、FA_DEPRN_SUMMARY、FA_MASS_ADDITIONS、PA_PROJECTS_ALL、PA_TASKS、PA_EXPENDITURE_ITEMS_ALL。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-05-assets-projects-asset-tracking-and-eam-readme--常见问题与排查"></a>
### 常见问题与排查
把费用化和资本化成本混同；只看资产头忽略账簿/分配历史；未确认可选 Projects/Property/Lease 产品范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-05-assets-projects-asset-tracking-and-eam-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-05-assets-projects-asset-tracking-and-eam-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-05-assets-projects-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/05-assets-projects/fixed-assets/README.md -->
<a id="src-docs-05-assets-projects-fixed-assets-readme"></a>
## Assets and Projects： fixed-assets


<a id="src-docs-05-assets-projects-fixed-assets-readme--业务定位"></a>
### 业务定位
本专题是 Assets and Projects 中的 fixed-assets 子域。覆盖固定资产、iAssets、eAM、Projects、项目成本/开票/计划/合同/Grants、项目转资产、物业与租赁。

<a id="src-docs-05-assets-projects-fixed-assets-readme--设计与配置"></a>
### 设计与配置
明确可资本化政策、项目/任务/资产类别/账簿、成本归集、资产行、Mass Additions、折旧和处置；项目成本、CIP、FA 和 GL 分别对账。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-05-assets-projects-fixed-assets-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
FA_ADDITIONS_B、FA_BOOKS、FA_DISTRIBUTION_HISTORY、FA_DEPRN_SUMMARY、FA_MASS_ADDITIONS、PA_PROJECTS_ALL、PA_TASKS、PA_EXPENDITURE_ITEMS_ALL。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-05-assets-projects-fixed-assets-readme--常见问题与排查"></a>
### 常见问题与排查
把费用化和资本化成本混同；只看资产头忽略账簿/分配历史；未确认可选 Projects/Property/Lease 产品范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-05-assets-projects-fixed-assets-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-05-assets-projects-fixed-assets-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-05-assets-projects-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/05-assets-projects/grants-accounting/README.md -->
<a id="src-docs-05-assets-projects-grants-accounting-readme"></a>
## Assets and Projects： grants-accounting


<a id="src-docs-05-assets-projects-grants-accounting-readme--业务定位"></a>
### 业务定位
本专题是 Assets and Projects 中的 grants-accounting 子域。覆盖固定资产、iAssets、eAM、Projects、项目成本/开票/计划/合同/Grants、项目转资产、物业与租赁。

<a id="src-docs-05-assets-projects-grants-accounting-readme--设计与配置"></a>
### 设计与配置
明确可资本化政策、项目/任务/资产类别/账簿、成本归集、资产行、Mass Additions、折旧和处置；项目成本、CIP、FA 和 GL 分别对账。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-05-assets-projects-grants-accounting-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
FA_ADDITIONS_B、FA_BOOKS、FA_DISTRIBUTION_HISTORY、FA_DEPRN_SUMMARY、FA_MASS_ADDITIONS、PA_PROJECTS_ALL、PA_TASKS、PA_EXPENDITURE_ITEMS_ALL。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-05-assets-projects-grants-accounting-readme--常见问题与排查"></a>
### 常见问题与排查
把费用化和资本化成本混同；只看资产头忽略账簿/分配历史；未确认可选 Projects/Property/Lease 产品范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-05-assets-projects-grants-accounting-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-05-assets-projects-grants-accounting-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-05-assets-projects-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/05-assets-projects/iassets/README.md -->
<a id="src-docs-05-assets-projects-iassets-readme"></a>
## Assets and Projects： iassets


<a id="src-docs-05-assets-projects-iassets-readme--业务定位"></a>
### 业务定位
本专题是 Assets and Projects 中的 iassets 子域。覆盖固定资产、iAssets、eAM、Projects、项目成本/开票/计划/合同/Grants、项目转资产、物业与租赁。

<a id="src-docs-05-assets-projects-iassets-readme--设计与配置"></a>
### 设计与配置
明确可资本化政策、项目/任务/资产类别/账簿、成本归集、资产行、Mass Additions、折旧和处置；项目成本、CIP、FA 和 GL 分别对账。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-05-assets-projects-iassets-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
FA_ADDITIONS_B、FA_BOOKS、FA_DISTRIBUTION_HISTORY、FA_DEPRN_SUMMARY、FA_MASS_ADDITIONS、PA_PROJECTS_ALL、PA_TASKS、PA_EXPENDITURE_ITEMS_ALL。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-05-assets-projects-iassets-readme--常见问题与排查"></a>
### 常见问题与排查
把费用化和资本化成本混同；只看资产头忽略账簿/分配历史；未确认可选 Projects/Property/Lease 产品范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-05-assets-projects-iassets-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-05-assets-projects-iassets-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-05-assets-projects-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/05-assets-projects/lease-and-finance-management/README.md -->
<a id="src-docs-05-assets-projects-lease-and-finance-management-readme"></a>
## Assets and Projects： lease-and-finance-management


<a id="src-docs-05-assets-projects-lease-and-finance-management-readme--业务定位"></a>
### 业务定位
本专题是 Assets and Projects 中的 lease-and-finance-management 子域。覆盖固定资产、iAssets、eAM、Projects、项目成本/开票/计划/合同/Grants、项目转资产、物业与租赁。

<a id="src-docs-05-assets-projects-lease-and-finance-management-readme--设计与配置"></a>
### 设计与配置
明确可资本化政策、项目/任务/资产类别/账簿、成本归集、资产行、Mass Additions、折旧和处置；项目成本、CIP、FA 和 GL 分别对账。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-05-assets-projects-lease-and-finance-management-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
FA_ADDITIONS_B、FA_BOOKS、FA_DISTRIBUTION_HISTORY、FA_DEPRN_SUMMARY、FA_MASS_ADDITIONS、PA_PROJECTS_ALL、PA_TASKS、PA_EXPENDITURE_ITEMS_ALL。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-05-assets-projects-lease-and-finance-management-readme--常见问题与排查"></a>
### 常见问题与排查
把费用化和资本化成本混同；只看资产头忽略账簿/分配历史；未确认可选 Projects/Property/Lease 产品范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-05-assets-projects-lease-and-finance-management-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-05-assets-projects-lease-and-finance-management-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-05-assets-projects-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/05-assets-projects/project-billing/README.md -->
<a id="src-docs-05-assets-projects-project-billing-readme"></a>
## Assets and Projects： project-billing


<a id="src-docs-05-assets-projects-project-billing-readme--业务定位"></a>
### 业务定位
本专题是 Assets and Projects 中的 project-billing 子域。覆盖固定资产、iAssets、eAM、Projects、项目成本/开票/计划/合同/Grants、项目转资产、物业与租赁。

<a id="src-docs-05-assets-projects-project-billing-readme--设计与配置"></a>
### 设计与配置
明确可资本化政策、项目/任务/资产类别/账簿、成本归集、资产行、Mass Additions、折旧和处置；项目成本、CIP、FA 和 GL 分别对账。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-05-assets-projects-project-billing-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
FA_ADDITIONS_B、FA_BOOKS、FA_DISTRIBUTION_HISTORY、FA_DEPRN_SUMMARY、FA_MASS_ADDITIONS、PA_PROJECTS_ALL、PA_TASKS、PA_EXPENDITURE_ITEMS_ALL。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-05-assets-projects-project-billing-readme--常见问题与排查"></a>
### 常见问题与排查
把费用化和资本化成本混同；只看资产头忽略账簿/分配历史；未确认可选 Projects/Property/Lease 产品范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-05-assets-projects-project-billing-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-05-assets-projects-project-billing-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-05-assets-projects-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/05-assets-projects/project-contracts/README.md -->
<a id="src-docs-05-assets-projects-project-contracts-readme"></a>
## Assets and Projects： project-contracts


<a id="src-docs-05-assets-projects-project-contracts-readme--业务定位"></a>
### 业务定位
本专题是 Assets and Projects 中的 project-contracts 子域。覆盖固定资产、iAssets、eAM、Projects、项目成本/开票/计划/合同/Grants、项目转资产、物业与租赁。

<a id="src-docs-05-assets-projects-project-contracts-readme--设计与配置"></a>
### 设计与配置
明确可资本化政策、项目/任务/资产类别/账簿、成本归集、资产行、Mass Additions、折旧和处置；项目成本、CIP、FA 和 GL 分别对账。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-05-assets-projects-project-contracts-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
FA_ADDITIONS_B、FA_BOOKS、FA_DISTRIBUTION_HISTORY、FA_DEPRN_SUMMARY、FA_MASS_ADDITIONS、PA_PROJECTS_ALL、PA_TASKS、PA_EXPENDITURE_ITEMS_ALL。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-05-assets-projects-project-contracts-readme--常见问题与排查"></a>
### 常见问题与排查
把费用化和资本化成本混同；只看资产头忽略账簿/分配历史；未确认可选 Projects/Property/Lease 产品范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-05-assets-projects-project-contracts-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-05-assets-projects-project-contracts-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-05-assets-projects-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/05-assets-projects/project-costing/README.md -->
<a id="src-docs-05-assets-projects-project-costing-readme"></a>
## Assets and Projects： project-costing


<a id="src-docs-05-assets-projects-project-costing-readme--业务定位"></a>
### 业务定位
本专题是 Assets and Projects 中的 project-costing 子域。覆盖固定资产、iAssets、eAM、Projects、项目成本/开票/计划/合同/Grants、项目转资产、物业与租赁。

<a id="src-docs-05-assets-projects-project-costing-readme--设计与配置"></a>
### 设计与配置
明确可资本化政策、项目/任务/资产类别/账簿、成本归集、资产行、Mass Additions、折旧和处置；项目成本、CIP、FA 和 GL 分别对账。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-05-assets-projects-project-costing-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
FA_ADDITIONS_B、FA_BOOKS、FA_DISTRIBUTION_HISTORY、FA_DEPRN_SUMMARY、FA_MASS_ADDITIONS、PA_PROJECTS_ALL、PA_TASKS、PA_EXPENDITURE_ITEMS_ALL。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-05-assets-projects-project-costing-readme--常见问题与排查"></a>
### 常见问题与排查
把费用化和资本化成本混同；只看资产头忽略账簿/分配历史；未确认可选 Projects/Property/Lease 产品范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-05-assets-projects-project-costing-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-05-assets-projects-project-costing-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-05-assets-projects-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/05-assets-projects/project-planning-control/README.md -->
<a id="src-docs-05-assets-projects-project-planning-control-readme"></a>
## Assets and Projects： project-planning-control


<a id="src-docs-05-assets-projects-project-planning-control-readme--业务定位"></a>
### 业务定位
本专题是 Assets and Projects 中的 project-planning-control 子域。覆盖固定资产、iAssets、eAM、Projects、项目成本/开票/计划/合同/Grants、项目转资产、物业与租赁。

<a id="src-docs-05-assets-projects-project-planning-control-readme--设计与配置"></a>
### 设计与配置
明确可资本化政策、项目/任务/资产类别/账簿、成本归集、资产行、Mass Additions、折旧和处置；项目成本、CIP、FA 和 GL 分别对账。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-05-assets-projects-project-planning-control-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
FA_ADDITIONS_B、FA_BOOKS、FA_DISTRIBUTION_HISTORY、FA_DEPRN_SUMMARY、FA_MASS_ADDITIONS、PA_PROJECTS_ALL、PA_TASKS、PA_EXPENDITURE_ITEMS_ALL。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-05-assets-projects-project-planning-control-readme--常见问题与排查"></a>
### 常见问题与排查
把费用化和资本化成本混同；只看资产头忽略账簿/分配历史；未确认可选 Projects/Property/Lease 产品范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-05-assets-projects-project-planning-control-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-05-assets-projects-project-planning-control-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-05-assets-projects-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/05-assets-projects/project-to-asset/README.md -->
<a id="src-docs-05-assets-projects-project-to-asset-readme"></a>
## Assets and Projects： project-to-asset


<a id="src-docs-05-assets-projects-project-to-asset-readme--业务定位"></a>
### 业务定位
本专题是 Assets and Projects 中的 project-to-asset 子域。覆盖固定资产、iAssets、eAM、Projects、项目成本/开票/计划/合同/Grants、项目转资产、物业与租赁。

<a id="src-docs-05-assets-projects-project-to-asset-readme--设计与配置"></a>
### 设计与配置
明确可资本化政策、项目/任务/资产类别/账簿、成本归集、资产行、Mass Additions、折旧和处置；项目成本、CIP、FA 和 GL 分别对账。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-05-assets-projects-project-to-asset-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
FA_ADDITIONS_B、FA_BOOKS、FA_DISTRIBUTION_HISTORY、FA_DEPRN_SUMMARY、FA_MASS_ADDITIONS、PA_PROJECTS_ALL、PA_TASKS、PA_EXPENDITURE_ITEMS_ALL。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-05-assets-projects-project-to-asset-readme--常见问题与排查"></a>
### 常见问题与排查
把费用化和资本化成本混同；只看资产头忽略账簿/分配历史；未确认可选 Projects/Property/Lease 产品范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-05-assets-projects-project-to-asset-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-05-assets-projects-project-to-asset-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-05-assets-projects-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/05-assets-projects/projects-foundation/README.md -->
<a id="src-docs-05-assets-projects-projects-foundation-readme"></a>
## Assets and Projects： projects-foundation


<a id="src-docs-05-assets-projects-projects-foundation-readme--业务定位"></a>
### 业务定位
本专题是 Assets and Projects 中的 projects-foundation 子域。覆盖固定资产、iAssets、eAM、Projects、项目成本/开票/计划/合同/Grants、项目转资产、物业与租赁。

<a id="src-docs-05-assets-projects-projects-foundation-readme--设计与配置"></a>
### 设计与配置
明确可资本化政策、项目/任务/资产类别/账簿、成本归集、资产行、Mass Additions、折旧和处置；项目成本、CIP、FA 和 GL 分别对账。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-05-assets-projects-projects-foundation-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
FA_ADDITIONS_B、FA_BOOKS、FA_DISTRIBUTION_HISTORY、FA_DEPRN_SUMMARY、FA_MASS_ADDITIONS、PA_PROJECTS_ALL、PA_TASKS、PA_EXPENDITURE_ITEMS_ALL。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-05-assets-projects-projects-foundation-readme--常见问题与排查"></a>
### 常见问题与排查
把费用化和资本化成本混同；只看资产头忽略账簿/分配历史；未确认可选 Projects/Property/Lease 产品范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-05-assets-projects-projects-foundation-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-05-assets-projects-projects-foundation-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-05-assets-projects-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/05-assets-projects/property-manager/README.md -->
<a id="src-docs-05-assets-projects-property-manager-readme"></a>
## Assets and Projects： property-manager


<a id="src-docs-05-assets-projects-property-manager-readme--业务定位"></a>
### 业务定位
本专题是 Assets and Projects 中的 property-manager 子域。覆盖固定资产、iAssets、eAM、Projects、项目成本/开票/计划/合同/Grants、项目转资产、物业与租赁。

<a id="src-docs-05-assets-projects-property-manager-readme--设计与配置"></a>
### 设计与配置
明确可资本化政策、项目/任务/资产类别/账簿、成本归集、资产行、Mass Additions、折旧和处置；项目成本、CIP、FA 和 GL 分别对账。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

<a id="src-docs-05-assets-projects-property-manager-readme--数据接口与会计追溯"></a>
### 数据、接口与会计追溯
FA_ADDITIONS_B、FA_BOOKS、FA_DISTRIBUTION_HISTORY、FA_DEPRN_SUMMARY、FA_MASS_ADDITIONS、PA_PROJECTS_ALL、PA_TASKS、PA_EXPENDITURE_ITEMS_ALL。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

<a id="src-docs-05-assets-projects-property-manager-readme--常见问题与排查"></a>
### 常见问题与排查
把费用化和资本化成本混同；只看资产头忽略账簿/分配历史；未确认可选 Projects/Property/Lease 产品范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

<a id="src-docs-05-assets-projects-property-manager-readme--实施边界"></a>
### 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

<a id="src-docs-05-assets-projects-property-manager-readme--关联与官方依据"></a>
### 关联与官方依据
[本知识域入口](#src-docs-05-assets-projects-readme)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/05-fa/README.md -->
<a id="src-docs-05-fa-readme"></a>
## Oracle Assets（FA / Acquire to Retire）


本目录覆盖资产账簿、类别、增加、资本化、折旧、调整、转移、处置、Mass Additions、会计与关账。资产来源可来自 AP、Projects/CIP、iAssets 或外部迁移；不同来源需要保留来源单据和资产编号之间的可追溯链。

<a id="src-docs-05-fa-readme--专题导航"></a>
### 专题导航

- [资产生命周期](#src-docs-05-fa-process)
- [账簿、类别、位置与配置](#src-docs-05-fa-setup)
- [增加、调整、转移、重分类与盘点](#src-docs-05-fa-asset-transactions)
- [折旧、税务折旧、处置与会计](#src-docs-05-fa-depreciation-accounting)
- [月结、报表、Mass Additions 与排错](#src-docs-05-fa-close-reports-interfaces)
- [Projects 到 Assets：CIP 与资本化](#src-docs-05-fa-projects-capitalization)
- [表结构](#src-docs-05-fa-tables)
- [Mass Additions 与迁移接口](#src-docs-05-fa-interfaces)

<a id="src-docs-05-fa-readme--会计与控制重点"></a>
### 会计与控制重点

| 业务动作 | 需确认的决定因素 | 常见遗漏 |
| --- | --- | --- |
| Capitalize | Asset Book、Category、Date Placed in Service、成本、资产来源 | 将 CIP、费用化和可资本化支出混在同一规则中 |
| Depreciate | Method、Life、Convention、Prorate、Period | 忘记先处理资产交易或未关闭前序模块期间 |
| Transfer/Adjust | Distribution、Location、Expense Account、Cost/Reserve | 只看资产头，遗漏分配行历史和会计影响 |
| Retire | Proceeds、Removal Cost、Partial Units、Gain/Loss | 处置日期/期间不一致，或遗漏 AP/AR/CE 清算链 |

<a id="src-docs-05-fa-readme--r122-边界"></a>
### R12.2 边界

使用 Mass Additions 或 Oracle 公开 API 处理集成与迁移；不直接更新 `FA_ADDITIONS_B`、`FA_BOOKS` 或 `FA_DISTRIBUTION_HISTORY`。资产账簿、税务账簿和折旧规则变更应完成影响分析并留存审批。

<a id="src-docs-05-fa-readme--官方依据"></a>
### 官方依据

- [Oracle Assets Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/05-fa/asset-transactions.md -->
<a id="src-docs-05-fa-asset-transactions"></a>
## FA 资产增加、调整、转移、重分类与盘点


<a id="src-docs-05-fa-asset-transactions--交易类型"></a>
### 交易类型

- Addition/CIP Addition：建立资产、Book 和 Distribution；CIP 通过 Capitalization 开始折旧。
- Cost/Book Adjustment：调整 Cost、Salvage、Life、Method、Rate、DPIS，可产生 Catch-up/Expensed Adjustment。
- Transfer：在 Employee/Expense Account/Location 间分配数量转移，总 Units 需平衡。
- Reclassification：改变 Category，可引起账户转移和折旧属性改变。
- Physical Inventory：将现场盘点与 FA Location/Employee 对比，差异审批后执行转移/处置。

<a id="src-docs-05-fa-asset-transactions--sql"></a>
### SQL

```sql
-- 当前分配（DATE_INEFFECTIVE 为空）
SELECT fdh.distribution_id, fdh.asset_id, fdh.book_type_code,
       fdh.units_assigned, fdh.code_combination_id,
       fdh.location_id, fdh.assigned_to,
       fdh.date_effective, fdh.date_ineffective
  FROM fa_distribution_history fdh
 WHERE fdh.asset_id = :p_asset_id
 ORDER BY fdh.date_effective, fdh.distribution_id;

SELECT fat.transaction_header_id, fat.transaction_type_code,
       fat.book_type_code, fat.transaction_date_entered,
       fat.date_effective, fat.transaction_name,
       fat.source_transaction_header_id
  FROM fa_transaction_headers fat
 WHERE fat.asset_id = :p_asset_id
 ORDER BY fat.date_effective, fat.transaction_header_id;
```

<a id="src-docs-05-fa-asset-transactions--排查"></a>
### 排查

- Transfer 不平：比较 Transfer Out/In Units，检查当前 Distribution 行、Location/Employee/CCID 有效性。
- Adjustment 不可做：查 Asset/Book Status、Period、已运行折旧、Retirement 和源交易限制。
- Reclass 后会计异常：比较新旧 Category Book 账户、交易日期和 SLA 行。
- Physical Inventory 差异太多：先统一 Asset Number/Tag/Location 映射与盘点截止日，再处理已退役/在途转移。

<a id="src-docs-05-fa-asset-transactions--关联"></a>
### 关联

- [FA Setup](#src-docs-05-fa-setup)
- [Depreciation](#src-docs-05-fa-depreciation-accounting)


<!-- source: docs/05-fa/close-reports-interfaces.md -->
<a id="src-docs-05-fa-close-reports-interfaces"></a>
## FA 月结、报表、Mass Additions 与排错


> `FA_MASS_ADDITIONS`、遗留资产迁移、Prepare/Post 和资产对账代码见 [FA 接口实现案例](#src-docs-05-fa-interfaces)。

<a id="src-docs-05-fa-close-reports-interfaces--mass-additions"></a>
### Mass Additions

```text
AP/Projects/External Source
→ FA_MASS_ADDITIONS
→ Prepare Mass Additions
→ Review/Merge/Split/Assign Category
→ Post Mass Additions
→ Asset/Book/Distribution
```

`POSTING_STATUS` 表示 New/On Hold/Posted/Delete/Error 等处理状态（具体 lookup 以实例为准）。一条 AP 分配是否进入 FA 取决于 Asset Tracking/Category/Account、Transfer to GL/FA 和接口程序。

<a id="src-docs-05-fa-close-reports-interfaces--月结"></a>
### 月结

1. 完成 Mass Additions、CIP Capitalization、Adjustments/Transfers/Retirements。
2. 运行并复核 Depreciation，处理异常资产和未完交易。
3. 运行 Create Accounting Final、Transfer to GL、Journal Import/Post。
4. 对账 Asset Cost、CIP、Reserve、Depreciation Expense、Retirement Gain/Loss、Clearing。
5. 运行 Asset Register、Reserve Ledger、Cost Detail、CIP Detail、Retirement 和 Account Reconciliation 报表，关闭 FA 期间。

<a id="src-docs-05-fa-close-reports-interfaces--sql"></a>
### SQL

```sql
SELECT mass_addition_id, book_type_code, description,
       fixed_assets_cost, payables_cost, posting_status,
       queue_name, asset_category_id, expense_code_combination_id,
       feeder_system_name, invoice_number, po_number,
       invoice_distribution_id, request_id
  FROM fa_mass_additions
 WHERE book_type_code = :p_book_type_code
   AND posting_status <> 'POSTED'
 ORDER BY mass_addition_id;

SELECT fdp.book_type_code, fdp.period_name, fdp.period_counter,
       fdp.period_open_date, fdp.period_close_date,
       fdp.deprn_run
  FROM fa_deprn_periods fdp
 WHERE fdp.book_type_code = :p_book_type_code
 ORDER BY fdp.period_counter DESC;

SELECT xah.gl_transfer_status_code, COUNT(*) cnt
  FROM xla_ae_headers xah
 WHERE xah.application_id = 140
   AND xah.accounting_date BETWEEN :p_start_date AND :p_end_date
 GROUP BY xah.gl_transfer_status_code;
```

<a id="src-docs-05-fa-close-reports-interfaces--排查"></a>
### 排查

- AP 行未进 FA：查 Track as Asset、Asset Clearing Account、AP 会计/转 GL、Create Mass Additions 参数和已转标志。
- Mass Addition 不能 Post：查 Category/Book、DPIS、Cost、Units、Location/Employee/Expense Account、Posting Status/Error。
- FA/GL 不平：区分未会计、未转/未过账、GL 手工分录、日期错位和 Asset Category 账户变更。
- 期间无法关闭：检查 Depreciation Run、Pending Transactions、Mass Additions、Accounting 和当期报表。

<a id="src-docs-05-fa-close-reports-interfaces--关联"></a>
### 关联

- [FA Process](#src-docs-05-fa-process)
- [Projects to Assets](09-end-to-end.md#src-docs-08-e2e-projects-assets)

<a id="src-docs-05-fa-close-reports-interfaces--官方参考"></a>
### 官方参考

- [Oracle E-Business Suite R12.2 Documentation Library](https://docs.oracle.com/cd/E26401_01/index.htm)


<!-- source: docs/05-fa/depreciation-accounting.md -->
<a id="src-docs-05-fa-depreciation-accounting"></a>
## FA 折旧、税务折旧、资产处置与会计


<a id="src-docs-05-fa-depreciation-accounting--折旧原理"></a>
### 折旧原理

折旧由 Cost Basis、Method/Rate/Life、Salvage Value、Depreciation Ceiling、Prorate Convention、DPIS、Calendar 和已提折旧决定。Run Depreciation 对当期资产计算，关期后结果进入 SLA/GL。Rollback 仅在标准程序允许的未关闭场景使用。

处置可为 Full/Partial Retirement，根据 Proceeds、Cost of Removal、Net Book Value 计算 Gain/Loss；Reinstatement 撤销处置并重建折旧/会计影响。

<a id="src-docs-05-fa-depreciation-accounting--sql"></a>
### SQL

```sql
SELECT fds.asset_id, fds.book_type_code, fds.period_counter,
       fds.deprn_amount, fds.ytd_deprn, fds.deprn_reserve,
       fds.deprn_adjustment_amount, fds.bonus_deprn_amount,
       fds.impairment_amount, fds.system_deprn_amount
  FROM fa_deprn_summary fds
 WHERE fds.asset_id = :p_asset_id
   AND fds.book_type_code = :p_book_type_code
 ORDER BY fds.period_counter;

SELECT retirement_id, asset_id, book_type_code,
       date_retired, cost_retired, proceeds_of_sale,
       cost_of_removal, status, gain_loss_amount,
       units, transaction_header_id_in,
       transaction_header_id_out
  FROM fa_retirements
 WHERE asset_id = :p_asset_id
 ORDER BY date_retired;
```

<a id="src-docs-05-fa-depreciation-accounting--排查"></a>
### 排查

- 资产未折旧：查 `DEPRECIATE_FLAG`、DPIS/Prorate Date、Asset Type、Cost、Method/Life、Fully Reserved/Retired 状态。
- 折旧金额不对：比较 Book/Method/Calendar、Cost Adjustments、Catch-up、Salvage/Ceiling、Bonus/Impairment 和舍入。
- Depreciation 请求失败：查日志中首个 Asset ID/Error，检查未完成 Mass Transaction、无效账户和并发冲突。
- Gain/Loss 异常：核对 Cost Retired、Reserve Retired、Proceeds/Removal、Retirement Convention 和处置日期。
- Tax Book 折旧差异：确认是政策差异而非 Mass Copy 遗漏，比较 Corporate/Tax Book 交易链。

<a id="src-docs-05-fa-depreciation-accounting--关联"></a>
### 关联

- [FA Transactions](#src-docs-05-fa-asset-transactions)
- [FA Close/Interface](#src-docs-05-fa-close-reports-interfaces)


<!-- source: docs/05-fa/interfaces.md -->
<a id="src-docs-05-fa-interfaces"></a>
## Oracle Assets 接口实现案例


<a id="src-docs-05-fa-interfaces--1-业界常用场景"></a>
### 1. 业界常用场景

| 场景 | 推荐接口 | 业务说明 |
| --- | --- | --- |
| AP 采购发票资本化 | Create Mass Additions | AP 已核算资产行进入 FA 待处理队列，保留 Invoice/PO 追溯 |
| Projects CIP 转固 | PRC: Interface Assets + Mass Additions | 项目资产线按 Project/Task/Asset Line 追溯 |
| 遗留资产迁移 | `FA_MASS_ADDITIONS` + Prepare/Post Mass Additions | 按 Book 分批导入成本、累计折旧和启用日 |
| 租赁/资产管理系统新增资产 | 自定义暂存层 + `FA_MASS_ADDITIONS` | 先做类别、地点、责任人、成本账户校验 |
| 大批量资产调整/转移/处置 | Oracle Assets 公共 API/标准批处理 | 以当前 Integration Repository/API 文档签名为准，不直接改 FA 历史表 |

<a id="src-docs-05-fa-interfaces--2-mass-additions-业务状态"></a>
### 2. Mass Additions 业务状态

外部来源通常只创建 `NEW` 待处理行，由资产会计在 Mass Additions Workbench 完善并置为可过账，再运行 Post Mass Additions。典型过程如下：

```text
Source/Staging → NEW → Review/Prepare → POST → Posted Asset
                       ↘ HOLD / MERGED / SPLIT / ERROR
```

状态代码、Queue 名称和允许转换必须以目标实例 FA Lookup 和标准界面为准。不要通过 `UPDATE FA_MASS_ADDITIONS` 人工推动状态。

<a id="src-docs-05-fa-interfaces--3-导入前校验"></a>
### 3. 导入前校验

```sql
-- 资产账簿和当前期间
SELECT fbc.book_type_code,
       fbc.set_of_books_id,
       fdp.period_name,
       fdp.period_open_date,
       fdp.period_close_date
  FROM fa_book_controls fbc
  LEFT JOIN fa_deprn_periods fdp
    ON fdp.book_type_code = fbc.book_type_code
   AND fdp.period_close_date IS NULL
 WHERE fbc.book_type_code = :p_book_type_code;

-- 类别在该 Book 的默认账户/折旧设置是否存在
SELECT fcb.category_id,
       fcb.book_type_code,
       fcb.asset_cost_acct,
       fcb.asset_clearing_acct,
       fcb.deprn_expense_acct
  FROM fa_category_books fcb
 WHERE fcb.category_id = :p_asset_category_id
   AND fcb.book_type_code = :p_book_type_code;

-- 位置键是否有效；段数按实例 Location KFF 调整
SELECT fl.location_id, fl.segment1, fl.segment2, fl.enabled_flag
  FROM fa_locations fl
 WHERE fl.location_id = :p_location_id;
```

员工、地点、类别、费用 CCID 都存在并不代表在启用日有效；接口程序应按 `DATE_PLACED_IN_SERVICE` 做有效期校验。

<a id="src-docs-05-fa-interfaces--4-famassadditions-具体实现"></a>
### 4. `FA_MASS_ADDITIONS` 具体实现

<a id="src-docs-05-fa-interfaces--41-外部资产新增"></a>
#### 4.1 外部资产新增

```sql
DECLARE
  l_mass_addition_id NUMBER := fa_mass_additions_s.NEXTVAL;
BEGIN
  INSERT INTO fa_mass_additions (
    mass_addition_id,
    asset_number,
    tag_number,
    description,
    asset_category_id,
    book_type_code,
    date_placed_in_service,
    fixed_assets_cost,
    payables_cost,
    payables_units,
    payables_code_combination_id,
    expense_code_combination_id,
    location_id,
    assigned_to,
    feeder_system_name,
    posting_status,
    queue_name,
    invoice_number,
    vendor_number,
    created_by,
    creation_date,
    last_updated_by,
    last_update_date,
    last_update_login
  ) VALUES (
    l_mass_addition_id,
    NULL,                             -- 由 FA 自动编号时留空
    :p_tag_number,
    :p_description,
    :p_asset_category_id,
    :p_book_type_code,
    :p_date_placed_in_service,
    :p_asset_cost,
    :p_asset_cost,
    1,
    :p_asset_clearing_ccid,
    :p_deprn_expense_ccid,
    :p_location_id,
    :p_employee_id,
    'XX ASSET HUB',
    'NEW',
    'NEW',
    :p_external_document_number,
    :p_supplier_number,
    fnd_global.user_id,
    SYSDATE,
    fnd_global.user_id,
    SYSDATE,
    fnd_global.login_id
  );

  COMMIT;
  dbms_output.put_line('MASS_ADDITION_ID=' || l_mass_addition_id);
END;
/
```

`FA_MASS_ADDITIONS` 的列和必填规则会受来源、Book、功能和补丁影响。上线前使用目标实例 eTRM/`ALL_TAB_COLUMNS` 复核列，并用一条标准 AP/Projects 生成的 Mass Addition 作为字段映射样本。

<a id="src-docs-05-fa-interfaces--42-运行前核对目标列"></a>
#### 4.2 运行前核对目标列

```sql
SELECT column_id,
       column_name,
       data_type,
       data_length,
       nullable
  FROM all_tab_columns
 WHERE owner = 'FA'
   AND table_name = 'FA_MASS_ADDITIONS'
 ORDER BY column_id;
```

自定义程序应将源系统主键保存在自定义暂存/映射表中，并以唯一约束保证幂等。不要依赖 `DESCRIPTION`、`INVOICE_NUMBER` 或 `TAG_NUMBER` 单列作为全局唯一键。

<a id="src-docs-05-fa-interfaces--5-遗留资产迁移的成本和累计折旧"></a>
### 5. 遗留资产迁移的成本和累计折旧

遗留迁移不只是插入当前成本。至少要确认：

- Corporate/Tax Book 的启用日、原始成本、当前成本和净残值；
- 折旧方法、年限、Prorate Convention 和累计折旧；
- 本年累计折旧（YTD）和迁移期间；
- 当前地点、责任人、单位数和折旧费用账户；
- 资产类别默认值是否允许被源数据覆盖。

先用少量样本在关闭的测试环境走完 Prepare/Post/Depreciation，再核对剩余价值和下一期折旧。不要用 DML 直接补 `FA_BOOKS`、`FA_DEPRN_SUMMARY` 或 `FA_DISTRIBUTION_HISTORY`。

<a id="src-docs-05-fa-interfaces--6-mass-additions-处理与监控"></a>
### 6. Mass Additions 处理与监控

标准流程通常为：

1. 运行 Create Mass Additions 或外部受控接口写入 Mass Additions。
2. 在 Mass Additions Workbench 合并、拆分、指定类别/地点/员工并处理异常。
3. 运行 Prepare Mass Additions，检查可过账条件。
4. 运行 Post Mass Additions，生成资产和 FA 交易历史。
5. 按 `MASS_ADDITION_ID`、资产号、Request ID 对账。

```sql
-- 队列与状态监控
SELECT feeder_system_name,
       book_type_code,
       posting_status,
       queue_name,
       COUNT(*) line_count,
       SUM(NVL(fixed_assets_cost, 0)) total_cost
  FROM fa_mass_additions
 WHERE feeder_system_name = 'XX ASSET HUB'
 GROUP BY feeder_system_name, book_type_code, posting_status, queue_name
 ORDER BY book_type_code, posting_status, queue_name;

-- 单笔接口追踪
SELECT mass_addition_id,
       posting_status,
       queue_name,
       asset_number,
       description,
       fixed_assets_cost,
       invoice_number,
       vendor_number
  FROM fa_mass_additions
 WHERE mass_addition_id = :p_mass_addition_id;
```

<a id="src-docs-05-fa-interfaces--7-成功结果对账"></a>
### 7. 成功结果对账

```sql
SELECT fma.mass_addition_id,
       fma.posting_status,
       fab.asset_id,
       fab.asset_number,
       fat.description,
       fb.book_type_code,
       fb.cost,
       fb.date_placed_in_service
  FROM fa_mass_additions fma
  JOIN fa_additions_b fab
    ON fab.asset_number = fma.asset_number
  LEFT JOIN fa_additions_tl fat
    ON fat.asset_id = fab.asset_id
   AND fat.language = USERENV('LANG')
  JOIN fa_books fb
    ON fb.asset_id = fab.asset_id
   AND fb.book_type_code = fma.book_type_code
   AND fb.date_ineffective IS NULL
 WHERE fma.mass_addition_id = :p_mass_addition_id;
```

部分来源/流程可能不会回写可直接关联的 `ASSET_NUMBER`。生产映射表应在 Posting 后保存 `MASS_ADDITION_ID → ASSET_ID/ASSET_NUMBER`，并以标准报表结果核验。

<a id="src-docs-05-fa-interfaces--8-常见问题与实现方法"></a>
### 8. 常见问题与实现方法

| 问题 | 常见原因 | 排查/处理 |
| --- | --- | --- |
| 行一直停留在 NEW | 类别、Book、启用日、地点或账户未准备 | 用 Workbench/Prepare Mass Additions 查看错误，不手工改状态 |
| 无法 Post | 期间、类别账簿设置、资产号、单位数或成本无效 | 校验 FA 当前期间和 Category Book Defaults |
| 重复资产 | 源消息重放、缺少幂等键 | 暂存表对 `SOURCE_SYSTEM + EXTERNAL_ASSET_ID + BOOK` 建唯一约束 |
| 折旧金额不符 | 方法、年限、Prorate、YTD/Reserve 映射错误 | 用标准资产样本模拟下一期折旧后再批量迁移 |
| 地点/员工无法选 | KFF/HR 有效期或 Security Profile 不匹配 | 按启用日查询有效记录，并核对职责权限 |

<a id="src-docs-05-fa-interfaces--9-关联文档"></a>
### 9. 关联文档

- [FA 增加、调整、转移与处置](#src-docs-05-fa-asset-transactions)
- [FA 折旧与会计](#src-docs-05-fa-depreciation-accounting)
- [FA 常用表](#src-docs-05-fa-tables)
- [项目与资产资本化](09-end-to-end.md#src-docs-08-e2e-projects-assets)

<a id="src-docs-05-fa-interfaces--10-官方参考"></a>
### 10. 官方参考

- [Oracle Assets User Guide: Mass Additions](https://docs.oracle.com/cd/E26401_01/doc.122/e48755/T293142T293157.htm)
- [Oracle Assets User Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48755/)
- [Oracle E-Business Suite eTRM User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/)


<!-- source: docs/05-fa/process.md -->
<a id="src-docs-05-fa-process"></a>
## Oracle Assets 资产全生命周期


<a id="src-docs-05-fa-process--流程"></a>
### 流程

```text
AP/CIP/Projects/Manual/Legacy
→ Mass Additions → Prepare/Post
→ Asset + Book + Distribution
→ Depreciation / Adjustment / Transfer / Reclass
→ Retirement/Reinstatement
→ Create Accounting → GL → Close
```

`FA_ADDITIONS_B` 保存资产主数据，`FA_BOOKS` 保存每个 Book 的成本/折旧属性历史，`FA_DISTRIBUTION_HISTORY` 保存责任人/费用账户/位置，`FA_TRANSACTION_HEADERS` 记录业务事件，`FA_DEPRN_SUMMARY/DETAIL` 保存折旧结果。

<a id="src-docs-05-fa-process--控制点"></a>
### 控制点

- Asset Category 决定默认账户和折旧属性，Asset Book 决定会计/税务表述。
- Corporate Book 与 Tax Book 通过 Mass Copy/Initial Mass Copy 关联，不应把税务调整直接混入公司账簿。
- 资产交易按 FA 期间和 Date Placed in Service 生效，回溯交易可引起 Catch-up Depreciation。

<a id="src-docs-05-fa-process--sql"></a>
### SQL

```sql
SELECT fab.asset_id, fab.asset_number, fab.description,
       fab.asset_category_id, fab.asset_type,
       fb.book_type_code, fb.date_placed_in_service,
       fb.cost, fb.original_cost, fb.salvage_value,
       fb.life_in_months, fb.deprn_method_code,
       fb.depreciate_flag, fb.date_ineffective
  FROM fa_additions_b fab
  JOIN fa_books fb ON fb.asset_id = fab.asset_id
 WHERE fab.asset_number = :p_asset_number
 ORDER BY fb.book_type_code, fb.date_effective;

SELECT transaction_header_id, asset_id, book_type_code,
       transaction_type_code, transaction_date_entered,
       date_effective, transaction_name, mass_reference_id
  FROM fa_transaction_headers
 WHERE asset_id = :p_asset_id
 ORDER BY date_effective, transaction_header_id;
```

<a id="src-docs-05-fa-process--排查"></a>
### 排查

- Asset Workbench 找不到：查 Book、Asset Number、Security by Book、有效历史行和职责。
- 交易日期不允许：查 FA Period、DPIS、已运行折旧、账簿开放期间和未完成批处理。
- 成本/分配不一致：沿 Transaction Header 查 Books/Distribution History，区分当前行与历史行。

<a id="src-docs-05-fa-process--关联"></a>
### 关联

- [FA 常用表结构与字段含义](#src-docs-05-fa-tables)
- [FA Setup](#src-docs-05-fa-setup)
- [Depreciation/Accounting](#src-docs-05-fa-depreciation-accounting)


<!-- source: docs/05-fa/projects-capitalization.md -->
<a id="src-docs-05-fa-projects-capitalization"></a>
## Projects 到 Assets：CIP 与资本化


<a id="src-docs-05-fa-projects-capitalization--业务边界"></a>
### 业务边界

资本项目通常由 Oracle Projects 累积成本、生成项目资产和资产行，再由 Oracle Assets 的 Mass Additions/资产增加流程资本化。不是所有项目成本均可资本化；资本化政策、项目类型、任务、资产分类和 In Service 日期必须由财务/项目控制共同治理。

<a id="src-docs-05-fa-projects-capitalization--推荐流程"></a>
### 推荐流程

```text
Project / Task / Expenditure Item
  → Cost Distribution / Burdening
  → Capital Project / Project Asset
  → Generate Asset Lines
  → Interface to FA Mass Additions
  → Prepare / Post Mass Additions
  → FA Asset / Depreciation / SLA / GL
```

<a id="src-docs-05-fa-projects-capitalization--配置与控制点"></a>
### 配置与控制点

- 定义可资本化项目类型、资产分类、CIP Clearing/Asset Clearing 账户、资产账簿、折旧方法及资产来源规则。
- 项目资产必须有唯一资产分组/来源追溯逻辑；避免按描述文本将同一项目资产重复送入 FA。
- 明确何时转固：达到可使用状态、相关成本冻结、验收完成和会计期间允许。生成资产行前确认成本分配和调整已完成。
- 建立 Projects 成本、CIP、FA Mass Additions、FA Asset Cost 和 GL 的四方对账，分别处理舍入、排除成本、未资本化成本和失败行。

<a id="src-docs-05-fa-projects-capitalization--只读诊断-sql"></a>
### 只读诊断 SQL

```sql
-- FA 侧以 Mass Additions 状态追踪项目/外部来源资产行；列和值以目标实例 eTRM 为准。
select fma.mass_addition_id,
       fma.asset_number,
       fma.description,
       fma.asset_cost,
       fma.posting_status,
       fma.queue_name,
       fma.creation_date
  from fa_mass_additions fma
 where fma.asset_number = :p_asset_number
 order by fma.creation_date desc;

-- 已资本化资产应同时检查资产头和账簿成本，而非只依据资产编号。
select fab.asset_id,
       fab.asset_number,
       fb.book_type_code,
       fb.cost,
       fb.date_placed_in_service
  from fa_additions_b fab
  join fa_books fb
    on fb.asset_id = fab.asset_id
 where fab.asset_number = :p_asset_number
   and fb.date_ineffective is null;
```

<a id="src-docs-05-fa-projects-capitalization--排错顺序"></a>
### 排错顺序

1. 在 Projects 确认项目/任务、成本分配、资本化资格和资产行是否生成。
2. 在 Mass Additions 确认状态、错误信息、资产分类、账簿、位置和资产来源字段。
3. 在 FA 确认 Prepare/Post、资产增加、折旧期间和会计创建；最后与项目/CIP/GL 对账。

<a id="src-docs-05-fa-projects-capitalization--官方参考"></a>
### 官方参考

- [Oracle Projects Documentation](https://docs.oracle.com/cd/E26401_01/nav/projects.htm)
- [Oracle Assets Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)


<!-- source: docs/05-fa/setup.md -->
<a id="src-docs-05-fa-setup"></a>
## FA 资产账簿、类别、位置与关键配置


<a id="src-docs-05-fa-setup--核心设置"></a>
### 核心设置

- **Asset Book**：Corporate/Tax，关联 Ledger、Calendar、Prorate Calendar、Deprn Calendar、SLA 和账户规则。
- **Category**：Major/Minor 弹性域组合，按 Book 默认 Asset Cost、Reserve、Expense、CIP、Clearing、Gain/Loss 账户。
- **Location**：Location KFF 用于实物位置与盘点，与 HR Location/Inventory Locator 不是同一对象。
- **Depreciation Method**：Straight Line/Table/Calculated/Production 等，结合 Life、Rate、Prorate Convention、Salvage/Ceiling。
- **Key Flexfields**：Category、Location、Asset Key；分配行还使用 GL Expense CCID 和 Employee。

<a id="src-docs-05-fa-setup--实施顺序"></a>
### 实施顺序

1. 定义 FA Calendar/Prorate Calendar、Fiscal Year、Methods/Conventions。
2. 定义 Category/Location/Asset Key KFF 及值，编译后创建 Category。
3. 定义 Corporate Book，分配 Category 账户和折旧默认。
4. 定义 Tax Book 与复制规则，配置 System Controls、Security by Book、Mass Additions、SLA。
5. 使用少量资产测试增加、折旧、调整、转移、处置和会计。

<a id="src-docs-05-fa-setup--sql"></a>
### SQL

```sql
SELECT book_type_code, book_class, set_of_books_id,
       initial_date, last_period_counter,
       deprn_calendar, prorate_calendar,
       current_fiscal_year, allow_mass_changes
  FROM fa_book_controls
 ORDER BY book_type_code;

SELECT fcb.category_id, fcb.segment1, fcb.segment2,
       fcb.enabled_flag, fcb.start_date_active, fcb.end_date_active,
       fcbt.description
  FROM fa_categories_b fcb
  LEFT JOIN fa_categories_tl fcbt
    ON fcbt.category_id = fcb.category_id
   AND fcbt.language = USERENV('LANG')
 ORDER BY fcb.segment1, fcb.segment2;

SELECT book_type_code, category_id,
       asset_cost_acct, asset_clearing_acct,
       deprn_reserve_acct, deprn_expense_acct,
       cip_cost_acct, cip_clearing_acct
  FROM fa_category_books
 WHERE book_type_code = :p_book_type_code;
```

<a id="src-docs-05-fa-setup--排查"></a>
### 排查

- Category 不可选：查 KFF 组合/值有效性、Category Enabled/Date、Book Assignment。
- 默认账户不对：查 Category Book 账户、Asset Type（Capitalized/CIP/Expense）和 SLA 覆盖。
- Tax Book 无数据：查 Corporate Book 关联、Initial/Mass Copy 参数、交易类型可复制性和请求日志。

<a id="src-docs-05-fa-setup--关联"></a>
### 关联

- [COA](01-foundation.md#src-docs-01-common-coa)
- [FA Process](#src-docs-05-fa-process)


<!-- source: docs/05-fa/tables.md -->
<a id="src-docs-05-fa-tables"></a>
## Oracle Assets 常用表结构


<a id="src-docs-05-fa-tables--业务说明"></a>
### 业务说明

FA 数据不能只查资产主表。一项资产的名称/类别在 Addition，成本/折旧属性按 Book 保存在 Books 历史，位置/责任人/费用账户在 Distribution History，每次业务变更在 Transaction Headers。当前行通常用 `DATE_INEFFECTIVE IS NULL` 识别，历史报表必须以业务截止日选取有效行。

<a id="src-docs-05-fa-tables--表级速查"></a>
### 表级速查

| 表 | 中文名 | 业务粒度 | 关键字段 |
| --- | --- | --- | --- |
| `FA_ADDITIONS_B` | 资产主数据 | 每项资产 | `ASSET_ID`, `ASSET_NUMBER`, `ASSET_CATEGORY_ID`, `ASSET_TYPE` |
| `FA_ADDITIONS_TL` | 资产多语言说明 | Asset+语言 | `ASSET_ID`, `LANGUAGE`, `DESCRIPTION` |
| `FA_BOOK_CONTROLS` | 资产账簿控制 | 每个 Asset Book | `BOOK_TYPE_CODE`, `SET_OF_BOOKS_ID`, Calendar/Period |
| `FA_BOOKS` | 资产账簿历史 | Asset+Book+有效期 | `ASSET_ID`, `BOOK_TYPE_CODE`, `TRANSACTION_HEADER_ID_IN/OUT` |
| `FA_CATEGORIES_B` | 资产类别 | 每个 Category KFF 组合 | `CATEGORY_ID`, `SEGMENT1..N` |
| `FA_CATEGORY_BOOKS` | 类别账簿设置 | Category+Book | 资产成本/折旧/CIP/处置账户 |
| `FA_DISTRIBUTION_HISTORY` | 资产分配历史 | Asset+Book+分配有效期 | `DISTRIBUTION_ID`, `LOCATION_ID`, `ASSIGNED_TO`, `CODE_COMBINATION_ID` |
| `FA_TRANSACTION_HEADERS` | 资产交易头 | 每次资产交易 | `TRANSACTION_HEADER_ID`, `TRANSACTION_TYPE_CODE`, `ASSET_ID` |
| `FA_DEPRN_PERIODS` | FA 折旧期间 | Book+Period | `BOOK_TYPE_CODE`, `PERIOD_COUNTER`, `PERIOD_OPEN/CLOSE_DATE` |
| `FA_DEPRN_SUMMARY` | 资产折旧汇总 | Asset+Book+Period | `ASSET_ID`, `BOOK_TYPE_CODE`, `PERIOD_COUNTER` |
| `FA_DEPRN_DETAIL` | 资产折旧分配明细 | Asset+Distribution+Period | `DISTRIBUTION_ID`, `DEPRN_AMOUNT`, `DEPRN_RESERVE` |
| `FA_RETIREMENTS` | 资产处置 | 每次 Full/Partial Retirement | `RETIREMENT_ID`, `ASSET_ID`, `STATUS` |
| `FA_MASS_ADDITIONS` | 批量资产增加接口 | 每个待处理资产行 | `MASS_ADDITION_ID`, `POSTING_STATUS`, `BOOK_TYPE_CODE` |

<a id="src-docs-05-fa-tables--faadditionsb-资产主数据"></a>
### `FA_ADDITIONS_B` — 资产主数据

| 字段 | 中文名 | 业务含义/常见值 |
| --- | --- | --- |
| `ASSET_ID` | 资产 ID | 所有 FA 历史表的核心关联键 |
| `ASSET_NUMBER` | 资产编号 | 可手工/自动生成，展示键 |
| `TAG_NUMBER` | 资产标签号 | 实物盘点常用，不一定所有资产都必填 |
| `ASSET_CATEGORY_ID` | 资产类别 ID | 决定各 Book 默认账户/折旧属性 |
| `ASSET_TYPE` | 资产类型 | 常见 `CAPITALIZED`、`CIP`、`EXPENSED`、`GROUP`；以 FA Lookup/已启用功能为准 |
| `CURRENT_UNITS` | 当前单位数 | 应与当前 Distribution History 单位合计一致 |
| `PARENT_ASSET_ID` | 父资产 ID | 组件/附属资产层级 |

<a id="src-docs-05-fa-tables--fabooks-资产账簿历史"></a>
### `FA_BOOKS` — 资产账簿历史

| 字段 | 中文名 | 业务说明 |
| --- | --- | --- |
| `BOOK_TYPE_CODE` | 资产账簿 | Corporate/Tax Book 关键键 |
| `DATE_EFFECTIVE/DATE_INEFFECTIVE` | 历史有效期 | `DATE_INEFFECTIVE IS NULL` 通常为当前账簿行 |
| `TRANSACTION_HEADER_ID_IN/OUT` | 生效/失效交易 | 将 Books 历史变化连回 Transaction Header |
| `COST` | 当前成本 | 当前历史行的 Book Cost |
| `ORIGINAL_COST` | 原始成本 | 不随普通成本调整同步表示“当前成本” |
| `SALVAGE_VALUE` | 净残值 | 影响 Depreciable Basis，受 Book 规则限制 |
| `DATE_PLACED_IN_SERVICE` | 启用日期 | 与 Prorate Convention 共同决定开始折旧日 |
| `DEPRN_METHOD_CODE` | 折旧方法 | 结合 `LIFE_IN_MONTHS`、Rate/Table 和 Convention |
| `DEPRECIATE_FLAG` | 是否计提折旧 | `YES/NO` 或实例对应标准值，以 FA Lookup/eTRM 为准 |
| `PERIOD_COUNTER_FULLY_RESERVED` | 完全折旧期 | 用于判断 Fully Reserved 时点 |
| `PERIOD_COUNTER_FULLY_RETIRED` | 完全处置期 | 用于判断 Full Retirement 时点 |

<a id="src-docs-05-fa-tables--fadistributionhistory-分配历史"></a>
### `FA_DISTRIBUTION_HISTORY` — 分配历史

| 字段 | 中文名 | 业务说明 |
| --- | --- | --- |
| `UNITS_ASSIGNED` | 分配单位数 | 当前所有分配行合计应与 Asset Current Units 一致 |
| `CODE_COMBINATION_ID` | 折旧费用账户 | 资产分配到的 GL Expense CCID，不是 Asset Cost Account |
| `LOCATION_ID` | FA 位置 ID | 关联 Asset Location KFF，不等于 HR Location/Inventory Locator |
| `ASSIGNED_TO` | 责任员工 ID | 通常关联 HR Person，需按有效日查员工名 |
| `DATE_INEFFECTIVE` | 失效日 | NULL 通常为当前分配；转移会关闭旧行并建新行 |

<a id="src-docs-05-fa-tables--fadeprnsummary-折旧汇总"></a>
### `FA_DEPRN_SUMMARY` — 折旧汇总

| 字段 | 中文名 | 说明 |
| --- | --- | --- |
| `DEPRN_AMOUNT` | 本期折旧 | 包含当期计算/调整影响，分析时还需查调整列 |
| `YTD_DEPRN` | 本年累计折旧 | Fiscal Year-to-Date，不是从启用日累计 |
| `DEPRN_RESERVE` | 累计折旧 | 至该期的折旧准备 |
| `DEPRN_ADJUSTMENT_AMOUNT` | 折旧调整 | 回溯 Cost/Life/Method 变更可产生 |
| `BONUS_DEPRN_AMOUNT` | 奖励折旧 | 只在相关税务/折旧功能启用时有业务意义 |
| `IMPAIRMENT_AMOUNT` | 减值金额 | 受资产减值功能和会计规则影响 |

<a id="src-docs-05-fa-tables--famassadditionspostingstatus"></a>
### `FA_MASS_ADDITIONS.POSTING_STATUS`

常见业务含义包括 New、On Hold、Posted、Delete、Merge/Split 过程和 Error。内部代码会随处理阶段改变，应通过 Mass Additions Queue/Posting Status Lookup 解码，不直接更新 `POSTING_STATUS` 推进数据。

<a id="src-docs-05-fa-tables--官方参考"></a>
### 官方参考

- [Oracle E-Business Suite R12.2 Documentation Library](https://docs.oracle.com/cd/E26401_01/index.htm)
- [Oracle E-Business Suite eTRM User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/)
