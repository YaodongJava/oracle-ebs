# Oracle EBS 端到端接口编排案例

## 1. 业界常用集成蓝图

| 业务链路 | 上游/下游系统 | 推荐实现 |
| --- | --- | --- |
| P2P | 采购平台、供应商门户、OCR、银行 | Supplier/PO 标准接口 → Receiving → AP Invoice Interface → IBY |
| O2C | 电商、CRM、OMS、3PL、计费平台 | OM API/Order Import → Shipping → AutoInvoice → Receipt/Lockbox |
| 库存制造 | WMS、3PL、MES、PLM | Item/On-hand 同步 + Inventory/WIP Interface + 事件回传 |
| 项目资产化 | PPM、工时费用、工程系统 | Projects Transaction Import → Capitalization → FA Mass Additions |
| Record-to-Report | 薪资、资金、海外 ERP | GL Interface → Journal Import → Approval/Post → Balance 回传 |

端到端集成不是把多个接口顺序调用完即可；必须统一业务相关号、状态语义、金额/数量控制、回调和补偿策略。

## 2. 统一业务相关号映射表

```sql
CREATE TABLE xxint_business_links (
  link_id              NUMBER        NOT NULL,
  correlation_id       VARCHAR2(100) NOT NULL,
  business_flow        VARCHAR2(30)  NOT NULL,
  source_system        VARCHAR2(30)  NOT NULL,
  source_object_type   VARCHAR2(30)  NOT NULL,
  source_object_key    VARCHAR2(240) NOT NULL,
  ebs_object_type      VARCHAR2(30)  NOT NULL,
  ebs_object_id        NUMBER,
  ebs_object_number    VARCHAR2(240),
  org_id               NUMBER,
  status_code          VARCHAR2(30)  NOT NULL,
  request_id           NUMBER,
  creation_date        DATE          DEFAULT SYSDATE NOT NULL,
  last_update_date     DATE          DEFAULT SYSDATE NOT NULL,
  CONSTRAINT xxint_business_links_pk PRIMARY KEY (link_id),
  CONSTRAINT xxint_business_links_u1 UNIQUE
    (source_system, source_object_type, source_object_key,
     ebs_object_type)
);

CREATE INDEX xxint_business_links_n1
  ON xxint_business_links (correlation_id, business_flow);
```

例：同一 O2C `CORRELATION_ID` 下可以分别保存 External Order、EBS Order Header、Delivery、AR Customer Trx 和 Receipt ID。业务编号只用于展示，稳定关联使用 ID。

## 3. Transactional Outbox 实现

当 EBS 业务完成后需要可靠通知 CRM/WMS/Data Lake，推荐先在同一数据库事务写 Outbox，再由异步 Worker/MQ 发布，避免业务交易等待外部 HTTP。

```sql
CREATE TABLE xxint_outbox (
  event_id          NUMBER        NOT NULL,
  event_name        VARCHAR2(100) NOT NULL,
  event_key         VARCHAR2(240) NOT NULL,
  correlation_id    VARCHAR2(100),
  aggregate_type    VARCHAR2(30)  NOT NULL,
  aggregate_id      VARCHAR2(240) NOT NULL,
  payload_clob      CLOB,
  status_code       VARCHAR2(20)  DEFAULT 'NEW' NOT NULL,
  attempt_count     NUMBER        DEFAULT 0 NOT NULL,
  next_attempt_date DATE,
  published_date    DATE,
  error_message     VARCHAR2(2000),
  creation_date     DATE          DEFAULT SYSDATE NOT NULL,
  CONSTRAINT xxint_outbox_pk PRIMARY KEY (event_id),
  CONSTRAINT xxint_outbox_u1 UNIQUE (event_name, event_key)
);
```

写入示例：

```sql
INSERT INTO xxint_outbox (
  event_id,
  event_name,
  event_key,
  correlation_id,
  aggregate_type,
  aggregate_id,
  payload_clob
) VALUES (
  xxint_outbox_s.NEXTVAL,
  'oracle.apps.xxint.ar.invoice.completed',
  'AR_INVOICE:' || :p_customer_trx_id,
  :p_correlation_id,
  'CUSTOMER_TRX',
  TO_CHAR(:p_customer_trx_id),
  :p_json_payload
);
```

只有在业务 API/Open Interface Import 确认成功、且业务 ID 已取得后才生成“completed”事件。事务回滚时 Outbox 必须同时回滚。

## 4. Outbox Worker 与安全重试

```sql
DECLARE
  CURSOR c_event IS
    SELECT event_id, event_name, event_key, payload_clob
      FROM xxint_outbox
     WHERE status_code IN ('NEW', 'RETRY')
       AND NVL(next_attempt_date, SYSDATE) <= SYSDATE
     ORDER BY event_id
     FOR UPDATE SKIP LOCKED;
BEGIN
  FOR r IN c_event LOOP
    SAVEPOINT one_event;
    BEGIN
      -- Adapter 将消息写入企业 MQ/Kafka/API Gateway；不要在业务触发器中直连外部 HTTP。
      xxint_event_adapter.publish(
        p_event_name => r.event_name,
        p_event_key  => r.event_key,
        p_payload    => r.payload_clob
      );

      UPDATE xxint_outbox
         SET status_code = 'SUCCESS',
             published_date = SYSDATE,
             error_message = NULL
       WHERE event_id = r.event_id;
      COMMIT;
    EXCEPTION
      WHEN OTHERS THEN
        ROLLBACK TO one_event;
        UPDATE xxint_outbox
           SET status_code = CASE
                               WHEN attempt_count + 1 >= 8 THEN 'DEAD'
                               ELSE 'RETRY'
                             END,
               attempt_count = attempt_count + 1,
               next_attempt_date = SYSDATE
                 + (POWER(2, LEAST(attempt_count + 1, 8)) / 1440),
               error_message = SUBSTR(SQLERRM, 1, 2000)
         WHERE event_id = r.event_id;
        COMMIT;
    END;
  END LOOP;
END;
/
```

`XXINT_EVENT_ADAPTER` 是企业适配层扩展点，可实现为 AQ、Workflow Business Event、ISG Service Invocation Framework 或中间件代理。消费者必须以 `EVENT_NAME + EVENT_KEY` 去重，因为“至少一次”投递可能重复。

## 5. O2C：电商订单到发票回传

```text
E-commerce Order
→ Order Import / OM Public API
→ Booking
→ Pick Release / Ship Confirm
→ Workflow Interface Trip Stop
→ AutoInvoice
→ AR Invoice Event
→ CRM/E-commerce Callback
```

### 5.1 跨模块追踪 SQL

```sql
SELECT ooha.header_id,
       ooha.order_number,
       oola.line_id,
       oola.line_number,
       wnd.delivery_id,
       rcta.customer_trx_id,
       rcta.trx_number,
       rctl.customer_trx_line_id
  FROM oe_order_headers_all ooha
  JOIN oe_order_lines_all oola
    ON oola.header_id = ooha.header_id
  LEFT JOIN wsh_delivery_details wdd
    ON wdd.source_code = 'OE'
   AND wdd.source_line_id = oola.line_id
  LEFT JOIN wsh_delivery_assignments wda
    ON wda.delivery_detail_id = wdd.delivery_detail_id
  LEFT JOIN wsh_new_deliveries wnd
    ON wnd.delivery_id = wda.delivery_id
  LEFT JOIN ra_customer_trx_lines_all rctl
    ON rctl.interface_line_context = 'ORDER ENTRY'
   AND rctl.interface_line_attribute6 = TO_CHAR(oola.line_id)
  LEFT JOIN ra_customer_trx_all rcta
    ON rcta.customer_trx_id = rctl.customer_trx_id
 WHERE ooha.header_id = :p_order_header_id
 ORDER BY oola.line_number;
```

Transaction Flexfield 属性位置由 OM/AR 标准配置决定，必须在目标实例核对。订单“已发运”但未开票时，应定位 Workflow、Interface Trip Stop、AutoInvoice 三个断点，不能立即重建订单或发票。

## 6. P2P：OCR 发票到付款

```text
OCR/Portal Invoice
→ XX Staging + duplicate check
→ AP Invoice Open Interface
→ Validation / Matching / Holds
→ Create Accounting
→ Payment Process Request
→ Bank File/ACK
→ CE Reconciliation
```

### 6.1 端到端状态查询

```sql
SELECT aia.invoice_id,
       aia.invoice_num,
       aia.invoice_amount,
       aia.payment_status_flag,
       aia.approval_status,
       apsa.due_date,
       apsa.amount_remaining,
       aca.check_id,
       aca.check_number,
       aca.status_lookup_code payment_status
  FROM ap_invoices_all aia
  JOIN ap_payment_schedules_all apsa
    ON apsa.invoice_id = aia.invoice_id
  LEFT JOIN ap_invoice_payments_all aipa
    ON aipa.invoice_id = aia.invoice_id
  LEFT JOIN ap_checks_all aca
    ON aca.check_id = aipa.check_id
 WHERE aia.invoice_id = :p_invoice_id
 ORDER BY apsa.payment_num, aca.check_id;
```

接口 ACK 应分别报告 Imported、Validated、Accounted、Approved、Paid、Cleared，而不是用一个“SUCCESS”掩盖后续业务状态。Invoice 已导入但被 Hold 属于业务待办，不是技术重试。

## 7. Projects 到 Assets 资本化

关键追溯链：Project/Task/Expenditure Item → Cost Distribution → Asset Line → Mass Addition → Asset ID → FA Accounting。

```sql
SELECT ppa.segment1 project_number,
       ppa.project_id,
       pt.task_id,
       pt.task_number,
       pal.project_asset_line_id,
       pal.project_asset_id,
       fma.mass_addition_id,
       fma.posting_status,
       fma.asset_number
  FROM pa_projects_all ppa
  JOIN pa_tasks pt
    ON pt.project_id = ppa.project_id
  LEFT JOIN pa_project_asset_lines_all pal
    ON pal.project_id = ppa.project_id
   AND pal.task_id = pt.task_id
  LEFT JOIN fa_mass_additions fma
    ON fma.project_asset_line_id = pal.project_asset_line_id
 WHERE ppa.project_id = :p_project_id
 ORDER BY pt.task_number, pal.project_asset_line_id;
```

列名/关联键会受 Projects/Assets 补丁和功能影响，目标实例需用 eTRM 校准。资本化失败时分别检查可资本化成本、Asset Line、Interface Assets、Mass Additions 和 Post Mass Additions。

## 8. 批次控制与对账签字

每个批次至少保存以下控制数据：

| 控制维度 | 示例 |
| --- | --- |
| 输入 | 文件数、消息数、Header/Line 数、金额、数量、币种 |
| 接口 | Validated/Rejected/Submitted 数，Request ID |
| 业务 | EBS 创建、Hold、取消、部分成功数 |
| 会计 | Accounted、Transferred、Imported、Posted 金额 |
| 输出 | ACK/Callback 成功、重试、Dead Letter 数 |

```sql
SELECT business_flow,
       status_code,
       COUNT(*) object_count,
       COUNT(DISTINCT correlation_id) flow_count
  FROM xxint_business_links
 WHERE creation_date >= :p_start_date
   AND creation_date < :p_end_date + 1
 GROUP BY business_flow, status_code
 ORDER BY business_flow, status_code;
```

## 9. 补偿而非回滚所有系统

- AP 发票已验证：使用标准 Cancel/Debit Memo 流程，不删除发票。
- AR 发票已完成：使用 Credit Memo/Cancel（按 Transaction Type 规则），不删 AR 基表。
- 库存已交易：用相反方向的标准交易，不删除 MMT。
- Journal 已过账：创建 Reversal Journal，不改 GL Balance。
- 付款已发送银行：先查询银行状态；必要时按银行和 IBY 标准 Stop/Void 流程。

## 10. 关联文档

- [P2P](procure-to-pay.md)
- [O2C](order-to-cash.md)
- [库存、WIP、成本到 GL](inventory-wip-cost-gl.md)
- [项目到资产](projects-assets.md)
- [技术接口实现](../09-technical/interfaces.md)

## 11. 官方参考

- [Oracle E-Business Suite Integrated SOA Gateway Developer's Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e20927/)
- [Oracle E-Business Suite Concepts: Integration Repository](https://docs.oracle.com/cd/E26401_01/doc.122/e22949/T120505T120507.htm)
- [Oracle Workflow Developer's Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e22008/)
