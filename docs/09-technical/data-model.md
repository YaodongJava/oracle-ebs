# EBS R12.2 数据模型与常用表

## 建模约定

- `_ALL` 通常包含 OU 级数据并带 `ORG_ID`；`_B` 为基表，`_TL` 为翻译，`_VL` 常为基表+当前语言视图，`_F` 常为 DateTrack，`_INTERFACE` 为接口，`_TEMP/_GT` 可为临时数据。
- WHO 列：`CREATION_DATE/CREATED_BY/LAST_UPDATE_DATE/LAST_UPDATED_BY/LAST_UPDATE_LOGIN`。
- ID 应作为关联键，业务编号用于显示；多 OU 关联还要核对 `ORG_ID`。
- 历史表通常使用 Effective/Ineffective Date 或 Current Flag，不能在无日期条件时当作当前值。
- APPS 通过 Synonym/View/Package 访问产品 Schema。R12.2 定制对象必须遵循 Editioning/Online Patching 标准。

## 模块速查

| 模块 | 主要对象 |
| --- | --- |
| FND | `FND_USER`, `FND_RESPONSIBILITY_VL`, `FND_CONCURRENT_REQUESTS`, `FND_PROFILE_OPTION_VALUES` |
| HR/ORG | `HR_ALL_ORGANIZATION_UNITS`, `HR_OPERATING_UNITS`, `ORG_ORGANIZATION_DEFINITIONS` |
| TCA | `HZ_PARTIES`, `HZ_CUST_ACCOUNTS`, `HZ_PARTY_SITES`, `HZ_LOCATIONS` |
| GL | `GL_LEDGERS`, `GL_CODE_COMBINATIONS`, `GL_JE_*`, `GL_BALANCES`, `GL_INTERFACE` |
| XLA | `XLA_TRANSACTION_ENTITIES`, `XLA_EVENTS`, `XLA_AE_HEADERS`, `XLA_AE_LINES` |
| AP | `AP_SUPPLIERS`, `AP_INVOICES_ALL`, `AP_INVOICE_DISTRIBUTIONS_ALL`, `AP_CHECKS_ALL` |
| AR | `RA_CUSTOMER_TRX_ALL`, `AR_PAYMENT_SCHEDULES_ALL`, `AR_CASH_RECEIPTS_ALL` |
| PO/RCV | `PO_HEADERS_ALL`, `PO_DISTRIBUTIONS_ALL`, `RCV_TRANSACTIONS` |
| OM/WSH | `OE_ORDER_HEADERS_ALL`, `OE_ORDER_LINES_ALL`, `WSH_DELIVERY_DETAILS` |
| INV/CST/WIP | `MTL_SYSTEM_ITEMS_B`, `MTL_MATERIAL_TRANSACTIONS`, `CST_ITEM_COSTS`, `WIP_ENTITIES` |
| FA | `FA_ADDITIONS_B`, `FA_BOOKS`, `FA_DISTRIBUTION_HISTORY`, `FA_DEPRN_SUMMARY` |
| CE/Tax | `CE_BANK_ACCOUNTS`, `CE_STATEMENT_*`, `ZX_LINES` |

## 元数据 SQL

```sql
SELECT owner, object_name, object_type, status
  FROM all_objects
 WHERE object_name = UPPER(:p_object_name)
 ORDER BY owner, object_type;

SELECT owner, table_name, column_id, column_name,
       data_type, data_length, nullable
  FROM all_tab_columns
 WHERE table_name = UPPER(:p_object_name)
 ORDER BY owner, column_id;

SELECT owner, synonym_name, table_owner, table_name, db_link
  FROM all_synonyms
 WHERE synonym_name = UPPER(:p_object_name);

SELECT acc.owner, acc.constraint_name, acc.table_name,
       acc.column_name, ac.constraint_type,
       ac.r_owner, ac.r_constraint_name
  FROM all_cons_columns acc
  JOIN all_constraints ac
    ON ac.owner = acc.owner
   AND ac.constraint_name = acc.constraint_name
 WHERE acc.table_name = UPPER(:p_table_name)
 ORDER BY acc.constraint_name, acc.position;
```

## 原则与排错

- 先用 eTRM/`ALL_*` 确认对象、列、同义词和约束，不凭记忆跨版本写 SQL。
- 查询必须包含组织/账簿、主键或日期范围，对大表先看执行计划。
- 不直接 DML 基表；使用标准 Form/OAF、Open Interface 或 Published API，数据修复走 Oracle Support。
- 不使用 `SELECT *`、隐式日期转换和字符串拼 SQL；使用显式列、绑定变量和 ANSI Join。

## 关联

- [FND、Concurrent、Workflow 与运维常用表](tables.md)
- [Integration](integration.md)
- [Operations](operations.md)
