# FA 折旧、税务折旧、资产处置与会计

## 折旧原理

折旧由 Cost Basis、Method/Rate/Life、Salvage Value、Depreciation Ceiling、Prorate Convention、DPIS、Calendar 和已提折旧决定。Run Depreciation 对当期资产计算，关期后结果进入 SLA/GL。Rollback 仅在标准程序允许的未关闭场景使用。

处置可为 Full/Partial Retirement，根据 Proceeds、Cost of Removal、Net Book Value 计算 Gain/Loss；Reinstatement 撤销处置并重建折旧/会计影响。

## SQL

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

## 排查

- 资产未折旧：查 `DEPRECIATE_FLAG`、DPIS/Prorate Date、Asset Type、Cost、Method/Life、Fully Reserved/Retired 状态。
- 折旧金额不对：比较 Book/Method/Calendar、Cost Adjustments、Catch-up、Salvage/Ceiling、Bonus/Impairment 和舍入。
- Depreciation 请求失败：查日志中首个 Asset ID/Error，检查未完成 Mass Transaction、无效账户和并发冲突。
- Gain/Loss 异常：核对 Cost Retired、Reserve Retired、Proceeds/Removal、Retirement Convention 和处置日期。
- Tax Book 折旧差异：确认是政策差异而非 Mass Copy 遗漏，比较 Corporate/Tax Book 交易链。

## 关联

- [FA Transactions](asset-transactions.md)
- [FA Close/Interface](close-reports-interfaces.md)
