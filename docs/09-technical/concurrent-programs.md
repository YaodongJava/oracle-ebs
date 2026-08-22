# Concurrent Program、请求集与日志排错

## 架构

```text
Submit Request → FND_CONCURRENT_REQUESTS(Pending)
→ ICM/Service Manager → Target Manager
→ Worker Process → Executable
→ Log/Output → OPP（XML/PDF/Excel 后处理）
→ Completed Normal/Warning/Error
```

Concurrent Program 关联 Executable、Parameters/Value Sets、Incompatibility、Request Group、Output Format、Printer/Style。Request Set 用 Stage 和 Link 组合多程序，需考虑失败分支、参数默认和重启。

R12.2 在 Online Patching 时使用 `ADZDPATCH` 协调不兼容程序；不要为了让 adop 继续而随意终止 ICM/ADZDPATCH。

## SQL

```sql
SELECT r.request_id, r.parent_request_id,
       cp.user_concurrent_program_name,
       r.phase_code, r.status_code, r.hold_flag,
       r.requested_start_date, r.actual_start_date,
       r.actual_completion_date, r.argument_text,
       r.concurrent_process_id, r.oracle_process_id,
       r.logfile_name, r.outfile_name
  FROM fnd_concurrent_requests r
  JOIN fnd_concurrent_programs_vl cp
    ON cp.concurrent_program_id = r.concurrent_program_id
   AND cp.application_id = r.program_application_id
 WHERE r.request_id = :p_request_id;

SELECT q.concurrent_queue_name, q.user_concurrent_queue_name,
       q.node_name, q.running_processes, q.max_processes,
       q.enabled_flag, q.control_code
  FROM fnd_concurrent_queues_vl q
 ORDER BY q.user_concurrent_queue_name;

SELECT request_id, phase_code, status_code,
       requested_start_date, actual_start_date,
       actual_completion_date
  FROM fnd_concurrent_requests
 WHERE phase_code IN ('P','R')
 ORDER BY requested_start_date;
```

## 排错

- Pending/Standby：查 Requested Start、Hold、Manager Specialization、Incompatibility、Parent Request、Manager Processes、Node/Work Shift。
- Running 过久：查日志最后进度、DB Session/Wait/Blocking、参数数据量和子请求；先评估 Cancel/Terminate 业务后果。
- Completed Error：从日志第一个有意义的 ORA-/APP-/Exception 开始，不只看最后的通用错误。
- OPP 超时/无输出：查 Data Engine 输出大小、Template/Locale、OPP Service/Thread、Heap、Temporary Directory、Font 和巨大 XML。
- Manager 异常：通过 OAM/标准脚本管理，保留 ICM/Manager/Service Manager 日志，不直接改 FND Queue/Request 状态。

## 关联

- [Operations](operations.md)
- [Integration](integration.md)
