# 文档与会计序列

## 适用范围与业务说明
Document Sequence 管理业务单据编号；Accounting/Reporting Sequence 支持会计与法定报告追溯，需按法规和业务上下文定义。

## 配置与实施要点
定义序列、范围、有效期、自动/手工规则；按应用/账簿/法人/来源/类别分配；测试跨期、并发、作废和例外。

## 核心对象与诊断范围
序列定义、序列分配、应用/账簿上下文、审计记录。查询前在目标实例使用数据字典复核对象、列、状态值和访问权限；大表查询必须限制组织、账簿、日期或主键。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见风险与排查
通过直接更新序列表补号；忽略取消、缺号、重开期间和并发编号的法规口径。 排查时先确认业务状态、组织/账簿/期间和权限上下文，再逐层检查接口、会计和报表结果。

## 关联与官方依据
[Financials Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48783/toc.htm)
