# 标准成本、平均成本与周期成本

## 方法对比

| 方法 | 价值基础 | 主要差异 |
| --- | --- | --- |
| Standard | Frozen Standard Cost | Purchase Price、Invoice Price、Resource/Usage/Efficiency、Overhead 等差异 |
| Average | 交易后加权平均单价 | 接收/生产/调整改变平均成本，销售/发料通常按当前成本出库 |
| FIFO/LIFO | 成本层 | 按层消耗并保留层次 |
| Periodic | 期间货值/交易后计算 | 期末运行周期成本处理并生成差异/调整 |

## 关键原理

Standard Cost Update 将 Pending 成本更新到 Frozen，对现有库存/WIP 产生重估会计。Average Cost 受负库存、交易顺序和补录日期影响；补录交易可重算后续成本。周期成本是独立的期末计算链，不等于简单查当前 Item Cost。

## SQL

```sql
SELECT cct.cost_type_id, cct.cost_type, cct.description,
       cct.default_cost_type_id, cct.allow_updates_flag,
       cct.multi_org_flag, cct.disable_date
  FROM cst_cost_types cct
 ORDER BY cct.cost_type;

SELECT inventory_item_id, organization_id, cost_type_id,
       item_cost, unburdened_cost, burden_cost,
       material_cost, material_overhead_cost,
       resource_cost, outside_processing_cost, overhead_cost
  FROM cst_item_costs
 WHERE organization_id = :p_organization_id
   AND inventory_item_id = :p_inventory_item_id;

SELECT inventory_item_id, organization_id, transaction_id,
       layer_id, layer_quantity, item_cost
  FROM cst_quantity_layers
 WHERE organization_id = :p_organization_id
   AND inventory_item_id = :p_inventory_item_id
 ORDER BY layer_id;
```

## 排查

- Standard Update 异常：先运行不更新的模拟/报表，审核成本差异、库存/WIP 重估和账户后再执行。
- Average Cost 跳变：按 Transaction ID/Date 跟踪 Receipt、Cost Update、Negative Balance 恢复和补录交易。
- 负库存差异：查允许负库存设置、交易时间顺序、成本层和恢复入库价格。
- 期间成本不完整：查所有交易已 Cost、期间范围、未处理接口、资源/间接费和处理日志。

## 关联

- [Cost Setup](setup.md)
- [Period Close](period-close-reports.md)
