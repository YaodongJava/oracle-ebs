# 财务公共基础接口实现

## 1. 适用场景

本章提供各业务模块共用的接口基础设计：

- EBS 用户/职责/MOAC 上下文初始化。
- 外部系统提交 EBS Concurrent Program。
- 统一 Staging、幂等、状态、错误和对账模型。
- EBS Business Event 发布与下游解耦。
- Integration Repository/Integrated SOA Gateway（ISG）服务暴露。

> **安全边界**：写入类代码仅用于开发/测试环境。生产必须完成代码审查、权限审批、性能测试和回退演练。不使用 APPS 账号作为外部系统直连账号。

## 2. 接口标准分层

```text
Source System
  → API Gateway / SFTP / MQ
  → Landing（原始报文，不可变）
  → XX Staging（标准化、幂等、业务校验）
  → Oracle Public API / Open Interface
  → Standard Import Concurrent Program
  → Base Transaction → SLA → GL
  → Acknowledgement / Reconciliation / Archive
```

### 2.1 统一状态

| 状态 | 中文含义 | 允许的下一步 |
| --- | --- | --- |
| `NEW` | 已接收 | 格式与幂等校验 |
| `VALIDATED` | 已校验 | 写入 Oracle 标准接口/API |
| `SUBMITTED` | 已提交 | 等待 Concurrent Request/API 结果 |
| `SUCCESS` | 已成功 | 对账、ACK、归档 |
| `ERROR` | 业务错误 | 修正后人工/受控重试 |
| `RETRY` | 技术重试 | 按指数退避再执行 |
| `DEAD` | 超过重试上限 | 人工介入，不自动循环 |

## 3. 自定义 Staging 表实现

```sql
-- 仅在自定义 schema 中创建，并通过 R12.2 adop/EBR 标准发布。
CREATE TABLE xxint_messages (
  message_id          NUMBER        NOT NULL,
  interface_code      VARCHAR2(30)  NOT NULL,
  source_system       VARCHAR2(30)  NOT NULL,
  external_key        VARCHAR2(240) NOT NULL,
  payload_hash        VARCHAR2(64)  NOT NULL,
  correlation_id      VARCHAR2(100),
  org_id              NUMBER,
  ledger_id           NUMBER,
  status_code         VARCHAR2(20)  DEFAULT 'NEW' NOT NULL,
  retry_count         NUMBER        DEFAULT 0 NOT NULL,
  next_retry_date     DATE,
  request_id          NUMBER,
  ebs_transaction_id  NUMBER,
  error_code          VARCHAR2(100),
  error_message       VARCHAR2(2000),
  payload_clob        CLOB,
  creation_date       DATE          DEFAULT SYSDATE NOT NULL,
  created_by          NUMBER        NOT NULL,
  last_update_date    DATE          DEFAULT SYSDATE NOT NULL,
  last_updated_by     NUMBER        NOT NULL,
  CONSTRAINT xxint_messages_pk PRIMARY KEY (message_id),
  CONSTRAINT xxint_messages_u1 UNIQUE
    (interface_code, source_system, external_key)
);
```

幂等不只检查 `EXTERNAL_KEY`：如同一键的 `PAYLOAD_HASH` 改变，应进入“源数据冲突”而非静默覆盖。

## 4. EBS/MOAC 上下文代码

```sql
DECLARE
  l_user_id       NUMBER := :p_user_id;
  l_resp_id       NUMBER := :p_resp_id;
  l_resp_appl_id  NUMBER := :p_resp_appl_id;
  l_org_id        NUMBER := :p_org_id;
BEGIN
  fnd_global.apps_initialize(
    user_id      => l_user_id,
    resp_id      => l_resp_id,
    resp_appl_id => l_resp_appl_id
  );

  -- 按实际产品使用应用短名，AP 通常为 SQLAP。
  mo_global.init('SQLAP');
  mo_global.set_policy_context('S', l_org_id);

  IF mo_global.get_current_org_id <> l_org_id THEN
    raise_application_error(-20001, 'MOAC context initialization failed');
  END IF;
END;
/
```

生产实现中，User/Responsibility 应是专用、最小权限、可审计的接口身份；不使用离职员工或 System Administrator 职责。

## 5. 提交 Concurrent Program

```sql
DECLARE
  l_request_id NUMBER;
BEGIN
  -- 必须已执行 FND_GLOBAL.APPS_INITIALIZE。
  l_request_id := fnd_request.submit_request(
    application => :p_application_short_name,
    program     => :p_concurrent_program_short_name,
    description => NULL,
    start_time  => NULL,
    sub_request => FALSE,
    argument1   => :p_argument1,
    argument2   => :p_argument2,
    argument3   => :p_argument3
  );

  IF l_request_id = 0 THEN
    raise_application_error(-20002,
      'Submit request failed: ' || fnd_message.get);
  END IF;

  -- FND_REQUEST 提交需要 commit 后 Concurrent Manager 才能看到。
  COMMIT;
  dbms_output.put_line('REQUEST_ID=' || l_request_id);
END;
/
```

`ARGUMENT1..100` 是位置参数。必须在当前实例的 Concurrent Program Parameters 窗口/数据字典中确认顺序，不从网上复制其他补丁级别的参数位置。

## 6. 等待请求完成

```sql
DECLARE
  l_phase       VARCHAR2(80);
  l_status      VARCHAR2(80);
  l_dev_phase   VARCHAR2(30);
  l_dev_status  VARCHAR2(30);
  l_message     VARCHAR2(240);
  l_done        BOOLEAN;
BEGIN
  l_done := fnd_concurrent.wait_for_request(
    request_id => :p_request_id,
    interval   => 5,
    max_wait   => 600,
    phase      => l_phase,
    status     => l_status,
    dev_phase  => l_dev_phase,
    dev_status => l_dev_status,
    message    => l_message
  );

  dbms_output.put_line(
    l_dev_phase || '/' || l_dev_status || ': ' || l_message);

  IF NOT l_done OR l_dev_phase <> 'COMPLETE'
     OR l_dev_status NOT IN ('NORMAL', 'WARNING') THEN
    raise_application_error(-20003, 'Concurrent request failed or timed out');
  END IF;
END;
/
```

对高并发接口，不应让每个 Web 请求长时间同步等待 Concurrent Program；建议返回 `REQUEST_ID`，由客户端轮询或通过回调/消息获取结果。

## 7. Business Event 发布示例

```sql
DECLARE
  l_parameter_list wf_parameter_list_t := wf_parameter_list_t();
BEGIN
  wf_event.addparametertolist(
    p_name          => 'SOURCE_SYSTEM',
    p_value         => :p_source_system,
    p_parameterlist => l_parameter_list
  );
  wf_event.addparametertolist(
    p_name          => 'TRANSACTION_ID',
    p_value         => TO_CHAR(:p_transaction_id),
    p_parameterlist => l_parameter_list
  );

  wf_event.raise(
    p_event_name => 'oracle.apps.xxint.transaction.completed',
    p_event_key  => :p_interface_code || ':' || :p_external_key,
    p_parameters => l_parameter_list,
    p_event_data => NULL
  );
END;
/
```

Event Name 必须先在 Workflow Administrator 中定义并启用。订阅应幂等，因为 Workflow/Agent 重试可能再次交付同一 Event Key。

## 8. 业界常用案例

| 案例 | 推荐方式 | 不推荐 |
| --- | --- | --- |
| 主数据批量导入 | 标准 Open Interface/API + Concurrent Program | 直接 DML TCA/FND/HR 基表 |
| 外部系统发起批处理 | ISG Concurrent Program REST 或中间件提交 | 长时间 HTTP 同步锁住 |
| EBS 业务完成通知 | Business Event + Queue/Subscriber | 在业务交易中同步调外部 HTTP |
| 参数/组织映射 | 客户自定义配置表 + 有效期 | 在代码中硬编码 OU/Ledger/CCID |

## 9. 官方参考

- [Oracle E-Business Suite Integrated SOA Gateway Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e20925/)
- [Oracle E-Business Suite Integrated SOA Gateway Developer's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e20927/)
- [Oracle E-Business Suite Concepts: Integration Repository](https://docs.oracle.com/cd/E26401_01/doc.122/e22949/T120505T120507.htm)

