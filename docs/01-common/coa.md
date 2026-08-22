# 会计科目与会计弹性域（COA / Accounting Flexfield）

## 核心原理

Ledger 通过 `CHART_OF_ACCOUNTS_ID` 使用一套 Accounting Key Flexfield 结构。结构由段、值集、限定词、交叉验证规则和安全规则组成；段值组合实例存放于 `GL_CODE_COMBINATIONS`（CCID）。

典型结构：

```text
Company（Balancing）- Cost Center（Cost Center）- Account（Natural Account）
- Intercompany（Intercompany）- Product - Project - Future
```

- Balancing Segment 用于平衡和资产负债表边界，常与法人平衡段分配配合。
- Natural Account 决定账户类型（资产/负债/所有者权益/收入/费用），影响年结。
- Cost Center 支持管理会计和成本归集，不应与 OU 概念混用。
- Dynamic Insertion 允许运行时创建 CCID，但仍受值集、安全和交叉验证约束。

## 设计与配置

1. 确定全局报表维度、法定报表边界和未来扩展需求。
2. 定义 Value Set，包括格式、长度、独立/从属验证和值安全。
3. 定义 Accounting Flexfield 结构、段顺序和限定词。
4. 定义段值、父子层级和汇总组；禁止将父值用于明细过账。
5. 定义 Cross-Validation Rules 防止非法组合，定义 Security Rules 限制用户可见值。
6. 编译弹性域并完成日记账、SLA、FSG 和接口测试。

R12.2 生产中改变已使用段的长度、顺序、限定词或值集是高风险变更，应先评估 SLA、FSG、定制接口、子账账户默认及历史报表。

## 常用 SQL

```sql
-- Ledger 与 COA
SELECT ledger_id, name, chart_of_accounts_id, currency_code,
       period_set_name, accounted_period_type
  FROM gl_ledgers
 WHERE ledger_id = :p_ledger_id;

-- 科目组合；SEGMENT 数量以实际 COA 为准
SELECT code_combination_id, chart_of_accounts_id,
       segment1, segment2, segment3, segment4,
       account_type, enabled_flag, detail_posting_allowed_flag,
       detail_budgeting_allowed_flag, start_date_active, end_date_active
  FROM gl_code_combinations
 WHERE chart_of_accounts_id = :p_coa_id
   AND segment1 = :p_company
   AND segment3 = :p_natural_account;

-- 值集值和父子层级
SELECT flex_value_set_id, flex_value, description,
       enabled_flag, summary_flag, start_date_active, end_date_active
  FROM fnd_flex_values_vl
 WHERE flex_value_set_id = :p_value_set_id
 ORDER BY flex_value;
```

## 常见问题

- **APP-FND-00828/组合无效**：依次检查段值有效期、启用标志、Security Rule、Cross-Validation Rule 和 Dynamic Insertion。
- **科目可选但不能过账**：检查 CCID 和各段的 `ENABLED_FLAG`、`DETAIL_POSTING_ALLOWED_FLAG`、有效期。
- **年结结果异常**：检查 Natural Account 的 `ACCOUNT_TYPE`，已产生历史数据时不要直接修改基表。
- **新规则不生效**：确认规则分配到正确职责、弹性域已编译、用户已重新登录。
- **FSG 汇总不全**：检查父子范围、汇总组、账户类型和层级变更日期。

## 关联文档

- [Ledger、日历和期间](calendar-currency-period.md)
- [SLA 会计派生](sla.md)
- [GL 报表与导入](../04-gl/reporting-interfaces.md)

## 官方参考

- [Oracle General Ledger Implementation Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48747/toc.htm)
