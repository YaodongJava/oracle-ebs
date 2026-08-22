# 日历、币种、汇率与期间

## 适用范围与业务说明
会计日历控制可记账期间；币种、Rate Type、Conversion Date 和 Daily Rate 控制外币换算；模块交易日期与会计日期可能不同。

## 配置与实施要点
定义 Period Type/Calendar/未来年份；配置币种和汇率；设置各模块期间职责；测试期初、月末、跨期、外币和重估。

## 核心对象与诊断范围
GL_PERIODS、GL_PERIOD_STATUSES、GL_DAILY_RATES。查询前在目标实例使用数据字典复核对象、列、状态值和访问权限；大表查询必须限制组织、账簿、日期或主键。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见风险与排查
忽略模块期间与 GL 期间差异；把缺失汇率、错误 Rate Type 和期间关闭混为同类问题。 排查时先确认业务状态、组织/账簿/期间和权限上下文，再逐层检查接口、会计和报表结果。

## 关联与官方依据
[既有日历专题](../../01-common/calendar-currency-period.md)
