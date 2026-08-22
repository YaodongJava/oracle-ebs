# Oracle 官方资料与验证顺序

## 适用范围与业务说明
列出首选官方资料，并规定从产品意图到目标实例验证的证据链。

## 配置与实施要点
先看官方产品/实施/技术指南；再查目标实例 Integration Repository、eTRM、菜单、数据字典；最后在非生产真实场景验证。

## 核心对象与诊断范围
Oracle Documentation Library、Financials/Projects/SCM/Technology Guides、MOS SR 证据。查询前在目标实例使用数据字典复核对象、列、状态值和访问权限；大表查询必须限制组织、账簿、日期或主键。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见风险与排查
用非官方字段清单替代目标实例验证；未经授权引用或分发 MOS 专属内容。 排查时先确认业务状态、组织/账簿/期间和权限上下文，再逐层检查接口、会计和报表结果。

## 关联与官方依据
[Oracle Documentation Library](https://docs.oracle.com/cd/E26401_01/index.htm)｜[Financials](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)｜[Technology](https://docs.oracle.com/cd/E26401_01/nav/technology.htm)
