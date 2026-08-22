# 日历、币种、汇率与期间控制

## 原理

Ledger 的核心属性包括 Accounting Calendar、Period Type 和 Ledger Currency。子账期间与 GL 期间分开控制：AP/AR/PO/INV/FA 可以在同一 GL 期间下处于不同状态，关闭顺序应从业务子账到 GL。

```text
Transaction Currency
  → Conversion Type + Conversion Date + Daily Rate
  → Ledger Currency（Accounted Amount）
  → Reporting Currency / Secondary Ledger（如已配置）
```

`GL_DAILY_RATES` 保存日汇率，汇率类型在 `GL_DAILY_CONVERSION_TYPES`。日记账头的汇率日期/类型/汇率决定折算；重估（Revaluation）处理外币账户未实现汇兑差额，折算（Translation）用于将账簿余额转为报告币种。

## 配置顺序

1. 定义 Calendar 和 Period Type，一次生成并仔细检查期间日期、期间号和年度。
2. 确认币种、精度、最小账户单位和启用日期。
3. 定义 Conversion Type，建立汇率获取、审批和导入控制。
4. 完成 Ledger 设置后打开首个 GL 期间，再按模块打开子账期间。
5. 月结时先处理接口/未过账交易、创建会计、转 GL 并对账，再关闭子账，最后关闭 GL。

## 常用 SQL

```sql
-- Ledger 日历和期间
SELECT gl.ledger_id, gl.name, gl.currency_code,
       gl.period_set_name, gl.accounted_period_type,
       gps.period_name, gps.start_date, gps.end_date,
       gps.closing_status
  FROM gl_ledgers gl
  JOIN gl_period_statuses gps
    ON gps.application_id = 101
   AND gps.set_of_books_id = gl.ledger_id
 WHERE gl.ledger_id = :p_ledger_id
 ORDER BY gps.start_date;

-- 某日汇率
SELECT from_currency, to_currency, conversion_date,
       conversion_type, conversion_rate, status_code
  FROM gl_daily_rates
 WHERE conversion_date = :p_conversion_date
   AND from_currency = :p_from_currency
   AND to_currency = :p_to_currency;

-- 各应用期间状态；APPLICATION_ID 需联接 FND_APPLICATION 解读
SELECT gps.application_id, fa.application_short_name,
       gps.set_of_books_id, gps.period_name, gps.closing_status,
       gps.start_date, gps.end_date
  FROM gl_period_statuses gps
  JOIN fnd_application fa ON fa.application_id = gps.application_id
 WHERE gps.set_of_books_id = :p_ledger_id
   AND gps.period_name = :p_period_name;
```

## 排查

- **缺少汇率**：核对 From/To Currency、Conversion Type/Date、直接或反向汇率、导入状态和精度。
- **日期不在开放期间**：检查交易日期对应的子账期间，不要只检查 GL。
- **期间无法关闭**：查待处理接口、未验证/未会计交易、未转 GL 会计和未完成并发请求。
- **会计金额不等**：比较交易汇率与公司标准汇率，检查手工覆盖和 SLA 舍入。
- **错误关期**：优先用标准 Reopen 流程并评估已发布报表，禁止更新 `GL_PERIOD_STATUSES`。

## 关联文档

- [GL 期间关闭](../04-gl/close-reports.md)
- [AP 会计与月结](../02-ap/accounting-close-reports.md)
- [AR 会计与月结](../03-ar/accounting-close-reports.md)
