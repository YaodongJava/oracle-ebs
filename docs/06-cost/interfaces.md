# Oracle Inventory、WIP 与成本接口实现案例

## 1. 业界常用场景

| 场景 | 推荐接口 | 说明 |
| --- | --- | --- |
| WMS/3PL 入库、出库、调拨 | Inventory Transaction Open Interface | 写 `MTL_TRANSACTIONS_INTERFACE`，由 Transaction Manager 处理 |
| 批次/序列控制物料交易 | MTI + Lot/Serial Interface | 子表数量必须与主交易数量及物料控制属性一致 |
| MES 发料、退料、完工、报废 | WIP/Inventory 标准接口或公开 API | 按工单、工序、资源、组织分别验证 |
| 外部采购收货 | Receiving Open Interface | 使用 `RCV_HEADERS_INTERFACE`/`RCV_TRANSACTIONS_INTERFACE`，不要模拟库存 Misc Receipt |
| 盘点系统上传差异 | Physical/Cycle Count 标准接口 | 保留 Count Batch、Tag/Sequence、复点和审批状态 |
| 标准成本批量维护 | Item Cost Import/标准 Cost Update 流程 | 必须区分 Pending Cost、Cost Type 和组织成本法 |

## 2. Transaction Open Interface 状态模型

```text
WMS/MES
→ 自定义暂存（幂等、主数据校验）
→ MTL_TRANSACTIONS_INTERFACE
→ Transaction Manager/Worker
→ MTL_MATERIAL_TRANSACTIONS_TEMP
→ MTL_MATERIAL_TRANSACTIONS
→ Cost Manager
→ SLA/GL
```

常见接口初始值为 `PROCESS_FLAG=1`、`LOCK_FLAG=2`、`TRANSACTION_MODE=3`。这些值应按目标实例 eTRM 和标准样本核对；错误后优先通过 Transaction Open Interface 界面修正/重提，不直接批量改 Flag。

## 3. 导入前业务校验

```sql
SELECT msi.inventory_item_id,
       msi.segment1 item_number,
       msi.primary_uom_code,
       msi.inventory_item_status_code,
       msi.lot_control_code,
       msi.serial_number_control_code,
       msi.restrict_subinventories_code,
       msi.restrict_locators_code
  FROM mtl_system_items_b msi
 WHERE msi.organization_id = :p_organization_id
   AND msi.inventory_item_id = :p_inventory_item_id;

SELECT msi.secondary_inventory_name,
       msi.disable_date,
       msi.asset_inventory,
       msi.quantity_tracked
  FROM mtl_secondary_inventories msi
 WHERE msi.organization_id = :p_organization_id
   AND msi.secondary_inventory_name = :p_subinventory_code;

SELECT oap.organization_id,
       oap.open_flag,
       oap.period_name,
       oap.period_start_date,
       oap.schedule_close_date
  FROM org_acct_periods oap
 WHERE oap.organization_id = :p_organization_id
   AND :p_transaction_date BETWEEN oap.period_start_date
                               AND oap.schedule_close_date;
```

还应校验 UOM 转换、Locator、Lot/Serial、负库存策略、Transaction Type/Source、账户和项目制造属性。

## 4. 非批次物料的 Miscellaneous Receipt

```sql
DECLARE
  l_interface_id NUMBER := mtl_material_transactions_s.NEXTVAL;
  l_header_id    NUMBER := mtl_material_transactions_s.NEXTVAL;
  l_trx_type_id  NUMBER;
BEGIN
  SELECT transaction_type_id
    INTO l_trx_type_id
    FROM mtl_transaction_types
   WHERE transaction_type_name = 'Miscellaneous receipt';

  INSERT INTO mtl_transactions_interface (
    transaction_interface_id,
    transaction_header_id,
    source_code,
    source_header_id,
    source_line_id,
    process_flag,
    transaction_mode,
    lock_flag,
    organization_id,
    inventory_item_id,
    subinventory_code,
    locator_id,
    transaction_type_id,
    transaction_quantity,
    primary_quantity,
    transaction_uom,
    transaction_date,
    distribution_account_id,
    transaction_reference,
    last_update_date,
    last_updated_by,
    creation_date,
    created_by,
    last_update_login
  ) VALUES (
    l_interface_id,
    l_header_id,
    'XX_WMS',
    :p_external_header_id,
    :p_external_line_id,
    1,
    3,
    2,
    :p_organization_id,
    :p_inventory_item_id,
    :p_subinventory_code,
    :p_locator_id,
    l_trx_type_id,
    :p_transaction_quantity,
    :p_primary_quantity,
    :p_transaction_uom,
    :p_transaction_date,
    :p_distribution_ccid,
    :p_external_document_number,
    SYSDATE,
    fnd_global.user_id,
    SYSDATE,
    fnd_global.user_id,
    fnd_global.login_id
  );

  COMMIT;
  dbms_output.put_line('TRANSACTION_INTERFACE_ID=' || l_interface_id);
  dbms_output.put_line('TRANSACTION_HEADER_ID=' || l_header_id);
END;
/
```

正负号由 Transaction Type/Action 与业务方向共同决定。上线前必须用目标组织手工交易生成样本，确认数量方向、账户来源和 Cost Group；不要用负数量“猜测”所有出库场景。

## 5. 批次物料交易

在写入主接口后，使用相同 `TRANSACTION_INTERFACE_ID` 写 Lot Interface：

```sql
INSERT INTO mtl_transaction_lots_interface (
  transaction_interface_id,
  lot_number,
  transaction_quantity,
  primary_quantity,
  last_update_date,
  last_updated_by,
  creation_date,
  created_by
) VALUES (
  :p_transaction_interface_id,
  :p_lot_number,
  :p_transaction_quantity,
  :p_primary_quantity,
  SYSDATE,
  fnd_global.user_id,
  SYSDATE,
  fnd_global.user_id
);
```

序列控制物料还需 `MTL_SERIAL_NUMBERS_INTERFACE`。一批多 Lot 或连续 Serial 时，所有子行数量汇总必须与主接口一致；组织间调拨还要同时校验 From/To Organization、Intransit 和接收路线。

## 6. 处理、错误与重试

Transaction Manager 可按后台周期处理，也可从标准 Transaction Open Interface 页面启动 Process Transactions Interface。不要在未核对并发程序参数的情况下硬编码内部程序签名。

```sql
-- 接口层错误
SELECT transaction_interface_id,
       transaction_header_id,
       source_code,
       source_header_id,
       source_line_id,
       process_flag,
       lock_flag,
       error_code,
       error_explanation
  FROM mtl_transactions_interface
 WHERE source_code = 'XX_WMS'
   AND source_header_id = :p_external_header_id
 ORDER BY source_line_id;

-- Temp 层等待/错误
SELECT transaction_temp_id,
       transaction_header_id,
       process_flag,
       lock_flag,
       error_code,
       error_explanation
  FROM mtl_material_transactions_temp
 WHERE transaction_header_id = :p_transaction_header_id;

-- 成功业务交易
SELECT transaction_id,
       organization_id,
       inventory_item_id,
       transaction_quantity,
       primary_quantity,
       transaction_date,
       transaction_type_id,
       costed_flag,
       transaction_reference
  FROM mtl_material_transactions
 WHERE transaction_source_name = :p_external_document_number
    OR transaction_reference = :p_external_document_number
 ORDER BY transaction_id;
```

成功后的稳定追溯应将 `TRANSACTION_INTERFACE_ID → TRANSACTION_ID` 写入自定义映射表；字段回写行为随交易类型而异，不能只依赖 `TRANSACTION_REFERENCE`。

## 7. 幂等与并发实现

建议暂存表唯一键：

```sql
ALTER TABLE xxinv_txn_stg ADD CONSTRAINT xxinv_txn_stg_u1
  UNIQUE (source_system, external_header_id, external_line_id, action_code);
```

工作进程以 `FOR UPDATE SKIP LOCKED` 领取待处理行：

```sql
SELECT message_id
  FROM xxinv_txn_stg
 WHERE status = 'VALIDATED'
 ORDER BY message_id
 FOR UPDATE SKIP LOCKED;
```

每个源业务行只生成一个稳定幂等键。HTTP 超时、并发请求仍 Running 或 Transaction Manager 结果未知时，先查 Interface/Temp/Base 三层，不直接复制重放。

## 8. WIP/MES 实施边界

- 组件发料/退料可经标准 WIP/Inventory 交易处理，必须传工单、Operation Sequence 和 Supply Type 所需信息。
- 完工/退回要验证 Routing、Completion Subinventory/Locator、Lot/Serial 和 Backflush。
- 资源计费要验证 Department、Resource、UOM、实际/标准计费和时间。
- 工单状态、成本更新、关闭和差异计算使用标准流程；不直接写 `WIP_ENTITIES`、`WIP_DISCRETE_JOBS`、`WIP_TRANSACTIONS`。
- 如果 Integration Repository 提供匹配的公开 WIP API，以当前实例方法签名、消息栈和提交语义为准。

## 9. 常见问题

| 症状 | 常见原因 | 处理 |
| --- | --- | --- |
| `PROCESS_FLAG=3` | Item/Org/UOM/Subinventory/Locator/Lot/Serial/Account 无效 | 查 `ERROR_CODE/ERROR_EXPLANATION` 并用标准页面修正 |
| 长时间 Locked | Worker 失效、数据库锁、并发管理器异常 | 查请求、Session 和 Worker 日志，不直接改 Lock Flag |
| 已交易但未成本化 | Cost Manager、Item Cost、前置交易、期间或负库存问题 | 查 `MMT.COSTED_FLAG` 和 Cost Worker 日志 |
| Lot 数量错误 | Lot 子行合计与主行不一致 | 重建整笔消息，不只修改一张子表 |
| 重复出入库 | 超时后无查询即重放 | 暂存唯一约束 + 三层查询 + 成功 ID 映射 |

## 10. 关联文档

- [成本接口与排错](interfaces-troubleshooting.md)
- [库存/WIP/成本会计流](accounting-flow.md)
- [成本常用表](tables.md)
- [库存、WIP、成本到 GL](../08-e2e/inventory-wip-cost-gl.md)

## 11. 官方参考

- [Oracle Inventory User Guide: Transaction Open Interface](https://docs.oracle.com/cd/E26401_01/doc.122/e48820/T291651T292013.htm)
- [Oracle Inventory User Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48820/)
- [Oracle E-Business Suite eTRM User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/)
