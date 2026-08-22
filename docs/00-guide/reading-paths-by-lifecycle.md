# 按实施生命周期阅读路径

## 适用范围与业务说明
覆盖 Assessment、Blueprint、Build、Test、Cutover 和 Run，确保每个阶段都有可审计输入、输出和签字。

## 配置与实施要点
范围/许可证评估；企业结构、流程、会计和安全蓝图；构建与迁移；CRP/SIT/UAT；切换对账；Hypercare 和持续运维。

## 核心对象与诊断范围
CEMLI 清单、配置工作簿、迁移批次、测试脚本、Go/No-Go、关账包。查询前在目标实例使用数据字典复核对象、列、状态值和访问权限；大表查询必须限制组织、账簿、日期或主键。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见风险与排查
没有真实会计和接口回归便上线；上线后以手工改数据处理设计缺陷。 排查时先确认业务状态、组织/账簿/期间和权限上下文，再逐层检查接口、会计和报表结果。

## 关联与官方依据
[实施与运维](../11-implementation-operations/README.md)
