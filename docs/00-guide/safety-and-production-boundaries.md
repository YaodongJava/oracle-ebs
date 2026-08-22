# 生产安全与支持边界

## 适用范围与业务说明
定义生产环境的最小安全原则：查询优先、标准入口写入、变更可回退、敏感数据最小暴露。

## 配置与实施要点
使用页面、公开 API、Open Interface、标准并发程序或客户自定义对象写入；发布遵循 EBR、ADOP、双文件系统和受支持扩展点。

## 核心对象与诊断范围
日志、请求号、接口批次、业务键、会计期间、对象版本、审批记录。查询前在目标实例使用数据字典复核对象、列、状态值和访问权限；大表查询必须限制组织、账簿、日期或主键。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见风险与排查
直接 DML EBS 业务/会计/FND/Workflow 运行时表；用总账手工分录长期掩盖子账问题。 排查时先确认业务状态、组织/账簿/期间和权限上下文，再逐层检查接口、会计和报表结果。

## 关联与官方依据
[R12.2 技术目录](../10-technical/README.md)
