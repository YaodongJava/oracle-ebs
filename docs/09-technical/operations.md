# 性能调优、权限审计与 R12.2 生产运维

## R12.2 运维边界

- 应用层管理节点、WebLogic/OHS、Forms、Concurrent Processing、Workflow Mailer、OPP 和集成服务。
- R12.2 使用 Online Patching（adop）的 Prepare、Apply、Finalize、Cutover、Cleanup 周期，并基于 Run/Patch File System 与 EBR。
- 管理脚本和环境文件必须在正确节点/文件系统执行；不在未确认的环境中混用 run/patch edition。
- 数据库、中间件、EBS Codelevel/ETCC、Java 和浏览器兼容性应按 Oracle 证证矩阵和 Support 建议维护。

## 性能诊断法

```text
Business Symptom
→ User/Responsibility/Function/Request + Exact Time
→ Tier（Browser/OHS/OAF/Forms/Concurrent/DB/External）
→ Session/Request/SQL ID/Trace
→ Wait/Plan/Rows/Locks/IO/CPU
→ Reproduce → Fix → Regression → Baseline
```

1. 从用户、职责、功能/请求 ID、参数和精确时间段开始，避免“系统很慢”式无边界排查。
2. 使用 OAM/标准 Diagnostics/Trace 有限时间采样；AWR/ASH/SQL Monitor 使用需符合数据库许可。
3. 优先修正 SQL 选择性、Join、Bind、统计信息和数据倾斜，不盲目加 Hint/Index。
4. 对并发程序将性能与数据量、参数、Manager Queue/Processes、Incompatibility 和 OPP 分开分析。

## 安全和审计

- 定期复核用户、职责、失效日期、User-level Profile、特权职责、共享账号和服务账号。
- 实施 SoD：Supplier/Bank Change、Invoice Approval、Payment Creation/Release、Journal Create/Approve/Post、User Admin 分离。
- 保护 APPS/APPLSYS/SYSTEM 和 WebLogic 管理凭据，轮换密码并验证下游集成。
- 日志/报表脱敏银行账号、税号、个人信息和 Token；限制输出保留和下载权限。

## 实用 SQL

```sql
-- 失效对象；不要无差别全库重编译
SELECT owner, object_type, COUNT(*) invalid_count
  FROM dba_objects
 WHERE status = 'INVALID'
 GROUP BY owner, object_type
 ORDER BY owner, object_type;

-- 近期失败请求
SELECT request_id, program_application_id, concurrent_program_id,
       phase_code, status_code, actual_start_date,
       actual_completion_date, argument_text
  FROM fnd_concurrent_requests
 WHERE actual_start_date >= SYSDATE - :p_days
   AND status_code IN ('E','G')
 ORDER BY actual_start_date DESC;

-- 有效用户层 Profile 覆盖
SELECT fu.user_name, fpo.user_profile_option_name,
       fpov.profile_option_value
  FROM fnd_profile_option_values fpov
  JOIN fnd_profile_options_vl fpo
    ON fpo.application_id = fpov.application_id
   AND fpo.profile_option_id = fpov.profile_option_id
  JOIN fnd_user fu ON fu.user_id = fpov.level_value
 WHERE fpov.level_id = 10004
   AND NVL(fu.end_date, SYSDATE + 1) > SYSDATE
 ORDER BY fu.user_name, fpo.user_profile_option_name;
```

> `DBA_OBJECTS` 需 DBA 权限；普通账号使用 `ALL_OBJECTS`。生产性能诊断必须有时间范围，避免高成本全库查询。

## 常见问题

- adop 卡住：从当前 Phase/Worker 日志和 adopscanlog 定位首个错误，检查节点、空间、ETCC、无效对象、ADZDPATCH/并发程序，不盲目 `abort/cleanup`。
- Forms/OAF 单点故障：比较用户/职责/功能和节点，查 OHS/WebLogic/Managed Server 日志、会话和近期变更。
- Workflow Mailer 不发信：查 Component Status、Inbound/Outbound Account、SMTP/IMAP/TLS、Notification Mail Status、Deferred/Error Queue 和日志。
- 补丁后性能回退：比较 SQL Plan/Stats/Codelevel、无效对象和定制兼容，使用可重现证据建 SR。

## 关联

- [Security](../01-common/security.md)
- [Concurrent Processing](concurrent-programs.md)
- [Customization](customization.md)

## 官方参考

- [Oracle E-Business Suite Concepts R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e22949/)
- [Oracle E-Business Suite R12.2 Documentation Library](https://docs.oracle.com/cd/E26401_01/index.htm)
