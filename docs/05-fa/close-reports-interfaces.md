# FA 月结、报表、Mass Additions 与排错

> `FA_MASS_ADDITIONS`、遗留资产迁移、Prepare/Post 和资产对账代码见 [FA 接口实现案例](interfaces.md)。

## Mass Additions

```text
AP/Projects/External Source
→ FA_MASS_ADDITIONS
→ Prepare Mass Additions
→ Review/Merge/Split/Assign Category
→ Post Mass Additions
→ Asset/Book/Distribution
```

`POSTING_STATUS` 表示 New/On Hold/Posted/Delete/Error 等处理状态（具体 lookup 以实例为准）。一条 AP 分配是否进入 FA 取决于 Asset Tracking/Category/Account、Transfer to GL/FA 和接口程序。

## 月结

1. 完成 Mass Additions、CIP Capitalization、Adjustments/Transfers/Retirements。
2. 运行并复核 Depreciation，处理异常资产和未完交易。
3. 运行 Create Accounting Final、Transfer to GL、Journal Import/Post。
4. 对账 Asset Cost、CIP、Reserve、Depreciation Expense、Retirement Gain/Loss、Clearing。
5. 运行 Asset Register、Reserve Ledger、Cost Detail、CIP Detail、Retirement 和 Account Reconciliation 报表，关闭 FA 期间。

## SQL

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

## 排查

- AP 行未进 FA：查 Track as Asset、Asset Clearing Account、AP 会计/转 GL、Create Mass Additions 参数和已转标志。
- Mass Addition 不能 Post：查 Category/Book、DPIS、Cost、Units、Location/Employee/Expense Account、Posting Status/Error。
- FA/GL 不平：区分未会计、未转/未过账、GL 手工分录、日期错位和 Asset Category 账户变更。
- 期间无法关闭：检查 Depreciation Run、Pending Transactions、Mass Additions、Accounting 和当期报表。

## 关联

- [FA Process](process.md)
- [Projects to Assets](../08-e2e/projects-assets.md)

## 官方参考

- [Oracle E-Business Suite R12.2 Documentation Library](https://docs.oracle.com/cd/E26401_01/index.htm)
