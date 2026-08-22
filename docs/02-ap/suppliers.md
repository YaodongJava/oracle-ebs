# 供应商与供应商地点

## 数据模型

R12 供应商基于 TCA：`AP_SUPPLIERS` 关联 `HZ_PARTIES`，供应商地点在 `AP_SUPPLIER_SITES_ALL`，地址与 Party Site 在 `HZ_LOCATIONS/HZ_PARTY_SITES`，联系人在 TCA Relationship/Contact 模型，银行信息由 CE/IBY 管理。Supplier 为全局层，Site 通常按 OU（`ORG_ID`）设置采购/付款/报价用途。

## 配置和治理

- 定义统一的供应商命名、税号、重复检查和黑名单规则。
- 区分 Supplier、Address、Site、Contact、Bank Account 的层级，不为每个 OU 重复建 Supplier。
- Site 上检查 Pay/Purchasing/RFQ Flag、Payment Terms、Currency、Liability/Prepayment Account、Tax、Withholding、Payment Method。
- 银行账户变更应实施双人复核、独立回拨验证和审计，防止支付欺诈。

## SQL

```sql
SELECT aps.vendor_id, aps.segment1 supplier_number,
       aps.vendor_name, aps.vendor_type_lookup_code,
       aps.enabled_flag, aps.start_date_active, aps.end_date_active,
       aps.party_id, hp.party_number
  FROM ap_suppliers aps
  LEFT JOIN hz_parties hp ON hp.party_id = aps.party_id
 WHERE aps.vendor_id = :p_vendor_id;

SELECT assa.vendor_site_id, assa.vendor_id, assa.org_id,
       assa.vendor_site_code, assa.pay_site_flag,
       assa.purchasing_site_flag, assa.rfq_only_site_flag,
       assa.inactive_date, assa.payment_currency_code,
       assa.terms_id, assa.accts_pay_code_combination_id,
       assa.prepay_code_combination_id
  FROM ap_supplier_sites_all assa
 WHERE assa.vendor_id = :p_vendor_id
 ORDER BY assa.org_id, assa.vendor_site_code;

-- 可疑重复，最终须结合税号/地址/银行人工判断
SELECT UPPER(TRIM(vendor_name)) normalized_name, COUNT(*) cnt
  FROM ap_suppliers
 GROUP BY UPPER(TRIM(vendor_name))
HAVING COUNT(*) > 1;
```

## 排查

- LOV 找不到：查 Supplier/Site 有效期、OU、Site Use Flag、Hold、业务单据日期。
- 默认账户不对：按 Supplier Site → Financial Options 的默认链路查，再查 SLA 是否覆盖。
- 重复供应商：先冻结新交易，评估 TCA Merge/Supplier Merge 标准功能，不直接合并表数据。
- 银行账户不可选：查 IBY 所有者/用途、有效期、币种、国家和付款方法要求。

## 关联

- [AP 发票](invoices.md)
- [AP 付款](payments.md)
- [EBTax](../07-ce-tax/ebtax.md)
