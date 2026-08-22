# FA 资产增加、调整、转移、重分类与盘点

## 交易类型

- Addition/CIP Addition：建立资产、Book 和 Distribution；CIP 通过 Capitalization 开始折旧。
- Cost/Book Adjustment：调整 Cost、Salvage、Life、Method、Rate、DPIS，可产生 Catch-up/Expensed Adjustment。
- Transfer：在 Employee/Expense Account/Location 间分配数量转移，总 Units 需平衡。
- Reclassification：改变 Category，可引起账户转移和折旧属性改变。
- Physical Inventory：将现场盘点与 FA Location/Employee 对比，差异审批后执行转移/处置。

## SQL

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

## 排查

- Transfer 不平：比较 Transfer Out/In Units，检查当前 Distribution 行、Location/Employee/CCID 有效性。
- Adjustment 不可做：查 Asset/Book Status、Period、已运行折旧、Retirement 和源交易限制。
- Reclass 后会计异常：比较新旧 Category Book 账户、交易日期和 SLA 行。
- Physical Inventory 差异太多：先统一 Asset Number/Tag/Location 映射与盘点截止日，再处理已退役/在途转移。

## 关联

- [FA Setup](setup.md)
- [Depreciation](depreciation-accounting.md)
