# Projects 到 Assets：CIP 与资本化

## 业务边界

资本项目通常由 Oracle Projects 累积成本、生成项目资产和资产行，再由 Oracle Assets 的 Mass Additions/资产增加流程资本化。不是所有项目成本均可资本化；资本化政策、项目类型、任务、资产分类和 In Service 日期必须由财务/项目控制共同治理。

## 推荐流程

```text
Project / Task / Expenditure Item
  → Cost Distribution / Burdening
  → Capital Project / Project Asset
  → Generate Asset Lines
  → Interface to FA Mass Additions
  → Prepare / Post Mass Additions
  → FA Asset / Depreciation / SLA / GL
```

## 配置与控制点

- 定义可资本化项目类型、资产分类、CIP Clearing/Asset Clearing 账户、资产账簿、折旧方法及资产来源规则。
- 项目资产必须有唯一资产分组/来源追溯逻辑；避免按描述文本将同一项目资产重复送入 FA。
- 明确何时转固：达到可使用状态、相关成本冻结、验收完成和会计期间允许。生成资产行前确认成本分配和调整已完成。
- 建立 Projects 成本、CIP、FA Mass Additions、FA Asset Cost 和 GL 的四方对账，分别处理舍入、排除成本、未资本化成本和失败行。

## 只读诊断 SQL

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

## 排错顺序

1. 在 Projects 确认项目/任务、成本分配、资本化资格和资产行是否生成。
2. 在 Mass Additions 确认状态、错误信息、资产分类、账簿、位置和资产来源字段。
3. 在 FA 确认 Prepare/Post、资产增加、折旧期间和会计创建；最后与项目/CIP/GL 对账。

## 官方参考

- [Oracle Projects Documentation](https://docs.oracle.com/cd/E26401_01/nav/projects.htm)
- [Oracle Assets Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
