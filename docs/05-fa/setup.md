# FA 资产账簿、类别、位置与关键配置

## 核心设置

- **Asset Book**：Corporate/Tax，关联 Ledger、Calendar、Prorate Calendar、Deprn Calendar、SLA 和账户规则。
- **Category**：Major/Minor 弹性域组合，按 Book 默认 Asset Cost、Reserve、Expense、CIP、Clearing、Gain/Loss 账户。
- **Location**：Location KFF 用于实物位置与盘点，与 HR Location/Inventory Locator 不是同一对象。
- **Depreciation Method**：Straight Line/Table/Calculated/Production 等，结合 Life、Rate、Prorate Convention、Salvage/Ceiling。
- **Key Flexfields**：Category、Location、Asset Key；分配行还使用 GL Expense CCID 和 Employee。

## 实施顺序

1. 定义 FA Calendar/Prorate Calendar、Fiscal Year、Methods/Conventions。
2. 定义 Category/Location/Asset Key KFF 及值，编译后创建 Category。
3. 定义 Corporate Book，分配 Category 账户和折旧默认。
4. 定义 Tax Book 与复制规则，配置 System Controls、Security by Book、Mass Additions、SLA。
5. 使用少量资产测试增加、折旧、调整、转移、处置和会计。

## SQL

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

## 排查

- Category 不可选：查 KFF 组合/值有效性、Category Enabled/Date、Book Assignment。
- 默认账户不对：查 Category Book 账户、Asset Type（Capitalized/CIP/Expense）和 SLA 覆盖。
- Tax Book 无数据：查 Corporate Book 关联、Initial/Mass Copy 参数、交易类型可复制性和请求日志。

## 关联

- [COA](../01-common/coa.md)
- [FA Process](process.md)
