# FND、Concurrent、Workflow 与运维常用表结构

## 业务说明

FND 是 EBS 应用对象字典、用户/职责、Profile、菜单、并发处理和 Lookup 的共享基础。Workflow 保存 Item/Activity/Notification 状态与延迟队列。运维 SQL 应优先查询状态与日志证据，不通过直接更新 FND/WF 表来“修复”请求或流程。

## FND 安全与设置表

| 表/视图 | 中文名 | 粒度/用途 | 关键字段 |
| --- | --- | --- | --- |
| `FND_USER` | EBS 用户 | 每个应用用户 | `USER_ID`, `USER_NAME`, `EMPLOYEE_ID`, `START_DATE/END_DATE` |
| `FND_RESPONSIBILITY` / `_TL` / `_VL` | 职责 | Responsibility+Application+语言 | `RESPONSIBILITY_ID`, `APPLICATION_ID`, `MENU_ID`, `REQUEST_GROUP_ID` |
| `FND_USER_RESP_GROUPS_DIRECT` | 用户直接职责分配 | User+Responsibility+有效期 | `USER_ID`, `RESPONSIBILITY_ID`, `RESPONSIBILITY_APPLICATION_ID` |
| `FND_MENUS` | 菜单 | 每个 Menu | `MENU_ID`, `MENU_NAME` |
| `FND_MENU_ENTRIES` | 菜单条目 | Menu+Sequence | `MENU_ID`, `ENTRY_SEQUENCE`, `SUB_MENU_ID`, `FUNCTION_ID` |
| `FND_FORM_FUNCTIONS` | 应用功能 | 每个 Function | `FUNCTION_ID`, `FUNCTION_NAME`, `FORM_ID`, `PARAMETERS` |
| `FND_PROFILE_OPTIONS` / `_TL` / `_VL` | Profile 定义 | 每个 Profile | `PROFILE_OPTION_ID`, `PROFILE_OPTION_NAME` |
| `FND_PROFILE_OPTION_VALUES` | Profile 设置值 | Profile+设置层级 | `LEVEL_ID`, `LEVEL_VALUE`, `LEVEL_VALUE_APPLICATION_ID` |
| `FND_LOOKUP_VALUES` / `_VL` | Lookup 代码与含义 | Lookup Type+Code+Language | `LOOKUP_TYPE`, `LOOKUP_CODE`, `MEANING` |
| `FND_APPLICATION` / `_TL` / `_VL` | EBS 应用 | 每个 Application | `APPLICATION_ID`, `APPLICATION_SHORT_NAME` |

### Profile `LEVEL_ID`

| 值 | 层级 | `LEVEL_VALUE` 含义 |
| --- | --- | --- |
| `10001` | Site | 通常不使用具体业务 ID |
| `10002` | Application | `APPLICATION_ID` |
| `10003` | Responsibility | `RESPONSIBILITY_ID`，并结合 `LEVEL_VALUE_APPLICATION_ID` |
| `10004` | User | `USER_ID` |

最终 Profile 值优先级通常为 User > Responsibility > Application > Site。但 Profile 可更新层级受其定义限制；诊断时应同时查显式设置和 `FND_PROFILE.VALUE` 运行时最终值。

## Concurrent Processing 表

| 表/视图 | 中文名 | 粒度/用途 | 关键字段 |
| --- | --- | --- | --- |
| `FND_CONCURRENT_PROGRAMS` / `_TL` / `_VL` | 并发程序定义 | Program+Application+语言 | `CONCURRENT_PROGRAM_ID`, `EXECUTABLE_ID`, `OUTPUT_FILE_TYPE` |
| `FND_EXECUTABLES` | 可执行对象 | 每个 Executable | `EXECUTABLE_ID`, `EXECUTION_METHOD_CODE`, `EXECUTION_FILE_NAME` |
| `FND_CONCURRENT_REQUESTS` | 并发请求 | 每次请求 | `REQUEST_ID`, `PHASE_CODE`, `STATUS_CODE`, Program/Application IDs |
| `FND_CONCURRENT_QUEUES` / `_VL` | 并发管理器队列 | 每个 Manager Queue | `CONCURRENT_QUEUE_ID`, `RUNNING_PROCESSES`, `MAX_PROCESSES` |
| `FND_CONCURRENT_PROCESSES` | 并发 Worker 进程 | 每个 Manager Process | `CONCURRENT_PROCESS_ID`, `CONCURRENT_QUEUE_ID`, `PROCESS_STATUS_CODE` |
| `FND_REQUEST_SETS` | 请求集 | 每个 Request Set | `REQUEST_SET_ID`, `REQUEST_SET_NAME` |
| `FND_REQUEST_SET_STAGES` | 请求集阶段 | Set+Stage | `REQUEST_SET_STAGE_ID`, `STAGE_NAME` |
| `FND_RUN_REQUESTS` | 请求子请求关系/打印选项 | Parent Request 执行元数据 | `PARENT_REQUEST_ID` 等 |

### `FND_CONCURRENT_REQUESTS`

| 字段 | 中文名 | 业务说明 |
| --- | --- | --- |
| `REQUEST_ID` | 请求 ID | 用户、日志、DB Session、子请求跟踪的核心键 |
| `PARENT_REQUEST_ID` | 父请求 ID | Request Set/父程序提交子请求时使用；根请求通常为特定默认值/NULL，以 eTRM 为准 |
| `PHASE_CODE` | 请求阶段 | 常见 `P`等待、`R`运行、`C`完成、`I`非活动；必须与 Status 组合解码 |
| `STATUS_CODE` | 请求状态 | Normal/Warning/Error/On Hold/No Manager/Cancelled/Terminated 等，单字母在不同 Phase 下解读，使用 `CP_STATUS_CODE` Lookup |
| `ARGUMENT_TEXT` | 参数文本 | 位置受 Program Parameter 定义决定，不可只凭逗号位置猜业务含义 |
| `ACTUAL_START/COMPLETION_DATE` | 实际开始/完成 | 计算运行时长；等待时长还要用 Requested Start Date |
| `CONCURRENT_PROCESS_ID` | Worker 进程 ID | 关联 Concurrent Process，用于定位 Manager/OS Process |
| `ORACLE_PROCESS_ID` | 数据库进程线索 | 值在 RAC/架构下需结合 Node/Session 使用 |
| `LOGFILE_NAME/OUTFILE_NAME` | 日志/输出路径 | 应通过 EBS 标准页面/授权访问，路径可受节点和迁移影响 |

## Workflow 表

| 表 | 中文名 | 粒度/用途 |
| --- | --- | --- |
| `WF_ITEMS` | Workflow 项实例 | `ITEM_TYPE + ITEM_KEY` 每个流程实例 |
| `WF_ITEM_ACTIVITY_STATUSES` | Workflow 活动状态历史 | Item+Process Activity+执行历史 |
| `WF_ITEM_ACTIVITY_STATUSES_H` | Workflow 已归档活动历史 | 已移入历史的活动状态 |
| `WF_NOTIFICATIONS` | Workflow 通知 | 每条审批/通知 |
| `WF_NOTIFICATION_ATTRIBUTES` | 通知属性 | Notification+Attribute |
| `WF_DEFERRED` | Workflow 延迟队列 | 等待 Background Engine/事件处理 |
| `WF_ERROR` | Workflow 错误队列 | 处理异常消息 |
| `WF_LOCAL_ROLES` / `WF_LOCAL_USER_ROLES` | Workflow 角色/成员 | 用户、职责、审批角色目录映射 |

`WF_ITEM_ACTIVITY_STATUSES.ACTIVITY_STATUS` 常见 Notified、Active、Complete、Error、Deferred、Suspended 等含义，必须结合 Activity Result、Error Name/Message、Begin/End Date 和归档表查询。`WF_NOTIFICATIONS.STATUS/MAIL_STATUS` 分别表示业务通知状态与邮件发送状态，Open 通知不等于邮件必然未发出。

## 状态解码 SQL

```sql
SELECT lookup_type, lookup_code, meaning, description
  FROM fnd_lookup_values_vl
 WHERE lookup_type IN ('CP_PHASE_CODE', 'CP_STATUS_CODE')
 ORDER BY lookup_type, lookup_code;

SELECT fcr.request_id,
       phase.meaning phase_name,
       status.meaning status_name,
       fcr.requested_start_date,
       fcr.actual_start_date,
       fcr.actual_completion_date
  FROM fnd_concurrent_requests fcr
  LEFT JOIN fnd_lookup_values_vl phase
    ON phase.lookup_type = 'CP_PHASE_CODE'
   AND phase.lookup_code = fcr.phase_code
  LEFT JOIN fnd_lookup_values_vl status
    ON status.lookup_type = 'CP_STATUS_CODE'
   AND status.lookup_code = fcr.status_code
 WHERE fcr.request_id = :p_request_id;
```

> Lookup 视图可包含多语言/多行有效性，如返回重复，增加 `LANGUAGE = USERENV('LANG')`、启用标志和有效日条件。

## 官方参考

- [Oracle E-Business Suite eTRM User's Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/f53031/)
- [Oracle E-Business Suite Concepts R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e22949/)
