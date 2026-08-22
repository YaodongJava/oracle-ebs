# 财务公共基础

## 适用范围与业务说明
作为 R12.2 财务实施的设计基线，覆盖组织、法人、账簿、COA、期间、MOAC、TCA、银行、审批、安全、序列和主数据治理。

## 配置与实施要点
按企业结构与法人、ASM/Ledger、COA/日历、OU/MOAC、安全/审批、TCA/银行、模块设置、端到端验证的顺序实施。

## 核心对象与诊断范围
组织、账簿、Code Combination、TCA、银行、Profile、FND、XLA。查询前在目标实例使用数据字典复核对象、列、状态值和访问权限；大表查询必须限制组织、账簿、日期或主键。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见风险与排查
在模块设置完成后才试图重构账簿或 COA；以单个名称而非主键和有效期治理基础数据。 排查时先确认业务状态、组织/账簿/期间和权限上下文，再逐层检查接口、会计和报表结果。

## 关联与官方依据
已有内容见[旧公共基础目录](../01-common/README.md)
