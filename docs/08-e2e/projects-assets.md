# 项目、费用与资产资本化

## 流程

```text
Project/Task + Expenditure Items
→ Cost Distribution / Burdening / Accounting
→ Capital Project + Asset Lines
→ Generate/Interface Assets
→ FA_MASS_ADDITIONS
→ Post Mass Additions / CIP Asset
→ Capitalize → Depreciate → XLA/GL
```

项目资本化将符合条件的 Expenditure Item 汇集为 Project Asset Line，按 Asset Assignment/Grouping 传至 FA。需明确 Project/Task Capitalizable Flag、Asset Category/Book、CIP Cost Account、Date Placed in Service、Common/Specific Cost Allocation 和冲销规则。

## SQL

```sql
SELECT ppa.project_id, ppa.segment1 project_number,
       ppa.name project_name, ppa.project_status_code,
       ppa.project_type, ppa.carrying_out_organization_id,
       ppa.org_id
  FROM pa_projects_all ppa
 WHERE ppa.project_id = :p_project_id;

SELECT peia.expenditure_item_id, peia.project_id, peia.task_id,
       peia.expenditure_item_date, peia.expenditure_type,
       peia.quantity, peia.raw_cost, peia.burdened_cost,
       peia.cost_distributed_flag, peia.billable_flag,
       peia.denom_currency_code
  FROM pa_expenditure_items_all peia
 WHERE peia.project_id = :p_project_id
 ORDER BY peia.expenditure_item_date, peia.expenditure_item_id;

SELECT mass_addition_id, feeder_system_name, description,
       fixed_assets_cost, posting_status, book_type_code,
       asset_category_id, project_id, task_id
  FROM fa_mass_additions
 WHERE project_id = :p_project_id
 ORDER BY mass_addition_id;
```

## 排错

- Cost 未分配：查 Expenditure Type/Organization、Cost Rate、Burden Schedule、Period、Account Generation 和 Distribution 日志。
- 未生成 Asset Line：查 Project/Task Capitalizable、Expenditure Item 可资本化、已分配成本、Asset Assignment、Cutoff Date。
- 未进 FA：查 Interface Assets 请求、Book/Category、FA Period、Mass Addition Posting Status/Error。
- PA CIP/FA CIP/GL 不平：统一截止日，区分未分配、未生成 Asset Line、未 Interface、未 Post、已 Capitalize 和未过账。
- 调整已资本化成本：使用 PA/FA 标准冲销、调整和追加流程，不直接更新资产成本。

## 关联

- [FA Process](../05-fa/process.md)
- [FA Interface/Close](../05-fa/close-reports-interfaces.md)
