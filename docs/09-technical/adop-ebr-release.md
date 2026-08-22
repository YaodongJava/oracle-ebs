# R12.2 Online Patching、EBR 与发布治理

## 核心原则

R12.2 使用 Online Patching（ADOP）与 Edition-Based Redefinition（EBR）降低在线补丁对业务的影响，但并不意味着自定义对象可以任意创建、修改或直接在生产库修复。每次交付均应识别对象类型、Edition 属性、依赖、部署方式、回滚策略和验证证据。

## ADOP 生命周期

```text
Prepare → Apply → Finalize → Cutover → Cleanup
                         ↘ 必要时按 Oracle 受支持流程 Abort/Recover
```

| 阶段 | 目标 | 发布控制 |
| --- | --- | --- |
| Prepare | 建立 Patch 文件系统与同步前提 | 检查环境健康、磁盘、服务、已有会话和未完成周期 |
| Apply | 应用 Oracle/自定义补丁 | 记录 patch 日志、失败对象、依赖、重跑范围 |
| Finalize | 将耗时工作前置 | 评估业务窗口、并发请求、OPP/Workflow/接口影响 |
| Cutover | 切换运行文件系统/Edition | 设置冻结、变更窗口、健康检查、业务签字与回退判定 |
| Cleanup | 清理旧版定义 | 归档日志、更新基线、完成回归与配置对比 |

## 自定义交付清单

1. 源码、数据库对象、Forms/OAF、报表、FND 注册、Workflow、Profile/Lookup 分别建立受版本控制的迁移工件。
2. 自定义数据库对象使用独立 schema、最小授权和受支持 synonym/grant；不得直接修改 Oracle 标准对象。
3. 在开发、测试、预生产完成 ADOP 演练，验证在线/切换窗口、并发程序、接口、会计和核心报表。
4. 发布包应具有唯一版本、依赖清单、校验 SQL、回滚/补偿方案、日志位置和责任人。

## 只读诊断 SQL

```sql
-- 检查自定义对象的有效性；不要在生产直接编译 Oracle 标准对象作为常规修复。
select owner,
       object_type,
       object_name,
       status,
       last_ddl_time
  from all_objects
 where owner = upper(:p_custom_schema)
   and status <> 'VALID'
 order by object_type, object_name;

-- 用数据字典复核对象/列，再决定补丁脚本的兼容性。
select owner, table_name, column_name, data_type, data_length
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见问题

- 只在 Run 文件系统修改文件：下一个 ADOP 周期可能被覆盖，且无法形成可部署基线。
- 直接在生产补数据或编译：可能绕过 Edition、审计和回退设计；应改用受支持 API、补丁或 Oracle Support 方案。
- Cutover 后接口/并发异常：按发布清单检查服务、context、custom library/报表、注册定义、日志与依赖，而不是盲目重跑整批业务。

## 官方参考

- [Oracle E-Business Suite Maintenance Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22954/toc.htm)
- [Oracle E-Business Suite Technology Documentation](https://docs.oracle.com/cd/E26401_01/nav/technology.htm)
