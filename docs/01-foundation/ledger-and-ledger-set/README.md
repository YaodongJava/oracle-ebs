# Ledger、Ledger Set 与报告币种

## 适用范围与业务说明
Primary Ledger 是主会计记录；Ledger Set 用于集合访问/报告；Secondary Ledger 支持不同会计表示；Reporting Currency 支持辅助币种。

## 配置与实施要点
同时评估法人、币种、COA、日历、SLA、公司间、报告、合并和访问；为每个业务场景设计转换、传输与对账路径。

## 核心对象与诊断范围
GL_LEDGERS、Ledger Set、Data Access Set、Journal Conversion Rule。查询前在目标实例使用数据字典复核对象、列、状态值和访问权限；大表查询必须限制组织、账簿、日期或主键。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见风险与排查
为权限隔离新建账簿；跨账簿差异时未检查转换规则、期间和汇率。 排查时先确认业务状态、组织/账簿/期间和权限上下文，再逐层检查接口、会计和报表结果。

## 关联与官方依据
[既有账簿与组织内容](../../01-common/organization.md)
