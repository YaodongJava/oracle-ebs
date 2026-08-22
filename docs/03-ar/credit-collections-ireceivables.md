# 信用管理、催收、争议与客户自助

## 产品边界

Oracle Receivables 保存交易、收款和未结余额；Credit Management 对客户信用审核/额度提供决策；Advanced Collections 管理逾期、策略、承诺付款和催收工作；iReceivables/Bill Presentment 提供客户自助查询、在线付款等能力。它们均依赖 TCA 与 AR 的准确客户层级和账龄。

## 业务流程

```text
TCA Party / Account / Site
  → Credit Profile / Credit Review（可选）
  → AR Transaction / Payment Schedule
  → Aging / Delinquency
  → Strategy / Work Item / Dunning / Promise to Pay（可选）
  → Dispute / Adjustment / Write-off（审批）
  → Receipt / Application / Recovery / Reporting
```

## 配置与治理清单

- 明确信用额度的层级（客户、账户、地点或业务单元）和审批责任，避免 OM/AR 使用不同的信用口径。
- 定义账龄桶、逾期日、催收等级、策略、工作项、信函模板和承诺付款的到期跟踪。
- 争议、扣款、坏账、核销、重分类和追回必须区分业务原因、审批权限、会计事件和审计证据。
- 客户自助/在线支付仅在已部署产品、DMZ/SSO、安全证书和隐私要求均满足时启用。

## 常用诊断 SQL

```sql
-- AR 未结余额是账龄和催收的基础；金额字段须按报表口径核实。
select aps.customer_id,
       aps.customer_trx_id,
       aps.payment_schedule_id,
       aps.due_date,
       aps.amount_due_original,
       aps.amount_due_remaining,
       aps.status
  from ar_payment_schedules_all aps
 where aps.org_id = :p_org_id
   and aps.status = 'OP'
   and aps.amount_due_remaining <> 0
   and aps.due_date < :p_as_of_date
 order by aps.customer_id, aps.due_date;

-- 客户账户与地点的有效性必须从 TCA 维度确认，避免仅按显示名称处理。
select hca.account_number,
       hp.party_name,
       hca.status
  from hz_cust_accounts hca
  join hz_parties hp
    on hp.party_id = hca.party_id
 where hca.cust_account_id = :p_cust_account_id;
```

## 排错方法

1. 先确认客户/账户/地点、交易与收款应用是否处于预期状态；账龄异常常由未应用、反应用、冲销或错误截止日期引起。
2. 再检查信用/催收产品是否已安装且数据同步/并发程序正常，不将未部署产品的对象误作基础 AR 缺陷。
3. 对争议和坏账先核对批准、原因码、原交易/收款关系和 SLA，再核查 GL 差异。

## 官方参考

- [Oracle Credit Management Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
- [Oracle Advanced Collections Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
- [Oracle iReceivables Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
