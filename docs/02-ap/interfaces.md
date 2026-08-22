# Oracle Payables 接口实现案例

## 1. 业界常用场景

| 场景 | 推荐接口 | 后续处理 |
| --- | --- | --- |
| OCR/电子发票平台导入 AP | `AP_INVOICES_INTERFACE` + `AP_INVOICE_LINES_INTERFACE` | Payables Open Interface Import → Validation/Approval |
| 采购系统传 PO Match Invoice | AP Invoice Interface，行上传 PO/Receipt 关键 ID | Import → Invoice Validation |
| 员工费用/差旅平台 | Internet Expenses/标准费用报销接口 | Expense Report Import → AP Invoice |
| 供应商门户主数据 | Supplier Open Interface/当前实例公开 Supplier API | Supplier/Site Import + 数据治理 |
| 付款状态回传 | AP/IBY 只读 Outbound View/Event | 银行回执、清算和对账 |

> 示例使用非 PO 标准发票，为了展示核心代码故省略 EBTax、DFF、外币和审批扩展字段。上线前必须以当前实例 eTRM 和 Payables Interface 字段校验。

## 2. AP 发票导入完整骨架

### 2.1 输入参数

```text
External invoice key, invoice number/date, supplier/site,
OU, currency, amount, GL date, expense CCID,
description, tax classification, attachment reference
```

### 2.2 写入标准接口表

```sql
DECLARE
  l_invoice_id       NUMBER := ap_invoices_interface_s.NEXTVAL;
  l_invoice_line_id  NUMBER := ap_invoice_lines_interface_s.NEXTVAL;
  l_source           VARCHAR2(80) := 'XX_OCR_INVOICE';
  l_group_id         NUMBER := :p_group_id;
BEGIN
  -- 导入前在 XX Staging 层检查幂等键，不要只查接口表。
  INSERT INTO ap_invoices_interface (
    invoice_id,
    invoice_num,
    invoice_type_lookup_code,
    invoice_date,
    vendor_id,
    vendor_site_id,
    invoice_amount,
    invoice_currency_code,
    description,
    source,
    group_id,
    org_id,
    gl_date,
    creation_date,
    created_by,
    last_update_date,
    last_updated_by,
    last_update_login
  ) VALUES (
    l_invoice_id,
    :p_invoice_num,
    'STANDARD',
    :p_invoice_date,
    :p_vendor_id,
    :p_vendor_site_id,
    :p_invoice_amount,
    :p_currency_code,
    :p_description,
    l_source,
    l_group_id,
    :p_org_id,
    :p_gl_date,
    SYSDATE,
    fnd_global.user_id,
    SYSDATE,
    fnd_global.user_id,
    fnd_global.login_id
  );

  INSERT INTO ap_invoice_lines_interface (
    invoice_id,
    invoice_line_id,
    line_number,
    line_type_lookup_code,
    amount,
    accounting_date,
    description,
    dist_code_combination_id,
    org_id,
    creation_date,
    created_by,
    last_update_date,
    last_updated_by,
    last_update_login
  ) VALUES (
    l_invoice_id,
    l_invoice_line_id,
    1,
    'ITEM',
    :p_invoice_amount,
    :p_gl_date,
    :p_line_description,
    :p_expense_ccid,
    :p_org_id,
    SYSDATE,
    fnd_global.user_id,
    SYSDATE,
    fnd_global.user_id,
    fnd_global.login_id
  );

  COMMIT;
  dbms_output.put_line('INTERFACE_INVOICE_ID=' || l_invoice_id);
END;
/
```

### 2.3 PO/Receipt Match 行扩展

对 PO 匹配发票，优先传稳定内部 ID，并在 Staging 校验它们属于同一 OU/Supplier：

```sql
INSERT INTO ap_invoice_lines_interface (
  invoice_id,
  invoice_line_id,
  line_number,
  line_type_lookup_code,
  amount,
  po_header_id,
  po_line_id,
  po_line_location_id,
  po_distribution_id,
  rcv_transaction_id,
  quantity_invoiced,
  unit_price,
  org_id,
  accounting_date,
  creation_date,
  created_by,
  last_update_date,
  last_updated_by
) VALUES (
  :p_interface_invoice_id,
  ap_invoice_lines_interface_s.NEXTVAL,
  :p_line_number,
  'ITEM',
  :p_line_amount,
  :p_po_header_id,
  :p_po_line_id,
  :p_line_location_id,
  :p_po_distribution_id,
  :p_rcv_transaction_id,
  :p_quantity_invoiced,
  :p_unit_price,
  :p_org_id,
  :p_gl_date,
  SYSDATE,
  fnd_global.user_id,
  SYSDATE,
  fnd_global.user_id
);
```

PO/Receipt 列的必填组合受 Match Option（2-way/3-way/4-way）和当前补丁级别影响，不要同时传入互相矛盾的编码与 ID。

## 3. 提交 Payables Open Interface Import

```sql
DECLARE
  l_request_id NUMBER;
BEGIN
  fnd_global.apps_initialize(
    user_id      => :p_user_id,
    resp_id      => :p_resp_id,
    resp_appl_id => :p_resp_appl_id
  );
  mo_global.init('SQLAP');
  mo_global.set_policy_context('S', :p_org_id);

  l_request_id := fnd_request.submit_request(
    application => 'SQLAP',
    program     => 'APXIIMPT',
    description => NULL,
    start_time  => NULL,
    sub_request => FALSE,
    argument1   => TO_CHAR(:p_org_id),
    argument2   => 'XX_OCR_INVOICE',
    argument3   => TO_CHAR(:p_group_id)
  );

  IF l_request_id = 0 THEN
    raise_application_error(-20010, fnd_message.get);
  END IF;
  COMMIT;
  dbms_output.put_line('REQUEST_ID=' || l_request_id);
END;
/
```

> `APXIIMPT` 的参数数量/顺序必须在目标实例的“Payables Open Interface Import”程序定义中核对。上例只展示 OU/Source/Group ID 核心位置的常见骨架，不可未核对即用于生产。

## 4. 拒绝错误回写

```sql
SELECT aii.invoice_id,
       aii.invoice_num,
       air.parent_table,
       air.parent_id,
       air.reject_lookup_code,
       lv.meaning reject_meaning
  FROM ap_invoices_interface aii
  JOIN ap_interface_rejections air
    ON (air.parent_table = 'AP_INVOICES_INTERFACE'
        AND air.parent_id = aii.invoice_id)
    OR (air.parent_table = 'AP_INVOICE_LINES_INTERFACE'
        AND air.parent_id IN
            (SELECT aili.invoice_line_id
               FROM ap_invoice_lines_interface aili
              WHERE aili.invoice_id = aii.invoice_id))
  LEFT JOIN fnd_lookup_values_vl lv
    ON lv.lookup_code = air.reject_lookup_code
   AND lv.language = USERENV('LANG')
 WHERE aii.source = 'XX_OCR_INVOICE'
   AND aii.group_id = :p_group_id;
```

不要把 `REJECT_LOOKUP_CODE` 直接展示给业务用户。在自定义错误字典中增加中文说明、责任方（源系统/主数据/AP 会计）和可重试标志。

## 5. 成功幂等和对账

```sql
SELECT aia.invoice_id,
       aia.invoice_num,
       aia.org_id,
       aia.source,
       aia.vendor_id,
       aia.invoice_amount,
       aia.invoice_currency_code,
       aia.creation_date
  FROM ap_invoices_all aia
 WHERE aia.source = 'XX_OCR_INVOICE'
   AND aia.org_id = :p_org_id
   AND aia.invoice_num = :p_invoice_num
   AND aia.vendor_id = :p_vendor_id;
```

推荐将接口 `INVOICE_ID` 写入自定义 Staging 的 `EBS_TRANSACTION_ID`，并对比源系统批次数、金额、币种、成功/拒绝数。

## 6. 实施方法

1. 先定义 Supplier/Site/OU/Tax/PO 映射和重复规则。
2. 将 OCR 识别值与人工确认值分开保存，保留原始图像参考。
3. 接口批次使用唯一 Source+Group ID，拒绝项修正后只重试失败行。
4. 导入成功不等于验证/审批/会计成功，状态回传需覆盖后续阶段。
5. 付款和银行账号变更使用 IBY/供应商标准流程与双人复核，不通过发票接口搭车修改。

## 7. 关联文档

- [AP Open Interface 排错](interfaces-troubleshooting.md)
- [AP 常用表](tables.md)
- [通用接口框架](../01-common/interfaces.md)

## 8. 官方参考

- [Oracle Payables Implementation Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48761/)
- [Oracle E-Business Suite Integration Repository](https://docs.oracle.com/cd/E26401_01/doc.122/e22949/T120505T120507.htm)

