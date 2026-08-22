# 安全与数据访问

## 适用范围与业务说明
User、Responsibility、Menu、Function、Request Group、Profile、Data Access Set、MOAC 与数据库授权共同决定访问权限和职责分离。

## 配置与实施要点
按岗位最小授权；分离主数据、交易、付款、日记账、审批、关账和系统管理；定期复核用户、职责、代理和特权 Profile。

## 核心对象与诊断范围
FND_USER、FND_USER_RESP_GROUPS_DIRECT、FND_PROFILE_OPTION_VALUES、Data Access Set。查询前在目标实例使用数据字典复核对象、列、状态值和访问权限；大表查询必须限制组织、账簿、日期或主键。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见风险与排查
以 System Administrator 权限运行日常操作；只因能查到数据就推断可以创建、审批或付款。 排查时先确认业务状态、组织/账簿/期间和权限上下文，再逐层检查接口、会计和报表结果。

## 关联与官方依据
[既有安全专题](../../01-common/security.md)
