# 文档、SQL 与示例规范

## 适用范围与业务说明
统一文件命名、内容结构、SQL 安全和接口代码质量，保障文档可持续维护。

## 配置与实施要点
文件名使用小写 kebab-case；每篇写明范围、原理、配置、数据、诊断、问题和依据；SQL 用绑定变量并限制组织/账簿/日期。

## 核心对象与诊断范围
APPS 只读查询、业务主键、ORG_ID、LEDGER_ID、请求号、接口批次号。查询前在目标实例使用数据字典复核对象、列、状态值和访问权限；大表查询必须限制组织、账簿、日期或主键。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见风险与排查
提供直接更新 Oracle 业务表的脚本；未限制大表扫描或未复核列定义。 排查时先确认业务状态、组织/账簿/期间和权限上下文，再逐层检查接口、会计和报表结果。

## 关联与官方依据
[生产安全边界](safety-and-production-boundaries.md)
