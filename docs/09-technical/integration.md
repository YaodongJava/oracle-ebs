# 开放接口、API、报表与数据迁移

## 选型原则

| 方式 | 适用场景 | 控制要点 |
| --- | --- | --- |
| Open Interface + Import | 高吞吐异步批量 | Group/Source、拒绝表、幂等、可重启 |
| Published PL/SQL API | 同步单笔/小批量 | FND/MOAC Context、Message Stack、Commit 边界 |
| Business Event/Workflow | 事件驱动 | Event Key、Subscription Phase、重试/错误队列 |
| Integrated SOA Gateway | SOAP/REST 暴露 EBS 接口 | Authentication、Grant、MOAC、限流/审计 |
| XML Gateway/EDI | B2B 标准消息 | Trading Partner、Map、Transaction Type、ACK |
| BI Publisher/Concurrent | 报表/文件 | Data Definition、Template、Locale、OPP/Output |

## 接口分层

```text
Source → Landing（原始不可变）
→ Staging（标准化/验证/幂等）
→ EBS Interface/API（标准业务验证）
→ Base Transaction → Accounting
→ Reconciliation/Acknowledgement/Archive
```

每条数据保存 Source System、External Key、Batch/Correlation ID、ORG_ID/Ledger、Payload Hash、Status、Retry Count、Request ID、EBS Transaction ID、Error Code/Message。技术重试必须幂等，业务驳回需修正后重提，不应无限自动重试。

## SQL

```sql
-- 并发请求跟踪
SELECT request_id, parent_request_id, phase_code, status_code,
       argument_text, actual_start_date, actual_completion_date,
       logfile_name, outfile_name
  FROM fnd_concurrent_requests
 WHERE request_id = :p_request_id;

-- Workflow 事件/活动跟踪；生产查询必须限定 Item Type/Key
SELECT item_type, item_key, begin_date, end_date,
       root_activity, root_activity_version
  FROM wf_items
 WHERE item_type = :p_item_type
   AND item_key = :p_item_key;

SELECT item_type, item_key, process_activity,
       activity_status, activity_result_code,
       begin_date, end_date, error_name, error_message
  FROM wf_item_activity_statuses
 WHERE item_type = :p_item_type
   AND item_key = :p_item_key
 ORDER BY begin_date;
```

## 迁移清单

1. 数据 Profile/清洗/映射，确定历史深度和未结业务边界。
2. 按主数据→期初余额→未结交易→历史的依赖顺序导入。
3. 多轮 Mock，每轮保存输入、拒绝、数量/金额对账、性能和时间。
4. Cutover 冻结、Delta、业务签字和回退标准必须事先定义。

## 排错

- 先定位 Landing/Staging/EBS Interface/Import/Base/Accounting 断点，再查对应状态。
- 重复：检查 External Key/Hash、超时后重试、EBS 已成功但 ACK 失败的情况。
- 部分成功：对每条保存 EBS ID/Status，仅重试失败项，不重放整批成功数据。
- 性能：使用批次、Bind/Array Processing、合理 Commit Size、并发限流，避免过度 API 单行循环。

## 关联

- [AP Interface](../02-ap/interfaces-troubleshooting.md)
- [AR Interface](../03-ar/interfaces-troubleshooting.md)
- [Concurrent Processing](concurrent-programs.md)
