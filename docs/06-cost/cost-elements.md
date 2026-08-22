# 物料、资源、间接费与成本更新

## 成本构成

```text
Assembly Item Cost
= Material（Components）
+ Material Overhead（采购/收货/物料附加）
+ Resource（Routing Resource Usage × Rate）
+ Outside Processing（外协工序）
+ Overhead（Resource/Unit/Activity Basis）
```

Cost Rollup 从 BOM/Routing 底层向上计算，受 Alternate BOM/Routing、Yield/Scrap、Basis Type、Lot Size、Resource Units/Rate、Overhead Association 影响。成本更新前应冻结 BOM/Routing 版本截止点并保留 Rollup 输出。

## SQL

```sql
SELECT cicd.inventory_item_id, cicd.organization_id,
       cicd.cost_type_id, cicd.cost_element_id,
       cicd.level_type, cicd.resource_id,
       cicd.item_cost, cicd.basis_type,
       cicd.basis_factor, cicd.net_yield_or_shrinkage_factor
  FROM cst_item_cost_details cicd
 WHERE cicd.organization_id = :p_organization_id
   AND cicd.inventory_item_id = :p_inventory_item_id
   AND cicd.cost_type_id = :p_cost_type_id
 ORDER BY cicd.cost_element_id, cicd.level_type, cicd.resource_id;

SELECT resource_id, resource_code, description,
       cost_element_id, disable_date
  FROM bom_resources
 WHERE organization_id = :p_organization_id
 ORDER BY resource_code;
```

## 排查

- Rollup 漏组件：查 BOM Effectivity、Alternate、Include in Cost Rollup、Phantom、Supply Type、Yield 和组件成本。
- Resource Cost 为零：查 Routing Operation/Resource Usage、Costed Flag、Resource Rate/Cost Type、UOM 和基准。
- Overhead 未吸收：查 Department/Resource Association、Basis Type/Rate、Activity、交易是否触发。
- Update 产生过大差异：对比 Pending/Frozen 的 Element/Level Detail，分离 BOM、Rate、Yield、Lot Size 和手工成本变更。

## 关联

- [Costing Methods](costing-methods.md)
- [Cost Accounting Flow](accounting-flow.md)
