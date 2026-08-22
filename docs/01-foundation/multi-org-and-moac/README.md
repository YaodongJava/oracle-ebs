# Multiple Organizations 与 MOAC

## 适用范围与业务说明
MOAC 让职责在受控范围访问多个 OU，但不改变 ORG_ID 数据隔离，也不替代 Ledger/Data Access Set。

## 配置与实施要点
定义 Security Profile；设置 MO Security Profile 和默认 OU；运行 Security List Maintenance；分别测试单/多 OU、报表和并发上下文。

## 核心对象与诊断范围
ORG_ID、Security Profile、Profile Options、MO_GLOBAL、FND_GLOBAL。查询前在目标实例使用数据字典复核对象、列、状态值和访问权限；大表查询必须限制组织、账簿、日期或主键。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见风险与排查
固定 ORG_ID 或扩大职责掩盖访问问题；自定义会话未初始化导致页面/SQL/API 结果不一致。 排查时先确认业务状态、组织/账簿/期间和权限上下文，再逐层检查接口、会计和报表结果。

## 关联与官方依据
[既有多组织详解](../../01-common/organization.md)
