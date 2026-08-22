# 成本分配、差异、结转、期间关闭与报表

## 关期顺序

1. 停止当期补录，处理 Inventory/Receiving/WIP/Cost Interface 待处理和错误交易。
2. 确保 Material/Resource/Overhead/Receiving 交易已 Cost 并创建会计。
3. 处理 WIP Jobs：发料、完工、差异、Close；处理 COGS Recognition。
4. 对账 Inventory Valuation、Receiving Accrual、WIP Valuation、COGS、差异和 GL。
5. 运行 Period Close Reconciliation/估值/交易分布报表，再关闭 Inventory Period。

## SQL

```sql
SELECT oap.organization_id, oap.acct_period_id,
       oap.period_name, oap.period_start_date,
       oap.schedule_close_date, oap.period_close_date,
       oap.open_flag, oap.summarized_flag
  FROM org_acct_periods oap
 WHERE oap.organization_id = :p_organization_id
 ORDER BY oap.period_start_date DESC;

SELECT costed_flag, transaction_source_type_id,
       transaction_action_id, COUNT(*) cnt
  FROM mtl_material_transactions
 WHERE organization_id = :p_organization_id
   AND transaction_date BETWEEN :p_start_date AND :p_end_date
 GROUP BY costed_flag, transaction_source_type_id,
          transaction_action_id;

SELECT accounting_line_type, reference_account,
       SUM(base_transaction_value) amount
  FROM mtl_transaction_accounts
 WHERE organization_id = :p_organization_id
   AND transaction_date BETWEEN :p_start_date AND :p_end_date
 GROUP BY accounting_line_type, reference_account;
```

## 排查

- Period Close 不允许：运行 Pending Transactions 检查，分别处理 MTL Transactions Interface、Pending Material、WIP Move/Cost、Receiving 和未会计交易。
- Inventory/GL 不平：统一组织、截止时间、Cost Group/Subinventory、Currency，分析未转 GL、手工 GL、补录和负库存。
- 估值报表负数：按 Item/Subinventory/Locator/Cost Group 查 On-hand、Pending Transaction 和负库存原因。
- 关期后发现遗漏：不更新 Period/Table；评估标准 Reopen 可行性或在下期用可审计调整处理。

## 关联

- [Cost Accounting Flow](accounting-flow.md)
- [GL Close](../04-gl/close-reports.md)
