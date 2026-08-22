# Oracle Assets 资产全生命周期

## 流程

```text
AP/CIP/Projects/Manual/Legacy
→ Mass Additions → Prepare/Post
→ Asset + Book + Distribution
→ Depreciation / Adjustment / Transfer / Reclass
→ Retirement/Reinstatement
→ Create Accounting → GL → Close
```

`FA_ADDITIONS_B` 保存资产主数据，`FA_BOOKS` 保存每个 Book 的成本/折旧属性历史，`FA_DISTRIBUTION_HISTORY` 保存责任人/费用账户/位置，`FA_TRANSACTION_HEADERS` 记录业务事件，`FA_DEPRN_SUMMARY/DETAIL` 保存折旧结果。

## 控制点

- Asset Category 决定默认账户和折旧属性，Asset Book 决定会计/税务表述。
- Corporate Book 与 Tax Book 通过 Mass Copy/Initial Mass Copy 关联，不应把税务调整直接混入公司账簿。
- 资产交易按 FA 期间和 Date Placed in Service 生效，回溯交易可引起 Catch-up Depreciation。

## SQL

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

## 排查

- Asset Workbench 找不到：查 Book、Asset Number、Security by Book、有效历史行和职责。
- 交易日期不允许：查 FA Period、DPIS、已运行折旧、账簿开放期间和未完成批处理。
- 成本/分配不一致：沿 Transaction Header 查 Books/Distribution History，区分当前行与历史行。

## 关联

- [FA 常用表结构与字段含义](tables.md)
- [FA Setup](setup.md)
- [Depreciation/Accounting](depreciation-accounting.md)
