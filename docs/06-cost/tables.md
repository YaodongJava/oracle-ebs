# Inventory / Cost / WIP 常用表结构

## 业务说明

库存数量、成本价值和会计分录是三个不同层次：On-hand 回答“现在有多少”，Material Transaction 回答“数量如何变化”，Item Cost/Cost Layer 回答“单价如何得出”，Transaction Accounts/SLA 回答“如何入账”。WIP 还需将工单、发退料、资源、移动/完工和差异结合。

## 表级速查

| 表 | 中文名 | 粒度/用途 | 关键字段 |
| --- | --- | --- | --- |
| `MTL_SYSTEM_ITEMS_B` | 物料组织属性 | Item+Inventory Organization | `INVENTORY_ITEM_ID`, `ORGANIZATION_ID` |
| `MTL_PARAMETERS` | 库存组织参数 | 每个 IO | `ORGANIZATION_ID`, `MASTER_ORGANIZATION_ID`, Costing options |
| `MTL_ONHAND_QUANTITIES_DETAIL` | 现有量明细 | Item+Org+Subinventory+Locator+Lot+Receipt Layer | `ONHAND_QUANTITIES_ID`, `PRIMARY_TRANSACTION_QUANTITY` |
| `MTL_MATERIAL_TRANSACTIONS` | 库存物料交易 | 每笔已处理物料交易 | `TRANSACTION_ID`, `ORGANIZATION_ID`, `INVENTORY_ITEM_ID` |
| `MTL_TRANSACTION_ACCOUNTS` | 库存交易会计分布 | Transaction+会计行 | `INV_SUB_LEDGER_ID`, `TRANSACTION_ID`, `REFERENCE_ACCOUNT` |
| `MTL_TRANSACTIONS_INTERFACE` | 库存交易接口 | 待 Transaction Manager 处理 | `TRANSACTION_INTERFACE_ID`, `PROCESS_FLAG`, `ERROR_CODE` |
| `MTL_MATERIAL_TRANSACTIONS_TEMP` | 库存待处理交易 | Transaction Worker 工作层 | `TRANSACTION_TEMP_ID`, `TRANSACTION_HEADER_ID` |
| `CST_COST_TYPES` | 成本类型 | 每个 Cost Type | `COST_TYPE_ID`, `COST_TYPE`, `ALLOW_UPDATES_FLAG` |
| `CST_ITEM_COSTS` | 物料成本汇总 | Item+Org+Cost Type | `INVENTORY_ITEM_ID`, `ORGANIZATION_ID`, `COST_TYPE_ID` |
| `CST_ITEM_COST_DETAILS` | 物料成本明细 | Item+Cost Type+Element/Resource/Level | `COST_ELEMENT_ID`, `LEVEL_TYPE`, `ITEM_COST` |
| `CST_QUANTITY_LAYERS` | 数量/成本层 | Item+Org+Cost Group/Layer | `LAYER_ID`, `LAYER_QUANTITY`, `ITEM_COST` |
| `WIP_ENTITIES` | WIP 工单实体 | 每个 Job/Schedule Entity | `WIP_ENTITY_ID`, `WIP_ENTITY_NAME`, `ORGANIZATION_ID` |
| `WIP_DISCRETE_JOBS` | 离散工单 | 每个 Discrete Job | `WIP_ENTITY_ID`, `STATUS_TYPE`, `PRIMARY_ITEM_ID` |
| `WIP_TRANSACTIONS` | WIP 资源/成本交易 | 每笔 WIP Transaction | `TRANSACTION_ID`, `WIP_ENTITY_ID`, `TRANSACTION_TYPE` |
| `WIP_TRANSACTION_ACCOUNTS` | WIP 会计分布 | WIP Transaction+会计行 | `TRANSACTION_ID`, `ACCOUNTING_LINE_TYPE`, `REFERENCE_ACCOUNT` |
| `ORG_ACCT_PERIODS` | 库存会计期间 | Organization+Period | `ACCT_PERIOD_ID`, `OPEN_FLAG`, `PERIOD_CLOSE_DATE` |

## `MTL_MATERIAL_TRANSACTIONS` — 物料交易

| 字段 | 中文名 | 业务说明 |
| --- | --- | --- |
| `TRANSACTION_ID` | 库存交易 ID | 连接 Lot/Serial、Cost Distribution、源单据的主线 |
| `TRANSACTION_TYPE_ID` | 交易类型 | 关联 `MTL_TRANSACTION_TYPES`，如 PO Receipt、Misc Issue、Sales Order Issue |
| `TRANSACTION_ACTION_ID` | 交易动作 | Issue/Receipt/Transfer/Cost Update 等底层动作 |
| `TRANSACTION_SOURCE_TYPE_ID` | 源类型 | PO、Sales Order、WIP Job、Account 等来源大类 |
| `TRANSACTION_SOURCE_ID` | 源单据 ID | 含义随 Source Type 改变，不能统一关联同一张表 |
| `TRANSACTION_QUANTITY` | 交易 UOM 数量 | UOM 见 `TRANSACTION_UOM` |
| `PRIMARY_QUANTITY` | 主单位数量 | On-hand/跨交易对账常用主 UOM |
| `SUBINVENTORY_CODE/LOCATOR_ID` | 子库/库位 | Transfer 交易还需检查 Transfer Subinventory/Locator |
| `COSTED_FLAG` | 成本处理状态 | 常见已成本/未成本/错误/不需成本等代码，必须结合 `ERROR_CODE/ERROR_EXPLANATION` 和 Cost Manager 日志 |
| `ACTUAL_COST` | 实际单位成本 | 对不同 Costing Method/交易类型含义不同，不能直接当标准成本 |

## `MTL_TRANSACTIONS_INTERFACE` 处理字段

| 字段 | 中文名 | 说明 |
| --- | --- | --- |
| `PROCESS_FLAG` | 处理状态 | 常见 `1`等待、`2`运行中、`3`错误；以 Inventory eTRM/Manager 逻辑为准 |
| `LOCK_FLAG` | 锁定状态 | 工作器选中与释放控制，不应手工改 Flag 解锁 |
| `TRANSACTION_MODE` | 处理模式 | Online/Concurrent/Background 等方式，具体代码用 Inventory Lookup 解码 |
| `SOURCE_CODE/HEADER_ID/LINE_ID` | 外部源键 | 应组成幂等键，避免超时重试造成重复交易 |
| `ERROR_CODE/ERROR_EXPLANATION` | 错误代码/说明 | 修正上游数据后按标准 Resubmit 流程重试 |

## 成本字段

### `CST_ITEM_COST_DETAILS`

| 字段 | 中文名 | 常见值/业务含义 |
| --- | --- | --- |
| `COST_ELEMENT_ID` | 成本要素 | 标准要素通常为 Material、Material Overhead、Resource、Outside Processing、Overhead；用 `CST_COST_ELEMENTS` 解码 |
| `LEVEL_TYPE` | 本层/下层 | 区分 This Level 与 Previous Level，内部代码以 Cost Lookup 为准 |
| `RESOURCE_ID` | 子要素/资源 | Material Subelement、Resource、Overhead 的具体来源 |
| `BASIS_TYPE` | 计费基础 | Item/Lot/Resource Units/Resource Value/Total Value/Activity 等，按 Cost Element 与设置解码 |
| `ITEM_COST` | 成本明细金额 | 汇总后与 `CST_ITEM_COSTS` 要素成本对账，注意 Yield/Shrinkage/Basis Factor |

## WIP 工单状态

`WIP_DISCRETE_JOBS.STATUS_TYPE` 是数字 Lookup，常见业务含义包括 Unreleased、Released、Complete、Complete-No Charges、Closed、Cancelled、On Hold。不应在报表中手写不完整 `DECODE`，应关联 WIP Job Status Lookup；Complete 不等于 Closed，只有 Close 后才会按规则识别/结转差异。

## 官方参考

- [Oracle Cost Management User's Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48829/)
- [Oracle Inventory User's Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48820/)
- [Oracle E-Business Suite eTRM User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/)
