# 客户、客户地点、付款条件与信用管理

## TCA 模型

```text
HZ_PARTIES（主体）
 → HZ_CUST_ACCOUNTS（客户账户）
   → HZ_CUST_ACCT_SITES_ALL（账户地点）
     → HZ_CUST_SITE_USES_ALL（Bill-To / Ship-To / Dunning）
HZ_PARTY_SITES → HZ_LOCATIONS（地址）
```

Party 表示真实世界主体，Customer Account 表示交易账户，Site Use 表示 OU 下的 Bill-To/Ship-To 用途。Profile 可在系统、Profile Class、Account、Site 层级默认，包括 Payment Terms、Credit Limit、Statement/Dunning、AutoCash 等。

## 设置与数据治理

- 统一 Party/Account 命名、注册号/税号、地址标准化和重复防护。
- 明确 Bill-To 与 Ship-To 关系、Primary Use、OU、Tax、Price List、Sales Territory。
- 信用检查还可受 OM Credit Check Rule、Order Type、Payment Terms、Credit Exposure 和外部信用数据影响。
- Merge 必须用 TCA Customer Merge 标准流程，并先评估 OM、AR、IBY、Tax、Install Base 等下游。

## SQL

```sql
SELECT hp.party_id, hp.party_number, hp.party_name,
       hca.cust_account_id, hca.account_number,
       hca.status account_status, hca.customer_type
  FROM hz_parties hp
  JOIN hz_cust_accounts hca ON hca.party_id = hp.party_id
 WHERE hca.account_number = :p_account_number;

SELECT hcasa.cust_acct_site_id, hcasa.cust_account_id,
       hcasa.org_id, hcasa.status site_status,
       hcsua.site_use_id, hcsua.site_use_code,
       hcsua.status use_status, hcsua.primary_flag,
       hcsua.payment_term_id, hcsua.location
  FROM hz_cust_acct_sites_all hcasa
  JOIN hz_cust_site_uses_all hcsua
    ON hcsua.cust_acct_site_id = hcasa.cust_acct_site_id
 WHERE hcasa.cust_account_id = :p_cust_account_id
 ORDER BY hcasa.org_id, hcsua.site_use_code;
```

## 排查

- Customer/Site LOV 缺失：检查 Account/Site/Site Use 状态、OU、用途、有效日期和职责 MOAC。
- Bill-To 不可用：检查 Site Use `BILL_TO`、Primary、Payment Term、Currency/Tax 必要数据。
- Credit Hold：分析 Credit Limit、Open AR、Open Orders、未入账交易、客户层级和 OM 规则，不直接更新 Hold Flag。
- 重复客户：同时比较名称、税号、地址、电话/邮箱，避免仅按名称误合并。

## 关联

- [AR 交易](transactions.md)
- [收款](receipts.md)
