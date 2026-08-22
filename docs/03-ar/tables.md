# Oracle Receivables 常用表结构

## 业务说明

AR 由 TCA 客户主数据、交易头/行/会计分配、Payment Schedule 余额、Receipt 状态历史与 Application 历史组成。查当前余额可使用 Payment Schedules；查历史截止日余额必须纳入截止日后核销/反核销/调整，优先使用标准 Aging/Trial Balance。

## 表级速查

| 表 | 中文名 | 粒度/用途 | 关键字段 |
| --- | --- | --- | --- |
| `HZ_PARTIES` | TCA 主体 | 组织/个人主体 | `PARTY_ID`, `PARTY_NUMBER`, `PARTY_TYPE` |
| `HZ_CUST_ACCOUNTS` | 客户账户 | Party 下的交易账户 | `CUST_ACCOUNT_ID`, `PARTY_ID`, `ACCOUNT_NUMBER` |
| `HZ_CUST_ACCT_SITES_ALL` | 客户账户地点 | Account+Party Site+OU | `CUST_ACCT_SITE_ID`, `CUST_ACCOUNT_ID`, `ORG_ID` |
| `HZ_CUST_SITE_USES_ALL` | 客户地点用途 | Bill-To/Ship-To 等 | `SITE_USE_ID`, `SITE_USE_CODE`, `ORG_ID` |
| `RA_CUSTOMER_TRX_ALL` | AR 交易头 | 发票/贷项/借项头 | `CUSTOMER_TRX_ID`, `TRX_NUMBER`, `ORG_ID` |
| `RA_CUSTOMER_TRX_LINES_ALL` | AR 交易行 | Line/Tax/Freight/Charges | `CUSTOMER_TRX_LINE_ID`, `CUSTOMER_TRX_ID`, `LINE_TYPE` |
| `RA_CUST_TRX_LINE_GL_DIST_ALL` | AR 交易 GL 分配 | 交易行会计/收入计划 | `CUST_TRX_LINE_GL_DIST_ID`, `ACCOUNT_CLASS`, `CODE_COMBINATION_ID` |
| `AR_PAYMENT_SCHEDULES_ALL` | AR 应收分期/余额 | 交易或收款的到期项 | `PAYMENT_SCHEDULE_ID`, `CLASS`, `STATUS` |
| `AR_CASH_RECEIPTS_ALL` | AR 收款头 | 每笔收款 | `CASH_RECEIPT_ID`, `RECEIPT_NUMBER`, `ORG_ID` |
| `AR_CASH_RECEIPT_HISTORY_ALL` | 收款状态历史 | Confirm/Remit/Clear/Reverse 历史 | `CASH_RECEIPT_HISTORY_ID`, `CURRENT_RECORD_FLAG` |
| `AR_RECEIVABLE_APPLICATIONS_ALL` | 收款/贷项核销 | 每次 Apply/Unapply 历史 | `RECEIVABLE_APPLICATION_ID`, `STATUS`, `DISPLAY` |
| `AR_ADJUSTMENTS_ALL` | AR 调整 | 每笔应收调整 | `ADJUSTMENT_ID`, `TYPE`, `STATUS`, `AMOUNT` |
| `RA_INTERFACE_LINES_ALL` | AutoInvoice 接口行 | 待导入头/行信息 | `INTERFACE_LINE_ID`, `INTERFACE_LINE_CONTEXT`, `ORG_ID` |
| `RA_INTERFACE_ERRORS_ALL` | AutoInvoice 错误 | 每个接口错误 | `INTERFACE_LINE_ID`, `MESSAGE_TEXT`, `INVALID_VALUE` |

## TCA 主数据字段

| 字段 | 中文名 | 业务说明 |
| --- | --- | --- |
| `HZ_PARTIES.PARTY_TYPE` | 主体类型 | 常见 `ORGANIZATION`、`PERSON`、`GROUP`、`RELATIONSHIP`；交易客户通常使用组织/个人 Party |
| `HZ_CUST_ACCOUNTS.STATUS` | 客户账户状态 | 常见 `A`有效、`I`无效 |
| `HZ_CUST_ACCT_SITES_ALL.STATUS` | 账户地点状态 | 地点失效不等于 Party/Account 失效 |
| `HZ_CUST_SITE_USES_ALL.SITE_USE_CODE` | 地点用途 | 常见 `BILL_TO`、`SHIP_TO`、`STATEMENT`、`DUN`；以 TCA/AR Lookup 为准 |
| `PRIMARY_FLAG` | 主要用途 | `Y/N`；同一账户/OU/用途的主要地点应受业务约束 |

## `RA_CUSTOMER_TRX_ALL` — 交易头

| 字段 | 中文名 | 业务含义 |
| --- | --- | --- |
| `CUSTOMER_TRX_ID` | AR 交易 ID | 头主键，关联 Lines/Payment Schedule/XLA 源交易 |
| `TRX_NUMBER` | 交易编号 | 由 Transaction Source/文档序列控制 |
| `CUST_TRX_TYPE_ID` | 交易类型 ID | 关联 `RA_CUST_TRX_TYPES_ALL`，决定 Invoice/Credit Memo/Debit Memo 等业务性质 |
| `BATCH_SOURCE_ID` | 交易来源 ID | 关联 `RA_BATCH_SOURCES_ALL`，控制自动编号与 AutoInvoice 验证 |
| `TRX_DATE` | 交易日期 | 发票/税务业务日期 |
| `COMPLETE_FLAG` | 是否完成 | `Y/N`；未完成交易通常不进入正常会计/收款链 |
| `BILL_TO_CUSTOMER_ID` | Bill-To 客户账户 | 关联 `HZ_CUST_ACCOUNTS.CUST_ACCOUNT_ID` |
| `BILL_TO_SITE_USE_ID` | Bill-To 地点用途 | 应属于相同 OU 且有效 |
| `PREVIOUS_CUSTOMER_TRX_ID` | 原交易 ID | Credit Memo/关联交易追溯线索 |
| `ORG_ID` | OU ID | AR 交易数据边界 |

## `RA_CUSTOMER_TRX_LINES_ALL.LINE_TYPE`

| 常见值 | 中文含义 | 关联说明 |
| --- | --- | --- |
| `LINE` | 商品/服务行 | 主交易行 |
| `TAX` | 税行 | `LINK_TO_CUST_TRX_LINE_ID` 可指向对应 LINE |
| `FREIGHT` | 运费 | 可为头级或关联具体 LINE |
| `CHARGES` | 其他收费 | 受交易来源/业务规则影响 |

## Payment Schedule 和收款

### `AR_PAYMENT_SCHEDULES_ALL`

| 字段/值 | 中文含义 | 说明 |
| --- | --- | --- |
| `CLASS='INV'` | 发票 | 应收正向交易 |
| `CLASS='CM'` | 贷项通知单 | 减少应收 |
| `CLASS='DM'` | 借项通知单 | 增加应收 |
| `CLASS='CB'` | Chargeback | 从收款短款/争议产生的新应收 |
| `CLASS='PMT'` | 收款 | 收款 Payment Schedule |
| `STATUS='OP'` | 开放 | `AMOUNT_DUE_REMAINING` 通常非零 |
| `STATUS='CL'` | 关闭 | 当前余额已处理完，不表示历史上从未反核销 |

### `AR_RECEIVABLE_APPLICATIONS_ALL`

| 字段 | 中文含义 | 常见含义 |
| --- | --- | --- |
| `STATUS` | 核销状态 | 常见 `APP`已核销、`UNAPP`未核销、`ACC`在账、`UNID`未识别；以 AR Lookup 为准 |
| `DISPLAY` | 当前有效标志 | 常见 `Y/N`；Apply/Unapply 会留下历史行，当前状态必须结合该标志 |
| `AMOUNT_APPLIED` | 核销金额 | 通常为被核销交易币金额 |
| `AMOUNT_APPLIED_FROM` | 来源币核销金额 | 跨币种核销时与 `AMOUNT_APPLIED` 可不同 |
| `APPLIED_CUSTOMER_TRX_ID` | 被核销交易 | APP 行指向发票/借项等 |

`AR_CASH_RECEIPTS_ALL.STATUS` 是头快照；Confirmed/Remitted/Cleared/Reversed 的完整变化应查 `AR_CASH_RECEIPT_HISTORY_ALL`，当前行通常以 `CURRENT_RECORD_FLAG='Y'` 识别。

## AutoInvoice 重要字段

| 字段 | 中文名 | 官方规则要点 |
| --- | --- | --- |
| `INTERFACE_LINE_CONTEXT` | 交易弹性域上下文 | 与 Attribute1..15 一起唯一标识来源交易 |
| `LINE_TYPE` | 接口行类型 | `LINE`、`TAX`、`FREIGHT`、`CHARGES` 的必填/禁填字段不同 |
| `CUST_TRX_TYPE_ID/NAME` | 交易类型 | 使用 ID 还是 Name 受 Batch Source 验证选项影响 |
| `GL_DATE` | GL 日期 | 必须在 Open/Future 可用期间，并与 Accounting Rule 周期兼容 |
| `PAYMENT_TRXN_EXTENSION_ID` | IBY 付款扩展 ID | R12 不应继续使用已废弃的客户银行账号接口列 |

## 官方参考

- [Oracle Receivables Reference Guide: AutoInvoice Tables and Columns](https://docs.oracle.com/cd/E26401_01/doc.122/f10312/T447348T383863.htm)
- [Oracle Receivables Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f10310/)
- [Oracle E-Business Suite eTRM User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/)
