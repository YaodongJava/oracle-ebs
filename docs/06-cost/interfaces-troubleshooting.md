# 成本接口、Transaction Processor 与排错

> `MTL_TRANSACTIONS_INTERFACE`、Lot Interface、幂等和 Transaction Manager 处理代码见 [库存/WIP/成本接口实现案例](interfaces.md)。

## 处理器链路

```text
MTL_TRANSACTIONS_INTERFACE
 → Transaction Manager/Worker
 → MTL_MATERIAL_TRANSACTIONS_TEMP
 → Material Transaction
 → Cost Manager/Cost Worker
 → MTL_TRANSACTION_ACCOUNTS / SLA / GL
```

WIP Move/Cost、Receiving 和 Resource 还有各自接口/待处理表。排错应先确定卡在“导入、业务处理、成本计算、SLA、GL”哪一层。

## SQL

```sql
SELECT transaction_interface_id, source_code, source_header_id,
       source_line_id, process_flag, transaction_mode,
       lock_flag, error_code, error_explanation,
       organization_id, inventory_item_id,
       transaction_quantity, transaction_uom,
       transaction_date, transaction_type_id
  FROM mtl_transactions_interface
 WHERE organization_id = :p_organization_id
 ORDER BY transaction_interface_id;

SELECT transaction_temp_id, transaction_header_id,
       process_flag, lock_flag, organization_id,
       inventory_item_id, transaction_quantity,
       transaction_date, transaction_type_id
  FROM mtl_material_transactions_temp
 WHERE organization_id = :p_organization_id
 ORDER BY transaction_temp_id;

SELECT transaction_id, costed_flag, error_code, error_explanation
  FROM mtl_material_transactions
 WHERE organization_id = :p_organization_id
   AND NVL(costed_flag, 'N') <> 'Y'
 ORDER BY transaction_date, transaction_id;
```

## 排错

- `PROCESS_FLAG=3`/错误：根据 Error Code/Explanation 检查 Item/Org、UOM、Subinventory/Locator、Lot/Serial、Account、Date/Period。
- 长期 `PROCESS_FLAG=1/LOCK_FLAG=2`：检查 Transaction Manager/Worker 是否运行、并发冲突、失效 Worker 和数据库锁，不直接改 Flag。
- 交易已入 MMT 但未 Cost：检查 Cost Manager、Item Cost、前置 Transaction、期间、负库存和 Cost Worker 日志。
- 重复交易：使用 Source Code/Header/Line 幂等键，导入前同时查 Interface/Temp/Base 三层。
- Lot/Serial 错：接口头与 Lot/Serial Interface 子表的 Transaction Interface ID、数量和控制级别必须一致。

## 关联

- [Inventory Transactions](accounting-flow.md)
- [Concurrent Programs](../09-technical/concurrent-programs.md)

## 官方参考

- [Oracle Inventory User's Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48820/)
