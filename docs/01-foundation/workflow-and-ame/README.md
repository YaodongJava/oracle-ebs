# Workflow、AME 与业务审批

## 适用范围与业务说明
Workflow 处理业务流程、通知、活动和 Business Event；AME 根据条件求得审批人，两者依赖交易属性与组织/人员关系。

## 配置与实施要点
识别状态和超时；定义 AME 属性/条件/规则；验证后台引擎和 Mailer；把工件纳入版本与 ADOP 发布。

## 核心对象与诊断范围
WF_ITEMS、WF_ITEM_ACTIVITY_STATUSES、AME 规则、人员/组织关系。查询前在目标实例使用数据字典复核对象、列、状态值和访问权限；大表查询必须限制组织、账簿、日期或主键。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见风险与排查
审批人错误时直接改 Workflow 表；通知未发时忽略流程、后台引擎、Mailer 和 SMTP 的不同断点。 排查时先确认业务状态、组织/账簿/期间和权限上下文，再逐层检查接口、会计和报表结果。

## 关联与官方依据
[技术治理专题](../../09-technical/workflow-ame-oaf-governance.md)
