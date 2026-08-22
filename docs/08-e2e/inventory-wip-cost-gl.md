# 库存、WIP、成本与 GL 衔接

## 主键与断点

```text
Inventory/WIP Source Transaction
→ MTL_MATERIAL_TRANSACTIONS / WIP_TRANSACTIONS
→ Cost Processor
→ MTL_TRANSACTION_ACCOUNTS / WIP_TRANSACTION_ACCOUNTS
→ SLA Event/AE Lines
→ GL Import References / Journal / Balance
```

Inventory Transaction ID 是物料交易与成本分录的主线；WIP Entity ID + Transaction ID 追踪工单发料、资源、完工和差异。需区分业务交易已处理、Costed、Accounted、Transferred、Imported、Posted 六个状态。

## SQL

```sql
SELECT mmt.transaction_id, mmt.organization_id,
       mmt.inventory_item_id, mmt.transaction_date,
       mmt.transaction_quantity, mmt.costed_flag,
       mmt.transaction_source_type_id,
       mmt.transaction_source_id, mmt.trx_source_line_id,
       mta.accounting_line_type, mta.reference_account,
       mta.base_transaction_value, mta.gl_batch_id
  FROM mtl_material_transactions mmt
  LEFT JOIN mtl_transaction_accounts mta
    ON mta.transaction_id = mmt.transaction_id
 WHERE mmt.transaction_id = :p_transaction_id
 ORDER BY mta.inv_sub_ledger_id;

SELECT wta.wip_entity_id, wta.transaction_id,
       wta.accounting_line_type, wta.base_transaction_value,
       wta.reference_account, wta.gl_batch_id
  FROM wip_transaction_accounts wta
 WHERE wta.wip_entity_id = :p_wip_entity_id
 ORDER BY wta.transaction_id, wta.accounting_line_type;
```

## 对账框架

- Quantity：On-hand = 期初 + Receipts + Completions + Transfers In - Issues - Sales - Transfers Out ± Adjustments。
- Value：按 Organization/Cost Group/Subinventory/Item 比较估值报表与成本分录。
- WIP：期初 WIP + Material/Resource/OSP/Overhead - Completion/Return - Variance = 期末 WIP。
- GL：成本子账 + SLA 未转 + GL 未过账 + GL 手工调整 = GL 相关账户余额。

## 排错

- 有数量无价值：查 Item Cost、Costed Flag、Cost Manager、Transaction Date/Period、负库存。
- WIP 账户不平：查工单状态、发退料/完工/退库、Resource、Scrap、Close/Variance 是否在同一期。
- Cost/GL 差异：按 `GL_BATCH_ID`/SLA Link 追踪，检查未转/未过账、Period Cutoff、Account Mapping 和 Manual Journal。
- 性能：对 MMT/MTA 查询必须限定 Organization+Date/Transaction ID，避免生产全表汇总。

## 关联

- [Cost Methods](../06-cost/costing-methods.md)
- [Cost Close](../06-cost/period-close-reports.md)
