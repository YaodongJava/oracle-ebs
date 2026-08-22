# 成本组织、成本类型、成本要素与成本组

## 核心模型

- Inventory Organization 是库存/制造交易边界，其 Costing Organization/Method 在库存组织参数中确定。
- Cost Type 是一套物料/资源/间接费成本表述；Frozen 常用于标准成本，Pending 或自定义类型用于模拟/更新。
- Cost Element 包括 Material、Material Overhead、Resource、Outside Processing、Overhead。
- Cost Group 在项目制造/WMS 等场景将同一组织库存按成本分区，不等于 Cost Type。
- Valuation Account 通常由 Organization/Subinventory/Cost Group/Item 和 SLA 共同决定。

## 设置顺序

1. 建立 Ledger/OU/Inventory Organization、物料主组织和会计信息。
2. 设置 Costing Method、Cost Organization、Transfer Detail、Negative Quantity 和账户。
3. 定义 Cost Types、Activities、Resources、Overheads、Departments/Resources 和 Absorption Rules。
4. 定义 Item Cost、Resource Rate、Overhead Rate，执行 Cost Rollup/Update 测试。
5. 测试 PO Receipt/Delivery、Misc/Transfer、WIP Issue/Completion、Sales Issue、Return 和月结。

## SQL

```sql
SELECT organization_id, organization_code, organization_name,
       operating_unit, set_of_books_id, master_organization_id,
       legal_entity, disable_date
  FROM org_organization_definitions
 WHERE organization_id = :p_organization_id;

SELECT cic.inventory_item_id, cic.organization_id,
       cic.cost_type_id, cct.cost_type,
       cic.material_cost, cic.material_overhead_cost,
       cic.resource_cost, cic.outside_processing_cost,
       cic.overhead_cost, cic.item_cost, cic.based_on_rollup_flag
  FROM cst_item_costs cic
  JOIN cst_cost_types cct ON cct.cost_type_id = cic.cost_type_id
 WHERE cic.organization_id = :p_organization_id
   AND cic.inventory_item_id = :p_inventory_item_id;
```

## 排查

- Item Cost 为零：检查 Cost Type、Buy/Make、BOM/Routing、Component Yield、Resource/Overhead Rate 和 Rollup 日志。
- 账户错：查 Organization/Subinventory/Item/Cost Group 默认及 SLA Account Derivation。
- 组织不能运行成本程序：查 Costing Method、Cost Organization Relationship、Period 和职责 Organization Access。
- 修改 Cost Method 需求：有交易后通常不能直接切换，应设计新组织/迁移方案并与 Oracle Support 确认。

## 关联

- [INV/CST/WIP 常用表结构与字段含义](tables.md)
- [Costing Methods](costing-methods.md)
- [Inventory/WIP/Cost/GL](../08-e2e/inventory-wip-cost-gl.md)
