# Cash Management / Payments / EBTax 常用表结构

## 业务说明

R12 银行与分行是 TCA Party，内部银行账户在 CE，账户所有者与 OU/应用用途决定 AP/AR 可见性，支付指令/文件在 IBY，银行对账单在 CE。EBTax 以 Application+Entity+Event Class+Transaction/Line 标识税务交易，不能仅按 `TRX_ID` 跨模块查税行。

## 表级速查

| 表 | 中文名 | 粒度/用途 | 关键字段 |
| --- | --- | --- | --- |
| `CE_BANK_ACCOUNTS` | 内部银行账户 | 每个银行账户 | `BANK_ACCOUNT_ID`, `BANK_ACCOUNT_NUM`, `CURRENCY_CODE` |
| `CE_BANK_ACCT_USES_ALL` | 银行账户 OU 用途 | Account+OU+Application Use | `BANK_ACCT_USE_ID`, `BANK_ACCOUNT_ID`, `ORG_ID` |
| `CE_STATEMENT_HEADERS` | 银行对账单头 | Account+Statement | `STATEMENT_HEADER_ID`, `BANK_ACCOUNT_ID`, `STATEMENT_NUMBER` |
| `CE_STATEMENT_LINES` | 银行对账单行 | 每条银行流水 | `STATEMENT_LINE_ID`, `TRX_TYPE`, `TRX_CODE`, `STATUS` |
| `CE_STATEMENT_HEADERS_INT` | 银行对账单头接口 | 待导入 Statement | 银行账户/对账单标识与控制金额 |
| `CE_STATEMENT_LINES_INTERFACE` | 银行对账单行接口 | 待导入流水 | 交易编码、日期、金额、参考 |
| `IBY_PAY_SERVICE_REQUESTS` | 付款服务请求/PPR | 每个 Payment Process Request | `PAYMENT_SERVICE_REQUEST_ID`, `CALL_APP_PAY_SERVICE_REQ_CODE`, `PAYMENT_SERVICE_REQUEST_STATUS` |
| `IBY_PAY_INSTRUCTIONS_ALL` | 付款指令 | 每个银行/格式/支付分组 | `PAYMENT_INSTRUCTION_ID`, `PAYMENT_STATUS` |
| `IBY_PAYMENTS_ALL` | IBY 付款 | 每笔支付 | `PAYMENT_ID`, `PAYMENT_INSTRUCTION_ID`, `PAYMENT_STATUS` |
| `IBY_EXT_BANK_ACCOUNTS` | 外部收款人银行账户 | 每个外部账户 | `EXT_BANK_ACCOUNT_ID`, Country/Bank/Branch/Account |
| `ZX_LINES` | EBTax 税行 | 源交易行的每个税 | `TAX_LINE_ID`, `APPLICATION_ID`, `TRX_ID`, `TRX_LINE_ID` |
| `ZX_LINES_DET_FACTORS` | 税确定因素 | 交易/行的税务输入快照 | Entity/Event/Trx/Line 组合 |
| `ZX_RATES_B` | 税率 | Regime+Tax+Status+Rate+有效期 | `TAX_RATE_ID`, `TAX_RATE_CODE`, `PERCENTAGE_RATE` |
| `ZX_PARTY_TAX_PROFILE` | 交易方税务档案 | Party/Party Site/LE/OU 税务属性 | `PARTY_TAX_PROFILE_ID`, `PARTY_ID`, `PARTY_TYPE_CODE` |
| `ZX_REGISTRATIONS` | 税务登记 | Party Tax Profile+Regime/Tax/Jurisdiction | `REGISTRATION_ID`, `REGISTRATION_NUMBER`, Effective Dates |

## CE 银行账户与对账单

### `CE_BANK_ACCOUNTS`

| 字段 | 中文名 | 说明 |
| --- | --- | --- |
| `BANK_ACCOUNT_ID` | 内部银行账户 ID | AP/AR/CE/IBY 关联的核心键 |
| `BANK_ACCOUNT_NUM` | 银行账号 | 敏感数据，报表/日志应脱敏 |
| `CURRENCY_CODE` | 账户币种 | 多币种能力还受 Account Setup 影响 |
| `ACCOUNT_OWNER_ORG_ID` | 账户所有者组织 | 通常为 Legal Entity/Legal Context，不等于具体 OU Use |
| `START_DATE/END_DATE` | 有效期 | 应与付款/收款/对账业务日期比较 |

### `CE_STATEMENT_LINES`

| 字段 | 中文名 | 业务含义 |
| --- | --- | --- |
| `TRX_TYPE` | 银行交易类型 | Debit/Credit、Payment/Receipt 等高层类型，以 CE Lookup 为准 |
| `TRX_CODE` | 银行交易代码 | 应映射 CE Bank Transaction Code，决定 AutoReconciliation 行为 |
| `AMOUNT` | 流水金额 | 正负号与 Debit/Credit 规则由文件格式/交易编码定义 |
| `STATUS` | 对账状态 | Unreconciled/Reconciled/Error 等含义，用 CE Lookup 解码 |
| `BANK_TRX_NUMBER` | 银行交易号 | 自动匹配和重复检查的重要参考，但不应单独作为全局唯一键 |

## IBY 支付状态

PPR → Proposed Payments → Payment Instruction → Payment File/Transmission 是分层模型。`PAYMENT_SERVICE_REQUEST_STATUS`、`PAYMENT_STATUS` 会出现 Submitted、Assigning/Validation、Formatting、Formatted、Transmitted、Confirmed、Failed/Terminated 等业务含义，必须使用 IBY Lookup 和 PPR 日志解码。上层 PPR 完成不代表每笔 Payment 都已被银行接受。

## `ZX_LINES` — 税行

| 字段 | 中文名 | 业务说明 |
| --- | --- | --- |
| `APPLICATION_ID` | 源应用 ID | AP、AR、PO 等，决定 `TRX_ID` 指向哪个产品对象 |
| `ENTITY_CODE/EVENT_CLASS_CODE` | 实体/事件类 | 如 AP Invoice/AR Transaction 中的具体事件模型 |
| `TRX_ID/TRX_LINE_ID/TRX_LEVEL_TYPE` | 源交易/行/层级 | 必须结合 Application/Entity 解读 |
| `TAX_REGIME_CODE/TAX` | 税制/税 | 税务设置主线 |
| `TAX_STATUS_CODE/TAX_RATE_CODE` | 税状态/税率码 | 规则确定结果，受有效期和 Owner 影响 |
| `TAX_RATE` | 税率数值 | 不等于 Tax Rate Code；税额还受 Basis/Rounding/Inclusive 影响 |
| `TAXABLE_AMT/TAX_AMT` | 计税基础/税额 | 注意交易币、税币与本位币列的区别 |
| `CANCEL_FLAG/DELETE_FLAG` | 取消/删除标志 | 历史行可保留，当前税查询应正确过滤 |

## 官方参考

- [Oracle Cash Management User Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48900/)
- [Oracle Financials Implementation Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48783/)
- [Oracle E-Business Suite eTRM User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/)
