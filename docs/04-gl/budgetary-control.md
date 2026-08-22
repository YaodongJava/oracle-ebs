# 预算、预算控制与资金可用性

## 概念

GL Budget 保存预算余额；Encumbrance 表示承诺/义务；Budgetary Control/Funds Check 按预算组织、账户范围、期间和边界判断可用资金。在不同实施中，可能使用传统 GL Encumbrance/Budgetary Control 或公共部门相关功能，须以已安装产品为准。

```text
Budget - Actual - Encumbrance = Funds Available
```

## 配置

1. 定义 Budget、Budget Organization、Account Ranges、Calendar/Periods。
2. 定义 Encumbrance Types 和采购阶段（Requisition/PO/Invoice）会计。
3. 定义 Funds Check Level（None/Advisory/Absolute）、Tolerance、Boundary 和 Override Authority。
4. 加载/过账预算，测试预算转移、补充、采购取消/反冲和期末结转。

## SQL

```sql
SELECT gb.budget_name, gb.status, gb.first_valid_period_name,
       gb.last_valid_period_name
  FROM gl_budgets gb
 WHERE gb.budget_name = :p_budget_name;

SELECT gb.ledger_id, gb.code_combination_id, gb.currency_code,
       gb.period_name, gb.actual_flag, gb.encumbrance_type_id,
       gb.period_net_dr, gb.period_net_cr,
       gb.begin_balance_dr, gb.begin_balance_cr
  FROM gl_balances gb
 WHERE gb.ledger_id = :p_ledger_id
   AND gb.period_name = :p_period_name
   AND gb.code_combination_id = :p_ccid
   AND gb.actual_flag IN ('A','B','E');
```

## 排查

- Funds Check 失败：检查预算组织账户范围、Budget Period/Amount、Actual/Encumbrance、Boundary、Currency 和控制级别。
- 可用金额不对：区分 Requisition/PO/Invoice 保留、未过账 Journal、取消/退货释放和期间跨度。
- Override 不可用：查用户限额/权限、Funds Check 级别、单据状态和审批链。
- 预算导入错：检查 Budget Name、Ledger/CCID、Currency、Period、Debit/Credit 方向和接口错误代码。

## 关联

- [COA](../01-common/coa.md)
- [P2P](../08-e2e/procure-to-pay.md)
