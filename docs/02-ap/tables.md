# Oracle Payables 常用表结构

## 业务说明

AP 数据从“供应商主数据 → 发票头/行/分配 → 挂起与审批 → 付款分期 → 付款与发票核销 → SLA”组成。发票头金额不是会计粒度；账户、PO/收货匹配和 GL Date 主要在分配层。

## 表级速查

| 表 | 中文名 | 粒度/用途 | 主键与关联 |
| --- | --- | --- | --- |
| `AP_SUPPLIERS` | 供应商 | 每个供应商 | `VENDOR_ID`, `PARTY_ID` 关联 TCA |
| `AP_SUPPLIER_SITES_ALL` | 供应商地点 | Supplier+OU+Site | `VENDOR_SITE_ID`, `VENDOR_ID`, `ORG_ID` |
| `AP_INVOICES_ALL` | AP 发票头 | 每张发票 | `INVOICE_ID`, `VENDOR_ID`, `VENDOR_SITE_ID`, `ORG_ID` |
| `AP_INVOICE_LINES_ALL` | AP 发票行 | 发票+行号 | `INVOICE_ID`, `LINE_NUMBER` |
| `AP_INVOICE_DISTRIBUTIONS_ALL` | AP 发票分配 | 发票行的会计/匹配分配 | `INVOICE_DISTRIBUTION_ID`, `DIST_CODE_COMBINATION_ID` |
| `AP_HOLDS_ALL` | AP 发票挂起 | 每次 Hold/Release | `INVOICE_ID`, `HOLD_LOOKUP_CODE`, `RELEASE_LOOKUP_CODE` |
| `AP_PAYMENT_SCHEDULES_ALL` | AP 付款分期 | 发票每个到期分期 | `INVOICE_ID`, `PAYMENT_NUM`, `DUE_DATE` |
| `AP_CHECKS_ALL` | AP 付款 | 每笔付款/支票 | `CHECK_ID`, `CHECK_NUMBER`, `ORG_ID` |
| `AP_INVOICE_PAYMENTS_ALL` | 发票付款核销 | Invoice+Payment 关联 | `INVOICE_PAYMENT_ID`, `INVOICE_ID`, `CHECK_ID` |
| `AP_INVOICES_INTERFACE` | AP 发票接口头 | 待导入发票 | `INVOICE_ID`, `SOURCE`, `GROUP_ID`, `ORG_ID` |
| `AP_INVOICE_LINES_INTERFACE` | AP 发票接口行 | 待导入发票行 | `INVOICE_ID`, `INVOICE_LINE_ID`, `LINE_NUMBER` |
| `AP_INTERFACE_REJECTIONS` | AP 接口拒绝 | 每个拒绝原因 | `PARENT_TABLE`, `PARENT_ID`, `REJECT_LOOKUP_CODE` |

## `AP_INVOICES_ALL` — 发票头

| 字段 | 中文名 | 业务含义/常见值 |
| --- | --- | --- |
| `INVOICE_ID` | 发票 ID | 内部主键；接口/跟踪使用 ID，展示使用 `INVOICE_NUM` |
| `INVOICE_TYPE_LOOKUP_CODE` | 发票类型 | 常见 `STANDARD`、`CREDIT`、`DEBIT`、`PREPAYMENT`、`EXPENSE REPORT`、`MIXED` |
| `INVOICE_NUM` | 供应商发票号 | 重复检查通常结合 Supplier/OU，以 Payables Options 为准 |
| `INVOICE_AMOUNT` | 发票总额 | 交易币金额；应与行/税及业务规则一致 |
| `INVOICE_CURRENCY_CODE` | 发票币种 | 与 `PAYMENT_CURRENCY_CODE`、Ledger Currency 可不同 |
| `INVOICE_DATE` | 发票日期 | 业务/税务日期，不等于会计日期 |
| `GL_DATE` | 头层 GL 日期 | 默认线索；最终会计应检查分配 `ACCOUNTING_DATE`/XLA |
| `SOURCE` | 发票来源 | 手工、Open Interface、Expense Report 等，实际值查 AP Lookup/Source 设置 |
| `PAYMENT_STATUS_FLAG` | 付款状态 | 常见 `N`未付、`P`部分付、`Y`已付；余额仍以 Payment Schedules 为准 |
| `WFAPPROVAL_STATUS` | Workflow 审批状态 | 可见 Initiated/Required/Approved/Rejected/Not Required 等含义，必须用 Lookup/Workflow 解码 |
| `CANCELLED_DATE` | 作废日期 | 非空表示已通过标准流程取消 |
| `ORG_ID` | OU ID | MOAC 数据边界 |

> Validation Status 并非只靠一个头字段判断。标准页面/报表会结合发票类型、分配匹配状态、Hold 和 API 结果，定制 SQL 不应只把 `WFAPPROVAL_STATUS` 当成 Validation Status。

## 发票行与分配

### `AP_INVOICE_LINES_ALL.LINE_TYPE_LOOKUP_CODE`

| 常见值 | 中文含义 | 说明 |
| --- | --- | --- |
| `ITEM` | 货物/费用行 | 可匹配 PO/Receipt 或非 PO 费用 |
| `TAX` | 税行 | 通常由 EBTax 生成/管理 |
| `FREIGHT` | 运费 | 可独立分配或按规则分摊 |
| `MISCELLANEOUS` | 杂项 | 非物料/税/运费的其他费用 |
| `PREPAY` | 预付核销行 | 标准发票上的预付核销影响 |
| `AWT` | 预扣税 | 取值/生成方式受 Withholding Tax 设置影响 |

### `AP_INVOICE_DISTRIBUTIONS_ALL`

| 字段 | 中文名 | 业务说明 |
| --- | --- | --- |
| `LINE_TYPE_LOOKUP_CODE` | 分配类型 | 比行类型更接近会计/税/价差粒度，具体值查 AP Lookup |
| `DIST_CODE_COMBINATION_ID` | 分配账户 CCID | 默认账户；最终 SLA 可覆盖全部或部分段 |
| `ACCOUNTING_DATE` | 会计日期 | 决定 AP/SLA 会计期间 |
| `MATCH_STATUS_FLAG` | 匹配/验证状态 | 常见 `A`已验证、`N`未验证、`T`需重新验证等含义；以 AP Lookup/标准 API 为准 |
| `POSTED_FLAG` | 是否已进入会计链 | 常见 `Y/N`；不能替代 XLA Final/Transfer/Post 状态 |
| `PO_DISTRIBUTION_ID` | PO 分配 ID | P2P 对账的稳定关联键 |
| `RCV_TRANSACTION_ID` | 收货交易 ID | 用于收货匹配和 Accrual Reconciliation |
| `REVERSAL_FLAG/PARENT_REVERSAL_ID` | 反冲标志/原行 | 跟踪取消、预付取消核销等反向分配 |

## 付款状态

### `AP_PAYMENT_SCHEDULES_ALL.PAYMENT_STATUS_FLAG`

| 值 | 含义 |
| --- | --- |
| `N` | 未付 |
| `P` | 部分付 |
| `Y` | 已付/当前分期余额为零 |

`HOLD_FLAG='Y'` 表示分期付款挂起。PPR 是否选中发票还受 Due Date、Payment Method、Pay Group/Priority、Currency、Supplier Site、IBY 和请求参数影响。

`AP_CHECKS_ALL.STATUS_LOOKUP_CODE` 会出现 Negotiable/Issued/Voided/Stopped/Cleared 等业务含义，确切代码必须通过 AP Lookup 解码；不能只以 `VOID_DATE` 判断整个 IBY/银行链路状态。

## 结构自检 SQL

```sql
SELECT owner, table_name, column_id, column_name,
       data_type, data_length, nullable
  FROM all_tab_columns
 WHERE table_name IN ('AP_INVOICES_ALL',
                      'AP_INVOICE_LINES_ALL',
                      'AP_INVOICE_DISTRIBUTIONS_ALL',
                      'AP_PAYMENT_SCHEDULES_ALL',
                      'AP_CHECKS_ALL')
 ORDER BY table_name, column_id;

SELECT lookup_type, lookup_code, meaning, description
  FROM fnd_lookup_values_vl
 WHERE lookup_type IN ('INVOICE TYPE',
                       'PAYMENT STATUS',
                       'CHECK STATE')
 ORDER BY lookup_type, lookup_code;
```

> Lookup Type 名称可因应用内部定义而不同。先根据页面 Meaning 在 `FND_LOOKUP_VALUES_VL` 搜索，再固化查询。

## 官方参考

- [Oracle Payables Implementation Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48761/)
- [Oracle E-Business Suite eTRM User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/)
