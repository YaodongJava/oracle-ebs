# Oracle Receivables 接口实现案例

## 1. 业界常用场景

| 场景 | 推荐接口 | 说明 |
| --- | --- | --- |
| 电商/计费系统导入销售发票 | AutoInvoice Interface | 支持 Invoice/Debit Memo/Credit Memo/On-account Credit |
| OM/Shipping 开票 | Invoice Interface Workflow + AutoInvoice | 保留 `ORDER ENTRY` Transaction Flexfield |
| 银行收款文件 | AutoLockbox | Import → Validate → QuickCash/Post |
| POS/支付平台实时收款 | 公开 Receipt API 或 IBY Funds Capture | 以当前 Integration Repository 签名为准 |
| 客户主数据同步 | TCA Customer Interface/公开 TCA API | 避免直接 DML `HZ_*` |

## 2. AutoInvoice 实现

### 2.1 最小非 PO 销售发票行

```sql
DECLARE
  l_interface_line_id NUMBER := ra_customer_trx_lines_s.NEXTVAL;
BEGIN
  INSERT INTO ra_interface_lines_all (
    interface_line_id,
    interface_line_context,
    interface_line_attribute1,
    interface_line_attribute2,
    batch_source_name,
    line_type,
    description,
    currency_code,
    amount,
    cust_trx_type_name,
    term_name,
    trx_date,
    gl_date,
    quantity,
    unit_selling_price,
    orig_system_bill_customer_id,
    orig_system_bill_address_id,
    org_id,
    created_by,
    creation_date,
    last_updated_by,
    last_update_date
  ) VALUES (
    l_interface_line_id,
    'XX_BILLING',
    :p_external_invoice_id,
    :p_external_line_id,
    'XX BILLING SOURCE',
    'LINE',
    :p_description,
    :p_currency_code,
    :p_line_amount,
    'Invoice',
    :p_payment_term_name,
    :p_trx_date,
    :p_gl_date,
    :p_quantity,
    :p_unit_selling_price,
    :p_cust_account_id,
    :p_bill_to_address_id,
    :p_org_id,
    fnd_global.user_id,
    SYSDATE,
    fnd_global.user_id,
    SYSDATE
  );

  COMMIT;
  dbms_output.put_line('INTERFACE_LINE_ID=' || l_interface_line_id);
END;
/
```

`ORIG_SYSTEM_BILL_ADDRESS_ID` 的目标对象与 Batch Source 的 ID/Reference 验证方式必须在目标实例中确认。如使用 `*_REF` 字段，不应同时传入互相矛盾的 ID。

### 2.2 指定收入账户（AutoAccounting 可派生时不必传）

```sql
INSERT INTO ra_interface_distributions_all (
  interface_line_id,
  interface_line_context,
  interface_line_attribute1,
  interface_line_attribute2,
  account_class,
  code_combination_id,
  percent,
  org_id,
  created_by,
  creation_date,
  last_updated_by,
  last_update_date
) VALUES (
  :p_interface_line_id,
  'XX_BILLING',
  :p_external_invoice_id,
  :p_external_line_id,
  'REV',
  :p_revenue_ccid,
  100,
  :p_org_id,
  fnd_global.user_id,
  SYSDATE,
  fnd_global.user_id,
  SYSDATE
);
```

在多数实施中，建议由 AutoAccounting/SLA 统一派生账户；仅在有明确业务要求、Batch Source 允许且 CCID 经验证时由源系统传账户。

### 2.3 税务实现

推荐传送 Tax Determining Factors（如 Product Fiscal Classification/Tax Classification、Ship-to/Bill-to），由 EBTax 计算；不推荐源系统直接传 Tax Amount 覆盖 EBTax。必须传税行时，`LINE_TYPE='TAX'` 的必填/禁填字段以 Receivables Reference Guide 为准。

## 3. 提交 AutoInvoice

```sql
DECLARE
  l_request_id NUMBER;
BEGIN
  fnd_global.apps_initialize(:p_user_id, :p_resp_id, :p_resp_appl_id);
  mo_global.init('AR');
  mo_global.set_policy_context('S', :p_org_id);

  l_request_id := fnd_request.submit_request(
    application => 'AR',
    program     => 'RAXTRX',
    description => NULL,
    start_time  => NULL,
    sub_request => FALSE,
    argument1   => TO_CHAR(:p_org_id),
    argument2   => 'XX BILLING SOURCE',
    argument3   => :p_default_date
  );

  IF l_request_id = 0 THEN
    raise_application_error(-20020, fnd_message.get);
  END IF;
  COMMIT;
  dbms_output.put_line('REQUEST_ID=' || l_request_id);
END;
/
```

> `RAXTRX` 参数位置随程序定义和补丁级别可变。在目标实例核对 Program Parameters 后再封装；也可将 AutoInvoice Concurrent Program 作为 ISG REST 服务部署。

## 4. AutoInvoice 错误与成功对账

```sql
-- 错误
SELECT ril.interface_line_id,
       ril.interface_line_context,
       ril.interface_line_attribute1,
       ril.interface_line_attribute2,
       rie.message_text,
       rie.invalid_value
  FROM ra_interface_lines_all ril
  JOIN ra_interface_errors_all rie
    ON rie.interface_line_id = ril.interface_line_id
 WHERE ril.interface_line_context = 'XX_BILLING'
   AND ril.interface_line_attribute1 = :p_external_invoice_id;

-- 成功生成的 AR 行
SELECT rctl.customer_trx_id,
       rcta.trx_number,
       rctl.customer_trx_line_id,
       rctl.line_number,
       rctl.extended_amount
  FROM ra_customer_trx_lines_all rctl
  JOIN ra_customer_trx_all rcta
    ON rcta.customer_trx_id = rctl.customer_trx_id
 WHERE rctl.interface_line_context = 'XX_BILLING'
   AND rctl.interface_line_attribute1 = :p_external_invoice_id
 ORDER BY rctl.line_number;
```

Transaction Flexfield 必须在 AR 中定义并设置唯一性，才能成为可靠的幂等/Drilldown 键。

## 5. AutoLockbox 银行收款案例

```text
Bank statement/lockbox file
→ SFTP landing + checksum/archive
→ SQL*Loader control file
→ AR_PAYMENTS_INTERFACE_ALL
→ AutoLockbox Import
→ Validate
→ QuickCash/Post
→ AR_CASH_RECEIPTS_ALL + AR_RECEIVABLE_APPLICATIONS_ALL
```

业界常用于银行代收、零售门店汇总收款、B2B 虚拟账号收款。文件必须保存 Bank Account + File Sequence + File Hash，防止同一文件被重复 Import/Apply。Oracle 官方也提醒 AutoLockbox 需建立操作控制避免重复处理银行文件。

## 6. ISG AutoInvoice REST 调用方式

Oracle 官方以 Open Interface `RAXMTR` 作为 REST 示例。管理员在 Integration Repository 部署后，必须从当前实例 WADL 取得真实 endpoint/operation/payload schema。

```bash
curl --fail-with-body \
  --request POST \
  --url 'https://ebs.example.com/webservices/rest/<alias>/<operation>/' \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer <token>' \
  --header 'X-Correlation-ID: BILLING-20260822-0001' \
  --data @autoinvoice-line.json
```

不在命令行历史、源码或日志中保存密码/Token。Open Interface REST 只负责写入接口数据，仍需调用 AutoInvoice Concurrent Program 完成业务导入。

## 7. 关联文档

- [AutoInvoice/Lockbox 排错](interfaces-troubleshooting.md)
- [AR 常用表](tables.md)
- [EBTax](../07-ce-tax/ebtax.md)

## 8. 官方参考

- [Oracle Receivables Reference Guide: Interface Tables](https://docs.oracle.com/cd/E26401_01/doc.122/f10312/T447348T383863.htm)
- [Oracle Receivables User Guide: AutoInvoice](https://docs.oracle.com/cd/E26401_01/doc.122/f10570/T355475T382065.htm)
- [Oracle Receivables User Guide: AutoLockbox](https://docs.oracle.com/cd/E26401_01/doc.122/f10570/T355475T382159.htm)
- [ISG Developer's Guide: Open Interface REST Services](https://docs.oracle.com/cd/E26401_01/doc.122/e20927/T511473T669558.htm)
