# PL/SQL、Forms、Personalization 与 OAF 定制

## R12.2 定制原则

- 优先级：标准设置 > Personalization/Extension > Published API/Open Interface > 经评审的定制；禁止修改 Oracle seeded 源码和基表。
- 自定义对象使用客户前缀/Schema，通过 APPS Synonym/Grant 接入，所有 DDL 满足 Edition-Based Redefinition（EBR）。
- 数据库对象变更通过 `adop` Online Patching 周期发布，开发环境执行一致性/在线补丁检查。
- Forms Personalization 用于可见性、默认、验证和菜单动作；CUSTOM.pll 仅在 Personalization 无法实现时使用。
- OAF 使用 Personalization 或 Controller Extension，不修改 seeded XML/Java；扩展需评估升级/补丁兼容。

## PL/SQL 标准

1. 公开 API 前初始化 FND/MOAC 上下文，传入 `p_api_version/p_init_msg_list/p_commit`。
2. 尊重 API 交易边界，默认由调用者 Commit/Rollback，不在底层工具函数隐式提交。
3. 读取 `x_return_status/x_msg_count/x_msg_data` 及 Message Stack，日志记录业务键而非敏感数据。
4. SQL 使用 Bind、明确组织/账簿条件、批量处理和可重启设计，避免 Row-by-row 大批量处理。

## 诊断 SQL

```sql
SELECT owner, object_name, object_type, status, edition_name
  FROM all_objects
 WHERE object_name = UPPER(:p_object_name)
 ORDER BY owner, object_type;

SELECT owner, name, type, line, position, text
  FROM all_errors
 WHERE name = UPPER(:p_object_name)
 ORDER BY owner, type, sequence;

SELECT lookup_type, lookup_code, meaning, enabled_flag,
       start_date_active, end_date_active
  FROM fnd_lookup_values_vl
 WHERE lookup_type = :p_lookup_type
 ORDER BY lookup_code;
```

## 排错

- 补丁后定制失效：检查 Invalid Objects、API Signature/View Columns 变更、Synonym/Grant、OAF Extension 兼容和 adop 日志。
- Personalization 不生效：查 Function/Form/Page Context、Level/Sequence/Condition、字段名、缓存，使用 Diagnostics 验证实际项。
- API 返回 Error：打印完整 Message Stack、输入 ID/OU/User/Responsibility、前置状态，不只记 `SQLERRM`。
- ORA-04061/4068：可能是 Package 重编译导致会话状态失效，重建会话并检查发布是否遵循 Online Patching。

## 关联

- [Data Model](data-model.md)
- [Operations](operations.md)
