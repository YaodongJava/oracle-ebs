# 财务公共基础常用表结构

## 业务边界

财务公共层为所有子账提供企业结构、Ledger、COA、日历/期间、币种/汇率、MOAC、SLA 和 FND 会话权限。排查任何子账问题时，应先回答：哪个 User/Responsibility、OU、Ledger、Legal Entity、COA、Period 和 Application。

## 表级速查

| 表/视图 | 中文名 | 业务粒度 | 关键字段 |
| --- | --- | --- | --- |
| `GL_LEDGERS` | 账簿 | 每个 Ledger 一行 | `LEDGER_ID`, `CHART_OF_ACCOUNTS_ID`, `CURRENCY_CODE`, `PERIOD_SET_NAME` |
| `HR_OPERATING_UNITS` | 业务实体 OU | 每个 OU 一行 | `ORGANIZATION_ID`, `SET_OF_BOOKS_ID`, `DEFAULT_LEGAL_CONTEXT_ID` |
| `HR_ALL_ORGANIZATION_UNITS` | HR 组织基本信息 | 组织及有效期 | `ORGANIZATION_ID`, `BUSINESS_GROUP_ID`, `DATE_FROM/DATE_TO` |
| `XLE_ENTITY_PROFILES` | 法人实体 | 每个 Legal Entity 一行 | `LEGAL_ENTITY_ID`, `LEGAL_ENTITY_IDENTIFIER` |
| `ORG_ORGANIZATION_DEFINITIONS` | 库存组织定义综合视图 | 每个 IO 一行 | `ORGANIZATION_ID`, `OPERATING_UNIT`, `LEGAL_ENTITY` |
| `GL_CODE_COMBINATIONS` | 会计科目组合 | 每个 CCID 一行 | `CODE_COMBINATION_ID`, `SEGMENT1..N`, `ACCOUNT_TYPE` |
| `FND_FLEX_VALUES_VL` | 弹性域段值 | 每个值集值一行 | `FLEX_VALUE_SET_ID`, `FLEX_VALUE`, `SUMMARY_FLAG` |
| `GL_PERIOD_STATUSES` | 应用期间状态 | Application+Ledger+Period | `APPLICATION_ID`, `SET_OF_BOOKS_ID`, `PERIOD_NAME`, `CLOSING_STATUS` |
| `GL_DAILY_RATES` | 日汇率 | 币种对+日期+类型 | `FROM_CURRENCY`, `TO_CURRENCY`, `CONVERSION_DATE/TYPE` |
| `FND_USER` | EBS 用户 | 每个用户一行 | `USER_ID`, `USER_NAME`, `START_DATE/END_DATE` |
| `FND_RESPONSIBILITY_VL` | 职责 | 每个职责一行 | `RESPONSIBILITY_ID`, `APPLICATION_ID`, `MENU_ID`, `REQUEST_GROUP_ID` |
| `FND_PROFILE_OPTION_VALUES` | Profile 选项值 | Profile+层级+层级值 | `PROFILE_OPTION_ID`, `LEVEL_ID`, `LEVEL_VALUE`, `PROFILE_OPTION_VALUE` |
| `XLA_TRANSACTION_ENTITIES` | SLA 交易实体 | 子账交易的 SLA 实体 | `APPLICATION_ID`, `ENTITY_ID`, `ENTITY_CODE`, `SOURCE_ID_INT_1..4` |
| `XLA_EVENTS` | SLA 会计事件 | 每个事件一行 | `EVENT_ID`, `ENTITY_ID`, `EVENT_TYPE_CODE`, `EVENT_STATUS_CODE` |
| `XLA_AE_HEADERS` | SLA 分录头 | 每个会计分录头 | `AE_HEADER_ID`, `EVENT_ID`, `LEDGER_ID`, `GL_TRANSFER_STATUS_CODE` |
| `XLA_AE_LINES` | SLA 分录行 | 每个分录行 | `AE_HEADER_ID`, `AE_LINE_NUM`, `CODE_COMBINATION_ID`, `ACCOUNTING_CLASS_CODE` |

## 重要表与字段

### `GL_LEDGERS` — 账簿

| 字段 | 中文含义 | 说明 |
| --- | --- | --- |
| `LEDGER_ID` | 账簿 ID | GL 和 SLA 的核心组织键 |
| `NAME` / `SHORT_NAME` | 账簿名/短名 | 业务显示与程序引用 |
| `LEDGER_CATEGORY_CODE` | 账簿类别 | 常见 `PRIMARY`、`SECONDARY`；报告币种相关表述以实例设置为准 |
| `CHART_OF_ACCOUNTS_ID` | 会计科目表 ID | 关联 Accounting KFF 结构 |
| `CURRENCY_CODE` | 账簿本位币 | 交易币折算后的 Accounted Currency |
| `PERIOD_SET_NAME` | 会计日历 | 与 `ACCOUNTED_PERIOD_TYPE` 共同确定期间 |
| `SLA_ACCOUNTING_METHOD_CODE` | SLA 会计方法 | 决定各子账应用会计定义 |

### `GL_CODE_COMBINATIONS` — 科目组合

| 字段 | 中文含义 | 常见值/业务影响 |
| --- | --- | --- |
| `CODE_COMBINATION_ID` | 科目组合 ID | 交易表通常保存 CCID，不重复保存全部段 |
| `SEGMENT1..N` | 各会计维度 | 顺序与含义由 COA 定义，不能假设 `SEGMENT1` 永远是公司 |
| `ACCOUNT_TYPE` | 账户类型 | 常见 `A`资产、`L`负债、`O`所有者权益、`R`收入、`E`费用；以 GL Lookup/eTRM 为准 |
| `ENABLED_FLAG` | CCID 是否启用 | `Y/N` |
| `DETAIL_POSTING_ALLOWED_FLAG` | 是否允许明细过账 | `N` 时可查到但日记账不可过账 |
| `SUMMARY_FLAG` | 是否汇总账户 | 明细交易通常应使用 `N` |
| `START_DATE_ACTIVE/END_DATE_ACTIVE` | 有效期 | 应与会计日期而不仅是当前日期比较 |

### `GL_PERIOD_STATUSES` — 期间状态

`CLOSING_STATUS` 常见值：

| 值 | 中文含义 | 说明 |
| --- | --- | --- |
| `N` | 从未打开 | 未建立可过账期间 |
| `F` | 将来可录入 | 允许某些未来期交易/日记账，不等于完全打开 |
| `O` | 打开 | 允许标准交易/过账 |
| `C` | 已关闭 | 可通过授权流程重开 |
| `P` | 永久关闭 | 不应作为普通关期使用，通常不可重开 |

> 同一 `PERIOD_NAME` 在不同 `APPLICATION_ID` 下可有不同状态；必须联接 `FND_APPLICATION` 识别 GL/AP/AR/PO 等模块。

### XLA 状态字段

| 字段 | 业务含义 | 注意 |
| --- | --- | --- |
| `XLA_EVENTS.EVENT_STATUS_CODE` | 事件会计状态 | 常见未处理/已处理代码，与 `PROCESS_STATUS_CODE` 组合判断 |
| `XLA_AE_HEADERS.ACCOUNTING_ENTRY_STATUS_CODE` | SLA 分录状态 | Draft/Final 等业务含义应通过 XLA Lookup 解码 |
| `XLA_AE_HEADERS.GL_TRANSFER_STATUS_CODE` | 转 GL 状态 | 不能仅以“已会计”推断“已过账 GL” |
| `XLA_AE_LINES.ACCOUNTING_CLASS_CODE` | 会计分类 | 如 Liability、Expense、Receivable、Revenue、Tax，按应用事件模型解读 |

SLA 状态代码可因产品和处理阶段而不同，查询时建议同时输出原始代码和 Lookup Meaning，不将单字母翻译硬编码到业务程序。

## 状态解码通用 SQL

```sql
SELECT flv.lookup_type, flv.lookup_code,
       flv.meaning, flv.description,
       flv.enabled_flag, flv.start_date_active,
       flv.end_date_active
  FROM fnd_lookup_values_vl flv
 WHERE flv.lookup_type = :p_lookup_type
 ORDER BY flv.lookup_code;

SELECT owner, table_name, column_id, column_name,
       data_type, data_length, nullable
  FROM all_tab_columns
 WHERE table_name = UPPER(:p_table_name)
 ORDER BY owner, column_id;
```

## 官方参考

- [Oracle E-Business Suite eTRM User's Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/)
- [Oracle E-Business Suite Multiple Organizations Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48833/)
- [Oracle Subledger Accounting Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48771/)
