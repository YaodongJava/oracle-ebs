# AP Open Interface、发票导入与排错

## 架构

```text
Source System/Staging
 → AP_INVOICES_INTERFACE / AP_INVOICE_LINES_INTERFACE
 → Payables Open Interface Import
 → AP_INVOICES_ALL / AP_INVOICE_LINES_ALL / Distributions
 → Validation / Approval / Accounting
```

接口必须保存 Source、外部唯一键、OU、Supplier/Site、Currency/Rate、Invoice Date/GL Date、Line Type/Amount、PO/Receipt 匹配键和 DFF。应使用 `GROUP_ID` 隔离批次，使用幂等键防止重复。

## SQL

```sql
SELECT invoice_id, invoice_num, source, group_id, org_id,
       vendor_id, vendor_site_id, invoice_amount,
       invoice_currency_code, invoice_date, gl_date,
       status, request_id
  FROM ap_invoices_interface
 WHERE source = :p_source
   AND group_id = :p_group_id
 ORDER BY invoice_id;

SELECT parent_id, reject_lookup_code, last_update_date
  FROM ap_interface_rejections
 WHERE parent_table IN ('AP_INVOICES_INTERFACE',
                        'AP_INVOICE_LINES_INTERFACE')
   AND parent_id IN
       (SELECT invoice_id FROM ap_invoices_interface
         WHERE group_id = :p_group_id)
 ORDER BY parent_id, last_update_date;

-- 重复外部单号风险
SELECT org_id, vendor_id, invoice_num, COUNT(*) cnt
  FROM ap_invoices_all
 WHERE source = :p_source
 GROUP BY org_id, vendor_id, invoice_num
HAVING COUNT(*) > 1;
```

## 导入检查清单

1. 确认 Import 参数 Source/Group/Batch Name/GL Date 与接口数据一致。
2. 查 `AP_INTERFACE_REJECTIONS` 的 Header/Line 拒绝原因，再查并发日志。
3. 检查 Supplier/Site 有效且属于 OU，Terms、Currency、Account、Tax 和 PO/Receipt 键有效。
4. 对比 Header Amount 与 Lines/Tax；负数、Credit Memo、预付类型须符合业务规则。
5. 拒绝数据修正后重跑；已成功导入的单据不应通过重新插入接口来“更新”。

## 常见错误

- Invalid Supplier/Site：查 ID 与编码是否混用、Site `ORG_ID`、有效期和 Pay Site Flag。
- Duplicate Invoice：查 Supplier + Invoice Number + OU 的重复规则、大小写/空格标准化和接口重试机制。
- Invalid PO/Receipt：查 PO Distribution、Shipment、Receipt Transaction、OU、开放数量和 Match Option。
- Invalid Account：查 CCID/Segments、COA、有效期、Cross Validation 和 Dynamic Insertion。
- 请求完成但无数据：检查 Source/Group 参数、MOAC OU、接口 `STATUS/REQUEST_ID` 和日志中选择行数。

## 关联

- [AP 发票](invoices.md)
- [集成设计](../09-technical/integration.md)

## 官方参考

- [Oracle Payables Implementation Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48761/toc.htm)
