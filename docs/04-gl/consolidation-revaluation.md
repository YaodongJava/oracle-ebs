# 合并、重估、折算与重复日记账

## 原理

- **Revaluation**：将外币账户余额按期末汇率重新计量，差额进入未实现损益。
- **Translation**：将 Ledger 余额从本位币折算为报告币种，资产负债/损益通常使用不同汇率规则。
- **Consolidation**：通过账户映射、数据转移和抵销将多账簿汇总；也可根据架构使用 Ledger Set/Secondary Ledger/Reporting 工具。
- **Recurring Journal/MassAllocation**：定期生成固定、公式或分配分录，生成后仍需审批与过账。

## 配置与执行

1. 定义 Period/Balance 汇率，定义 Revaluation 账户范围、Rate Type、Unrealized Gain/Loss。
2. 为 Translation 配置 Cumulative Translation Adjustment 账户和历史汇率。
3. 为 Consolidation 定义 COA Mapping、币种处理、子/母账簿期间映射、抵销与少数股东。
4. 对 Recurring/Allocation 使用独立 Source/Category，审查公式、统计量、分配基础和反冲。

## SQL

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

## 排查

- Revaluation 遗漏账户：查账户范围、Currency、Balance、Period Rate 和余额是否已为零。
- Translation 不平：查 Historical/Average/Period-end Rate、CTA Account、期间顺序和当期日记账是否全部过账。
- Consolidation 差异：查 COA Mapping、期间/币种、增量/全量转移、重复转移和抵销分录。
- Allocation 异常：保存生成批次，输出分配基础与公式中间值，不直接修改已过账行。

## 关联

- [Currency/Rate](../01-common/calendar-currency-period.md)
- [GL 结账](close-reports.md)
