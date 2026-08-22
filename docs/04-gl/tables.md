# Oracle General Ledger 常用表结构

## 业务说明

GL 的业务层级是 Ledger → Batch → Journal Header → Journal Line → Balance。子账数据经 SLA 进入 `GL_INTERFACE`，Journal Import 生成日记账，Posting 更新 `GL_BALANCES`。日记账行是交易流量，Balance 是 Ledger+CCID+Currency+Period+Balance Type 的累计/期间快照。

## 表级速查

| 表 | 中文名 | 业务粒度 | 关键字段 |
| --- | --- | --- | --- |
| `GL_LEDGERS` | 账簿 | 每个 Ledger | `LEDGER_ID`, `CHART_OF_ACCOUNTS_ID` |
| `GL_LEDGER_SETS` | 账簿集 | 每个 Ledger Set | `LEDGER_SET_ID`, `NAME` |
| `GL_ACCESS_SETS` | 数据访问集 | 每个 Data Access Set | `ACCESS_SET_ID`, `SECURITY_SEGMENT_CODE` |
| `GL_CODE_COMBINATIONS` | 会计科目组合 | 每个 CCID | `CODE_COMBINATION_ID`, `SEGMENT1..N` |
| `GL_JE_BATCHES` | 日记账批 | 每个 Batch | `JE_BATCH_ID`, `STATUS`, `APPROVAL_STATUS_CODE` |
| `GL_JE_HEADERS` | 日记账头 | 每个 Journal | `JE_HEADER_ID`, `LEDGER_ID`, `PERIOD_NAME`, `STATUS` |
| `GL_JE_LINES` | 日记账行 | Journal+行号 | `JE_HEADER_ID`, `JE_LINE_NUM`, `CODE_COMBINATION_ID` |
| `GL_INTERFACE` | GL 日记账接口 | 待 Import 分录行 | `STATUS`, `LEDGER_ID`, `GROUP_ID`, `USER_JE_SOURCE_NAME` |
| `GL_IMPORT_REFERENCES` | GL 导入参考 | GL Journal Line 与子账链接 | `JE_HEADER_ID`, `JE_LINE_NUM`, `GL_SL_LINK_ID/TABLE` |
| `GL_BALANCES` | GL 科目余额 | Ledger+CCID+Currency+Period+Flag | `LEDGER_ID`, `CODE_COMBINATION_ID`, `PERIOD_NAME`, `ACTUAL_FLAG` |
| `GL_PERIOD_STATUSES` | GL/子账期间状态 | Application+Ledger+Period | `APPLICATION_ID`, `CLOSING_STATUS` |
| `GL_DAILY_RATES` | 日汇率 | 币种对+日期+类型 | `CONVERSION_DATE`, `CONVERSION_TYPE`, `CONVERSION_RATE` |

## `GL_JE_BATCHES` — 日记账批

| 字段 | 中文名 | 业务含义 |
| --- | --- | --- |
| `JE_BATCH_ID` | 日记账批 ID | Header 的外键，审批/过账常以 Batch 为单位 |
| `NAME` | 批名称 | 可包含 Source/Period/系统生成信息，不应作为稳定唯一集成键 |
| `STATUS` | 批状态 | 与 Header Status 共同判断是否可审批/过账 |
| `APPROVAL_STATUS_CODE` | 审批状态 | Required/In Process/Approved/Rejected 等，请用 GL Lookup 解码 |
| `POSTED_DATE` | 过账日期 | 已过账批的实际过账时间 |
| `POSTING_RUN_ID` | 过账运行 ID | 跟踪 Posting 程序批次 |

## `GL_JE_HEADERS` — 日记账头

| 字段 | 中文名 | 业务含义/值 |
| --- | --- | --- |
| `LEDGER_ID` | 账簿 ID | 决定 COA、日历、本位币和 Data Access Set |
| `JE_SOURCE` | 日记账来源 | Payables/Receivables/Assets/Manual 等，应与 `GL_JE_SOURCES` 解码 |
| `JE_CATEGORY` | 日记账类别 | Invoices/Payments/Receipts/Adjustment 等业务性质 |
| `STATUS` | Journal 状态 | `U` 常表示 Unposted，`P` 常表示 Posted；其他值可为错误/导入状态，必须通过 GL 标准解码 |
| `ACTUAL_FLAG` | 余额类型 | `A`实际、`B`预算、`E`保留/承诺 |
| `CURRENCY_CODE` | Journal 币种 | 交易币；`ACCOUNTED_*` 为 Ledger Currency |
| `PERIOD_NAME` | 会计期间 | 需与 `DEFAULT_EFFECTIVE_DATE` 及 Ledger Calendar 一致 |
| `CONVERSION_TYPE/DATE/RATE` | 汇率属性 | 非本位币 Journal 的折算依据 |
| `REVERSED_JE_HEADER_ID` | 被反冲 Journal | 用于反冲链跟踪，同时查 Reversal Period/Method |

## `GL_JE_LINES` — 日记账行

| 字段 | 中文名 | 说明 |
| --- | --- | --- |
| `ENTERED_DR/ENTERED_CR` | 交易币借/贷 | 同一行通常仅一侧有值 |
| `ACCOUNTED_DR/ACCOUNTED_CR` | 本位币借/贷 | 外币 Journal 经汇率折算后金额 |
| `CODE_COMBINATION_ID` | 会计科目 CCID | 必须属于 Ledger COA，在有效日可过账 |
| `EFFECTIVE_DATE` | 有效/过账日期 | 决定 Period，不等于 Creation Date |
| `REFERENCE_1..10` | 导入参考 | 含义由 Source/Interface 决定，不应跨 Source 固定解读 |

## `GL_BALANCES` — 余额

| 字段 | 中文名 | 说明 |
| --- | --- | --- |
| `ACTUAL_FLAG` | 实际/预算/保留 | `A/B/E` |
| `CURRENCY_CODE` | 余额币种 | Ledger Currency、Foreign Currency、Statistical Currency 需按报表参数区分 |
| `TRANSLATED_FLAG` | 折算标志 | 用于外币/折算余额识别，NULL 不一定是错误 |
| `BEGIN_BALANCE_DR/CR` | 期初借/贷余额 | 净额通常用 Dr-Cr 计算，显示符号受账户类型影响 |
| `PERIOD_NET_DR/CR` | 本期借/贷发生 | 与期初共同计算期末 |
| `*_BEQ` | 本位币等值 | 外币余额的 Ledger Currency Equivalent |

## `GL_INTERFACE` 常见状态原则

- `STATUS='NEW'` 通常表示等待 Journal Import。
- Import 失败后 `STATUS` 可变为具体错误代码，应用 Journal Import Execution Report/GL Lookup 解码，不建立不完整的自制代码表。
- `GROUP_ID` 隔离一次导入批次；`REFERENCE*` 应保存可追溯源单据的值。
- 已成功 Import 的数据不再以 `GL_INTERFACE` 为完整审计源，应跟踪 Journal 和 `GL_IMPORT_REFERENCES`。

## 官方参考

- [Oracle General Ledger Implementation Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48747/)
- [Oracle E-Business Suite eTRM User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/)
