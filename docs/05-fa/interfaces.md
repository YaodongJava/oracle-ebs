# Oracle Assets 接口实现案例

## 1. 业界常用场景

| 场景 | 推荐接口 | 业务说明 |
| --- | --- | --- |
| AP 采购发票资本化 | Create Mass Additions | AP 已核算资产行进入 FA 待处理队列，保留 Invoice/PO 追溯 |
| Projects CIP 转固 | PRC: Interface Assets + Mass Additions | 项目资产线按 Project/Task/Asset Line 追溯 |
| 遗留资产迁移 | `FA_MASS_ADDITIONS` + Prepare/Post Mass Additions | 按 Book 分批导入成本、累计折旧和启用日 |
| 租赁/资产管理系统新增资产 | 自定义暂存层 + `FA_MASS_ADDITIONS` | 先做类别、地点、责任人、成本账户校验 |
| 大批量资产调整/转移/处置 | Oracle Assets 公共 API/标准批处理 | 以当前 Integration Repository/API 文档签名为准，不直接改 FA 历史表 |

## 2. Mass Additions 业务状态

外部来源通常只创建 `NEW` 待处理行，由资产会计在 Mass Additions Workbench 完善并置为可过账，再运行 Post Mass Additions。典型过程如下：

```text
Source/Staging → NEW → Review/Prepare → POST → Posted Asset
                       ↘ HOLD / MERGED / SPLIT / ERROR
```

状态代码、Queue 名称和允许转换必须以目标实例 FA Lookup 和标准界面为准。不要通过 `UPDATE FA_MASS_ADDITIONS` 人工推动状态。

## 3. 导入前校验

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

## 4. `FA_MASS_ADDITIONS` 具体实现

### 4.1 外部资产新增

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

### 4.2 运行前核对目标列

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

## 5. 遗留资产迁移的成本和累计折旧

遗留迁移不只是插入当前成本。至少要确认：

- Corporate/Tax Book 的启用日、原始成本、当前成本和净残值；
- 折旧方法、年限、Prorate Convention 和累计折旧；
- 本年累计折旧（YTD）和迁移期间；
- 当前地点、责任人、单位数和折旧费用账户；
- 资产类别默认值是否允许被源数据覆盖。

先用少量样本在关闭的测试环境走完 Prepare/Post/Depreciation，再核对剩余价值和下一期折旧。不要用 DML 直接补 `FA_BOOKS`、`FA_DEPRN_SUMMARY` 或 `FA_DISTRIBUTION_HISTORY`。

## 6. Mass Additions 处理与监控

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

## 7. 成功结果对账

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

## 8. 常见问题与实现方法

| 问题 | 常见原因 | 排查/处理 |
| --- | --- | --- |
| 行一直停留在 NEW | 类别、Book、启用日、地点或账户未准备 | 用 Workbench/Prepare Mass Additions 查看错误，不手工改状态 |
| 无法 Post | 期间、类别账簿设置、资产号、单位数或成本无效 | 校验 FA 当前期间和 Category Book Defaults |
| 重复资产 | 源消息重放、缺少幂等键 | 暂存表对 `SOURCE_SYSTEM + EXTERNAL_ASSET_ID + BOOK` 建唯一约束 |
| 折旧金额不符 | 方法、年限、Prorate、YTD/Reserve 映射错误 | 用标准资产样本模拟下一期折旧后再批量迁移 |
| 地点/员工无法选 | KFF/HR 有效期或 Security Profile 不匹配 | 按启用日查询有效记录，并核对职责权限 |

## 9. 关联文档

- [FA 增加、调整、转移与处置](asset-transactions.md)
- [FA 折旧与会计](depreciation-accounting.md)
- [FA 常用表](tables.md)
- [项目与资产资本化](../08-e2e/projects-assets.md)

## 10. 官方参考

- [Oracle Assets User Guide: Mass Additions](https://docs.oracle.com/cd/E26401_01/doc.122/e48755/T293142T293157.htm)
- [Oracle Assets User Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48755/)
- [Oracle E-Business Suite eTRM User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/)
