# Oracle Assets 常用表结构

## 业务说明

FA 数据不能只查资产主表。一项资产的名称/类别在 Addition，成本/折旧属性按 Book 保存在 Books 历史，位置/责任人/费用账户在 Distribution History，每次业务变更在 Transaction Headers。当前行通常用 `DATE_INEFFECTIVE IS NULL` 识别，历史报表必须以业务截止日选取有效行。

## 表级速查

| 表 | 中文名 | 业务粒度 | 关键字段 |
| --- | --- | --- | --- |
| `FA_ADDITIONS_B` | 资产主数据 | 每项资产 | `ASSET_ID`, `ASSET_NUMBER`, `ASSET_CATEGORY_ID`, `ASSET_TYPE` |
| `FA_ADDITIONS_TL` | 资产多语言说明 | Asset+语言 | `ASSET_ID`, `LANGUAGE`, `DESCRIPTION` |
| `FA_BOOK_CONTROLS` | 资产账簿控制 | 每个 Asset Book | `BOOK_TYPE_CODE`, `SET_OF_BOOKS_ID`, Calendar/Period |
| `FA_BOOKS` | 资产账簿历史 | Asset+Book+有效期 | `ASSET_ID`, `BOOK_TYPE_CODE`, `TRANSACTION_HEADER_ID_IN/OUT` |
| `FA_CATEGORIES_B` | 资产类别 | 每个 Category KFF 组合 | `CATEGORY_ID`, `SEGMENT1..N` |
| `FA_CATEGORY_BOOKS` | 类别账簿设置 | Category+Book | 资产成本/折旧/CIP/处置账户 |
| `FA_DISTRIBUTION_HISTORY` | 资产分配历史 | Asset+Book+分配有效期 | `DISTRIBUTION_ID`, `LOCATION_ID`, `ASSIGNED_TO`, `CODE_COMBINATION_ID` |
| `FA_TRANSACTION_HEADERS` | 资产交易头 | 每次资产交易 | `TRANSACTION_HEADER_ID`, `TRANSACTION_TYPE_CODE`, `ASSET_ID` |
| `FA_DEPRN_PERIODS` | FA 折旧期间 | Book+Period | `BOOK_TYPE_CODE`, `PERIOD_COUNTER`, `PERIOD_OPEN/CLOSE_DATE` |
| `FA_DEPRN_SUMMARY` | 资产折旧汇总 | Asset+Book+Period | `ASSET_ID`, `BOOK_TYPE_CODE`, `PERIOD_COUNTER` |
| `FA_DEPRN_DETAIL` | 资产折旧分配明细 | Asset+Distribution+Period | `DISTRIBUTION_ID`, `DEPRN_AMOUNT`, `DEPRN_RESERVE` |
| `FA_RETIREMENTS` | 资产处置 | 每次 Full/Partial Retirement | `RETIREMENT_ID`, `ASSET_ID`, `STATUS` |
| `FA_MASS_ADDITIONS` | 批量资产增加接口 | 每个待处理资产行 | `MASS_ADDITION_ID`, `POSTING_STATUS`, `BOOK_TYPE_CODE` |

## `FA_ADDITIONS_B` — 资产主数据

| 字段 | 中文名 | 业务含义/常见值 |
| --- | --- | --- |
| `ASSET_ID` | 资产 ID | 所有 FA 历史表的核心关联键 |
| `ASSET_NUMBER` | 资产编号 | 可手工/自动生成，展示键 |
| `TAG_NUMBER` | 资产标签号 | 实物盘点常用，不一定所有资产都必填 |
| `ASSET_CATEGORY_ID` | 资产类别 ID | 决定各 Book 默认账户/折旧属性 |
| `ASSET_TYPE` | 资产类型 | 常见 `CAPITALIZED`、`CIP`、`EXPENSED`、`GROUP`；以 FA Lookup/已启用功能为准 |
| `CURRENT_UNITS` | 当前单位数 | 应与当前 Distribution History 单位合计一致 |
| `PARENT_ASSET_ID` | 父资产 ID | 组件/附属资产层级 |

## `FA_BOOKS` — 资产账簿历史

| 字段 | 中文名 | 业务说明 |
| --- | --- | --- |
| `BOOK_TYPE_CODE` | 资产账簿 | Corporate/Tax Book 关键键 |
| `DATE_EFFECTIVE/DATE_INEFFECTIVE` | 历史有效期 | `DATE_INEFFECTIVE IS NULL` 通常为当前账簿行 |
| `TRANSACTION_HEADER_ID_IN/OUT` | 生效/失效交易 | 将 Books 历史变化连回 Transaction Header |
| `COST` | 当前成本 | 当前历史行的 Book Cost |
| `ORIGINAL_COST` | 原始成本 | 不随普通成本调整同步表示“当前成本” |
| `SALVAGE_VALUE` | 净残值 | 影响 Depreciable Basis，受 Book 规则限制 |
| `DATE_PLACED_IN_SERVICE` | 启用日期 | 与 Prorate Convention 共同决定开始折旧日 |
| `DEPRN_METHOD_CODE` | 折旧方法 | 结合 `LIFE_IN_MONTHS`、Rate/Table 和 Convention |
| `DEPRECIATE_FLAG` | 是否计提折旧 | `YES/NO` 或实例对应标准值，以 FA Lookup/eTRM 为准 |
| `PERIOD_COUNTER_FULLY_RESERVED` | 完全折旧期 | 用于判断 Fully Reserved 时点 |
| `PERIOD_COUNTER_FULLY_RETIRED` | 完全处置期 | 用于判断 Full Retirement 时点 |

## `FA_DISTRIBUTION_HISTORY` — 分配历史

| 字段 | 中文名 | 业务说明 |
| --- | --- | --- |
| `UNITS_ASSIGNED` | 分配单位数 | 当前所有分配行合计应与 Asset Current Units 一致 |
| `CODE_COMBINATION_ID` | 折旧费用账户 | 资产分配到的 GL Expense CCID，不是 Asset Cost Account |
| `LOCATION_ID` | FA 位置 ID | 关联 Asset Location KFF，不等于 HR Location/Inventory Locator |
| `ASSIGNED_TO` | 责任员工 ID | 通常关联 HR Person，需按有效日查员工名 |
| `DATE_INEFFECTIVE` | 失效日 | NULL 通常为当前分配；转移会关闭旧行并建新行 |

## `FA_DEPRN_SUMMARY` — 折旧汇总

| 字段 | 中文名 | 说明 |
| --- | --- | --- |
| `DEPRN_AMOUNT` | 本期折旧 | 包含当期计算/调整影响，分析时还需查调整列 |
| `YTD_DEPRN` | 本年累计折旧 | Fiscal Year-to-Date，不是从启用日累计 |
| `DEPRN_RESERVE` | 累计折旧 | 至该期的折旧准备 |
| `DEPRN_ADJUSTMENT_AMOUNT` | 折旧调整 | 回溯 Cost/Life/Method 变更可产生 |
| `BONUS_DEPRN_AMOUNT` | 奖励折旧 | 只在相关税务/折旧功能启用时有业务意义 |
| `IMPAIRMENT_AMOUNT` | 减值金额 | 受资产减值功能和会计规则影响 |

## `FA_MASS_ADDITIONS.POSTING_STATUS`

常见业务含义包括 New、On Hold、Posted、Delete、Merge/Split 过程和 Error。内部代码会随处理阶段改变，应通过 Mass Additions Queue/Posting Status Lookup 解码，不直接更新 `POSTING_STATUS` 推进数据。

## 官方参考

- [Oracle E-Business Suite R12.2 Documentation Library](https://docs.oracle.com/cd/E26401_01/index.htm)
- [Oracle E-Business Suite eTRM User's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/)
