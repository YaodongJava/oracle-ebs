# 收货、库存、WIP 与销售成本会计流

## 事件链

```text
PO Receipt/Delivery/Return → Receiving/Inventory Accounting
Inventory Issue/Receipt/Transfer → Material Accounting
WIP Issue/Resource/Completion/Close → WIP Accounting/Variances
OM Ship Confirm → Inventory Issue → COGS Recognition
→ SLA → GL
```

典型标准成本分录（实际以 SLA/设置为准）：收货借 Receiving Inspection/贷 Accrual；Delivery 借 Inventory/贷 Receiving Inspection；领料借 WIP Valuation/贷 Inventory；完工借 Inventory/贷 WIP；销售出库借 Deferred COGS/贷 Inventory，按收入确认比例转至 COGS。

## SQL

```sql
SELECT mmt.transaction_id, mmt.organization_id,
       mmt.inventory_item_id, mmt.transaction_date,
       mmt.transaction_type_id, mmt.transaction_action_id,
       mmt.transaction_source_type_id, mmt.transaction_source_id,
       mmt.transaction_quantity, mmt.primary_quantity,
       mmt.actual_cost, mmt.costed_flag
  FROM mtl_material_transactions mmt
 WHERE mmt.transaction_id = :p_transaction_id;

SELECT mta.transaction_id, mta.accounting_line_type,
       mta.reference_account, mta.base_transaction_value,
       mta.primary_quantity, mta.rate_or_amount,
       mta.gl_batch_id
  FROM mtl_transaction_accounts mta
 WHERE mta.transaction_id = :p_transaction_id
 ORDER BY mta.inv_sub_ledger_id;

SELECT wt.transaction_id, wt.wip_entity_id, wt.organization_id,
       wt.transaction_type, wt.transaction_date,
       wt.primary_quantity, wt.actual_resource_rate
  FROM wip_transactions wt
 WHERE wt.wip_entity_id = :p_wip_entity_id
 ORDER BY wt.transaction_date, wt.transaction_id;
```

## 排查

- Material Transaction 未 Cost：查 `COSTED_FLAG`、Error Code/Explanation、Item Cost、Period、账户、前置交易和 Cost Manager。
- Receipt/AP Accrual 不平：按 PO Distribution/Receipt Transaction/Invoice Distribution 对比数量、价格、汇率、退货/更正和截止日。
- WIP Variance 异常：检查发料/退料、Resource Usage/Rate、Completion/Scrap、Standard Update 时间和 Job Close。
- COGS 未确认：跟踪 OM Line/Delivery/Material Transaction、AR Invoice/Revenue、COGS Recognition 请求和会计期间。

## 关联

- [Inventory/WIP/Cost/GL E2E](../08-e2e/inventory-wip-cost-gl.md)
- [P2P](../08-e2e/procure-to-pay.md)
