# AutoInvoice、Lockbox 接口与排错

## AutoInvoice

```text
OM/Projects/External System
→ RA_INTERFACE_LINES_ALL (+ DISTRIBUTIONS/SALESCREDITS)
→ AutoInvoice Import
→ RA_CUSTOMER_TRX_ALL / Lines / GL Distributions
```

`INTERFACE_LINE_CONTEXT + INTERFACE_LINE_ATTRIBUTE1..15` 应组成稳定的外部唯一键和 Drilldown 键。Grouping Rule 决定哪些行合并到同一交易；Transaction Source 决定自动编号、字段验证和引用规则。

## Lockbox

```text
Bank File → SQL*Loader/Transmission
→ AR_PAYMENTS_INTERFACE_ALL / Interim Tables
→ Validate Lockbox → Post QuickCash
→ Cash Receipt + Applications
```

## SQL

```sql
SELECT interface_line_id, interface_line_context,
       interface_line_attribute1, batch_source_name,
       line_type, amount, currency_code, trx_date, gl_date,
       orig_system_bill_customer_id,
       orig_system_bill_address_id, org_id, request_id
  FROM ra_interface_lines_all
 WHERE interface_line_context = :p_context
   AND interface_line_attribute1 = :p_external_key;

SELECT ril.interface_line_id, rie.message_text,
       rie.invalid_value
  FROM ra_interface_lines_all ril
  JOIN ra_interface_errors_all rie
    ON rie.interface_line_id = ril.interface_line_id
 WHERE ril.request_id = :p_request_id
 ORDER BY ril.interface_line_id;

SELECT transmission_record_id, transmission_id, record_type,
       status, amount, item_number, customer_number,
       invoice1, org_id
  FROM ar_payments_interface_all
 WHERE transmission_id = :p_transmission_id
 ORDER BY transmission_record_id;
```

## 排查

- AutoInvoice 无数据：检查 Source/Group ID/ORG_ID 参数、`REQUEST_ID`、已处理标志和 MOAC。
- Invalid Bill-to：检查 TCA Account/Site Use、OU、Status、Orig System Reference 和 ID/Reference 使用方式。
- Grouping 异常：输出 Grouping Rule 字段，比较 Currency、Terms、Bill-to、Source、Trx Date、Reference。
- Invalid Tax/Account：分别检查 EBTax 确定因素和 AutoAccounting/SLA，不通过接口硬塞一个无效 CCID。
- Lockbox 控制总数错：对比 Header/Trailer 记录数、金额、小数位、正负号和 Transmission Format。
- Lockbox 无法识别客户/发票：检查 Matching Order、Customer/Invoice/Bank Account 引用和 AutoCash Rules。

## 关联

- [AR 交易](transactions.md)
- [AR 收款](receipts.md)
- [集成设计](../09-technical/integration.md)

## 官方参考

- [Oracle Receivables Implementation Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/f10310/)
