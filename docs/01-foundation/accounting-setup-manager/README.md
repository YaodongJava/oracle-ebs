# Accounting Setup Manager

## 适用范围与业务说明
ASM 用于定义会计设置及法人/账簿关系；Primary Ledger 的四要素是 COA、Calendar、Currency 与 Accounting Method。

## 配置与实施要点
确定四要素并创建 Primary Ledger；分配法人、Balancing Segment、OU 与 SLA 选项；按需设计 Secondary Ledger、Reporting Currency、Ledger Set。

## 核心对象与诊断范围
GL_LEDGERS、SLA 选项、Ledger Set、Reporting Currency。查询前在目标实例使用数据字典复核对象、列、状态值和访问权限；大表查询必须限制组织、账簿、日期或主键。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见风险与排查
把 Secondary Ledger 与 Reporting Currency 混为一谈；子账无法会计时直接修改 XLA。 排查时先确认业务状态、组织/账簿/期间和权限上下文，再逐层检查接口、会计和报表结果。

## 关联与官方依据
[Oracle Financials Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48783/toc.htm)
