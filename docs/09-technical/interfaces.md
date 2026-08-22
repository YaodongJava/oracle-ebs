# Oracle EBS 技术接口实现手册

## 1. 接口方式选型

| 方式 | 适合场景 | 调用/处理特点 | 主要风险控制 |
| --- | --- | --- | --- |
| Open Interface + Import | 高吞吐异步批量 | 先写接口表，再跑标准 Import | 幂等、Group ID、错误表、对账 |
| Published PL/SQL API | 同步单笔/小批量 | 完整业务验证和 Message Stack | FND/MOAC Context、Commit 语义 |
| ISG REST/SOAP | 对外暴露 EBS 标准能力 | Integration Repository 部署与授权 | HTTPS、Grant、限流、WADL/WSDL 版本 |
| Concurrent Program REST | 长任务/报表/批处理 | REST 仅 POST，异步返回 Request ID | 参数位置、轮询、超时、并发队列 |
| Business Event | EBS 业务事件通知 | 本地/Deferred Subscription | Event Key 幂等、Error Queue |
| Service Invocation Framework | EBS 调外部 SOAP/REST | Workflow BES + Java Deferred Agent | 证书、凭据、回调、Invocation Monitor |
| XML Gateway/EDI | B2B 标准报文 | Trading Partner、Map、ACK | 版本、字符集、签名、重复报文 |

Oracle R12.2 官方说明：PL/SQL API、Concurrent Program、Business Service Object 可暴露为 SOAP/REST；Inbound Open Interface REST 支持 POST/GET/PUT/DELETE；Concurrent Program REST 只支持 POST。自定义 Open Interface 表/视图不能直接作为 Integration Repository 的 Custom Interface 类型发布，应使用自定义 PL/SQL API 或标准接口暂存层。

## 2. 自定义 Concurrent Program Worker

### 2.1 Package Specification

```sql
CREATE OR REPLACE PACKAGE xxint_worker_pkg AUTHID DEFINER AS
  PROCEDURE main(
    errbuf           OUT VARCHAR2,
    retcode          OUT NUMBER,
    p_interface_code IN  VARCHAR2,
    p_batch_size     IN  NUMBER DEFAULT 500
  );
END xxint_worker_pkg;
/
```

### 2.2 Package Body

```sql
CREATE OR REPLACE PACKAGE BODY xxint_worker_pkg AS
  PROCEDURE main(
    errbuf           OUT VARCHAR2,
    retcode          OUT NUMBER,
    p_interface_code IN  VARCHAR2,
    p_batch_size     IN  NUMBER DEFAULT 500
  ) IS
    CURSOR c_message IS
      SELECT message_id
        FROM xxint_messages
       WHERE interface_code = p_interface_code
         AND status_code IN ('VALIDATED', 'RETRY')
         AND NVL(next_retry_date, SYSDATE) <= SYSDATE
       ORDER BY message_id
       FOR UPDATE SKIP LOCKED;

    l_ebs_transaction_id NUMBER;
    l_request_id         NUMBER;
    l_success_count      PLS_INTEGER := 0;
    l_error_count        PLS_INTEGER := 0;
    l_processed_count    PLS_INTEGER := 0;
  BEGIN
    errbuf := NULL;
    retcode := 0;

    fnd_file.put_line(fnd_file.log,
      'Start interface=' || p_interface_code ||
      ', batch_size=' || p_batch_size);

    FOR r IN c_message LOOP
      EXIT WHEN l_processed_count >= p_batch_size;
      l_processed_count := l_processed_count + 1;
      SAVEPOINT one_message;
      BEGIN
        UPDATE xxint_messages
           SET status_code = 'SUBMITTED',
               last_update_date = SYSDATE,
               last_updated_by = fnd_global.user_id
         WHERE message_id = r.message_id;

        -- Router 内部调用 AP/AR/GL/FA/INV 的标准 API 或 Open Interface。
        xxint_router.process_message(
          p_message_id         => r.message_id,
          x_ebs_transaction_id => l_ebs_transaction_id,
          x_request_id         => l_request_id
        );

        UPDATE xxint_messages
           SET status_code = CASE
                               WHEN l_request_id IS NULL THEN 'SUCCESS'
                               ELSE 'SUBMITTED'
                             END,
               ebs_transaction_id = l_ebs_transaction_id,
               request_id = l_request_id,
               error_code = NULL,
               error_message = NULL,
               last_update_date = SYSDATE,
               last_updated_by = fnd_global.user_id
         WHERE message_id = r.message_id;

        l_success_count := l_success_count + 1;
        COMMIT;
      EXCEPTION
        WHEN OTHERS THEN
          ROLLBACK TO one_message;
          UPDATE xxint_messages
             SET status_code = CASE
                                 WHEN retry_count + 1 >= 5 THEN 'DEAD'
                                 ELSE 'RETRY'
                               END,
                 retry_count = retry_count + 1,
                 next_retry_date = SYSDATE
                   + (POWER(2, LEAST(retry_count + 1, 8)) / 1440),
                 error_code = TO_CHAR(SQLCODE),
                 error_message = SUBSTR(SQLERRM, 1, 2000),
                 last_update_date = SYSDATE,
                 last_updated_by = fnd_global.user_id
           WHERE message_id = r.message_id;
          l_error_count := l_error_count + 1;
          COMMIT;
      END;
    END LOOP;

    IF l_error_count > 0 THEN
      retcode := 1; -- Warning
      errbuf := l_error_count || ' message(s) failed';
    END IF;

    fnd_file.put_line(fnd_file.output,
      'success=' || l_success_count || ', error=' || l_error_count);
  EXCEPTION
    WHEN OTHERS THEN
      ROLLBACK;
      retcode := 2;
      errbuf := SUBSTR(SQLERRM, 1, 240);
      fnd_file.put_line(fnd_file.log,
        'Fatal error: ' || DBMS_UTILITY.format_error_backtrace);
  END main;
END xxint_worker_pkg;
/
```

R12.2 自定义数据库对象应放在自定义 Schema/APPS 同义词策略下，并通过 ADOP/Edition-Based Redefinition 合规发布。Worker 使用 `FOR UPDATE SKIP LOCKED` 支持多并发实例，并在达到批次大小后停止领取。

## 3. 标准 PL/SQL API 调用模板

```sql
DECLARE
  l_return_status VARCHAR2(1);
  l_msg_count     NUMBER;
  l_msg_data      VARCHAR2(2000);
BEGIN
  fnd_global.apps_initialize(:p_user_id, :p_resp_id, :p_resp_appl_id);
  mo_global.init(:p_application_short_name);
  mo_global.set_policy_context('S', :p_org_id);

  fnd_msg_pub.initialize;

  xx_public_api.do_business_action(
    p_api_version      => 1.0,
    p_init_msg_list    => fnd_api.g_true,
    p_commit           => fnd_api.g_false,
    p_business_key     => :p_business_key,
    x_return_status    => l_return_status,
    x_msg_count        => l_msg_count,
    x_msg_data         => l_msg_data
  );

  IF l_return_status <> fnd_api.g_ret_sts_success THEN
    FOR i IN 1 .. l_msg_count LOOP
      dbms_output.put_line(
        fnd_msg_pub.get(p_msg_index => i, p_encoded => 'F'));
    END LOOP;
    ROLLBACK;
    raise_application_error(-20060,
      'API failed: ' || SUBSTR(l_msg_data, 1, 1800));
  END IF;

  COMMIT;
END;
/
```

`XX_PUBLIC_API` 只是调用模板占位符。实际 API 名、版本、Record Type、`G_MISS_*` 默认值、是否自动 Commit，必须从当前实例 Integration Repository/Package Specification 获取。

## 4. ISG REST 服务部署与调用

### 4.1 管理端步骤

1. 在 Integration Repository 按 Internal Name 搜索标准 API/Concurrent/Open Interface。
2. 检查方法签名、方向、生命周期状态和产品补丁说明。
3. 在 REST Web Service 页设置唯一 Service Alias，仅勾选需要的方法/HTTP Verb。
4. 设置 Authentication Type，Deploy 后为专用用户建立 Grant。
5. 从当前实例下载 WADL/XSD，生成客户端契约测试。
6. 使用最小权限职责/MOAC/Data Access Set 验证不同组织的数据隔离。

### 4.2 REST 调用

```bash
curl --fail-with-body \
  --request POST \
  --url 'https://ebs.example.com/webservices/rest/<service-alias>/<operation>/' \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer <access-token>' \
  --header 'X-Correlation-ID: P2P-20260822-000001' \
  --data @request.json
```

REST Endpoint、资源路径、Context Header 和 Payload 必须从已部署服务的 WADL/XSD 获得。Token、密码和 Cookie 不写脚本、Git 或 Concurrent Log；Basic Authentication 只通过 HTTPS 使用。

### 4.3 带退避的客户端示例

```bash
#!/usr/bin/env bash
set -euo pipefail

endpoint="$1"
payload_file="$2"
correlation_id="$3"

for attempt in 1 2 3 4 5; do
  http_code="$(curl --silent --show-error \
    --output response.json \
    --write-out '%{http_code}' \
    --request POST \
    --url "$endpoint" \
    --header 'Content-Type: application/json' \
    --header "Authorization: Bearer ${EBS_ACCESS_TOKEN:?}" \
    --header "X-Correlation-ID: $correlation_id" \
    --data "@$payload_file")"

  case "$http_code" in
    200|201|202) exit 0 ;;
    408|429|500|502|503|504) sleep "$((2 ** attempt))" ;;
    *) exit 1 ;;
  esac
done
exit 2
```

客户端只能重试确认幂等的操作。连接超时不代表 EBS 未处理，应先以业务幂等键/Request ID 查询结果；HTTP 4xx 业务/权限错误不应自动重试。

## 5. Concurrent Program 异步状态 API

```sql
SELECT fcr.request_id,
       fcr.parent_request_id,
       fcr.phase_code,
       fcr.status_code,
       fcr.actual_start_date,
       fcr.actual_completion_date,
       fcr.logfile_name,
       fcr.outfile_name,
       fcr.completion_text
  FROM fnd_concurrent_requests fcr
 WHERE fcr.request_id = :p_request_id;
```

外部 REST 接口推荐返回：

```json
{
  "correlationId": "P2P-20260822-000001",
  "requestId": 12345678,
  "status": "SUBMITTED",
  "statusUrl": "/integrations/P2P-20260822-000001"
}
```

状态查询服务将 EBS `PHASE_CODE/STATUS_CODE` 映射为 Submitted/Running/Success/Warning/Error，不直接把内部单字符状态暴露给消费者。

## 6. EBS 调用外部 REST

Oracle 官方 Service Invocation Framework 使用 Workflow Business Event System；PL/SQL 通过 `WF_EVENT.RAISE` 触发，Java Deferred Agent Listener 调用服务，并可在 Service Invocation Monitor 监控。

```sql
DECLARE
  l_params wf_parameter_list_t := wf_parameter_list_t();
BEGIN
  wf_event.addparametertolist(
    p_name          => 'X-Correlation-ID',
    p_value         => :p_correlation_id,
    p_parameterlist => l_params
  );

  wf_event.raise(
    p_event_name => 'oracle.apps.xxint.rest.invoke',
    p_event_key  => :p_correlation_id,
    p_parameters => l_params,
    p_event_data => :p_json_payload
  );
END;
/
```

需要先定义 Event、REST Invocation Metadata 和带 Java Rule Function 的 Subscription。不要自行在数据库中保存明文密码或使用不受支持的网络 ACL 绕过框架。

## 7. 可观测性和错误分类

```sql
SELECT interface_code,
       status_code,
       COUNT(*) message_count,
       MIN(creation_date) oldest_message,
       MAX(retry_count) max_retry_count
  FROM xxint_messages
 WHERE creation_date >= SYSDATE - 1
 GROUP BY interface_code, status_code
 ORDER BY interface_code, status_code;
```

| 错误类型 | 示例 | 自动重试 |
| --- | --- | --- |
| Payload | JSON/XML 格式、必填字段缺失 | 否 |
| Master Data | Supplier/Customer/Item/CCID 无效 | 否，修数后重提 |
| Business Rule | Period Closed、Hold、Credit Limit | 通常否 |
| Authorization | ISG Grant、MOAC、Data Access Set | 否 |
| Technical | Timeout、429、临时网络/DB 资源 | 有上限地重试 |
| Unknown Result | 请求超时但 EBS 可能已成功 | 先查询，不直接重放 |

日志必须包含 Correlation ID、Interface Code、External Key、Request ID、EBS ID 和阶段，但要脱敏银行账户、税号、身份证、Token、密码和完整 Payload。

## 8. 测试矩阵

- Contract：WADL/WSDL/XSD/JSON Schema 与客户端版本兼容。
- Functional：正常、缺字段、无效主数据、跨 OU、期间关闭、重复消息。
- Transaction：部分成功、API 回滚、Concurrent Warning/Error、超时结果未知。
- Performance：批量吞吐、Commit Size、并发 Worker、热点唯一键、大表查询计划。
- Security：最小 Grant、无权 OU/Ledger、Token 过期、TLS/证书轮换、日志脱敏。
- Recovery：Worker 中断、并发管理器重启、消息重放、Dead Letter 修复、灾备切换。
- Reconciliation：输入/接口/业务/SLA/GL/ACK 的数量和金额闭环。

## 9. 关联文档

- [开放接口、API 与迁移](integration.md)
- [Concurrent Program](concurrent-programs.md)
- [技术常用表](tables.md)
- [端到端接口编排](../08-e2e/interfaces.md)
- [通用接口治理](../01-common/interfaces.md)

## 10. 官方参考

- [ISG Implementation Guide: Deploying REST Services](https://docs.oracle.com/cd/E26401_01/doc.122/e20925/T511175T513044.htm)
- [ISG Developer's Guide: Using PL/SQL APIs as Web Services](https://docs.oracle.com/cd/E26401_01/doc.122/e20927/T511473T522190.htm)
- [ISG Implementation Guide: Service Invocation Framework](https://docs.oracle.com/cd/E26401_01/doc.122/e20925/T511175T513090.htm)
- [Oracle E-Business Suite Concepts: Integration Repository](https://docs.oracle.com/cd/E26401_01/doc.122/e22949/T120505T120507.htm)
