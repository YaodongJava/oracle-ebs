# EBTax 税务报告、本地化与合规控制

## 原则

EBTax 负责交易税务确定，不等于自动满足某一国家/地区的全部申报、电子发票或归档要求。税务报告、本地化功能和外部税引擎均应按部署国家、法规版本、许可证、补丁和法定顾问意见实施。

## 从交易到申报的控制链

```text
Legal Entity / Registration / Party Tax Profile
  → Tax Regime / Tax / Status / Rate / Jurisdiction
  → Determining Factors / Applicability / Recovery
  → ZX Tax Line / AP-AR-PO-OM 交易
  → SLA / GL / Tax Reporting Ledger（如适用）
  → 法定报表、电子申报/外部系统、归档与对账
```

## 实施清单

- 每个法人/登记主体明确税号、注册地址、税务管辖、有效期、开票主体和报表责任人。
- 对标准、免税、零税率、反向计税、自行计税、代扣税、可抵扣/不可抵扣和复合税建立可测试的业务矩阵。
- 外部税引擎或本地化适配器应设计超时/不可用时的业务策略、版本控制、审计请求/响应摘要和日终对账。
- 税务申报前按交易、税行、税率、登记、会计期间和 GL 口径交叉核对；由税务负责人签字确认。

## SQL：税务行范围校验

```sql
-- ZX_LINES 的对象/字段需按目标补丁级别复核；查询必须按来源交易或日期收缩。
select zl.trx_id,
       zl.trx_line_id,
       zl.tax_regime_code,
       zl.tax,
       zl.tax_status_code,
       zl.tax_rate_code,
       zl.tax_amt,
       zl.taxable_amt
  from zx_lines zl
 where zl.trx_id = :p_trx_id
 order by zl.trx_line_id, zl.tax_line_id;
```

## 常见问题

- 税率未命中：按交易日期、注册、地点、产品/税分类、客户/供应商税务档案和确定因素逐层检查。
- 税务金额正确但申报不一致：检查报告口径、会计/申报期间、取消/冲销交易、税务登记有效期和外部提取批次。
- 国家本地化需求：不要以通用 EBTax 设置替代法定评估；需同步核对 Oracle 本地化文档、MOS 补丁和当地法规。

## 官方参考

- [Oracle E-Business Tax Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
- [Oracle Financials Localizations Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
