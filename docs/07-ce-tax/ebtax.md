# Oracle E-Business Tax（EBTax）税种、税率、规则与排错

## 税务确定模型

```text
Configuration Owner / Legal Entity / OU
→ Tax Regime → Tax → Tax Status → Tax Rate / Jurisdiction
→ Party Tax Profile / Registration / Exemption
→ Determining Factors + Rules
→ Tax Lines / Recoverability / Accounting
```

EBTax 是一套中央税引擎。应用产品（AP/AR/PO/OM）传入交易日期、法人/OU、交易业务类别、产品财政分类、交易方税务分类、Ship From/To/Bill From/To 等确定因素；规则按优先级确定 Applicable Tax、Place of Supply、Status、Rate、Taxable Basis、Recovery 等。

## 配置

1. 确定 Configuration Owner Tax Options 和 Party Tax Profile/Registrations。
2. 定义 Regime/Tax/Status/Rate/Jurisdiction，检查有效期与地理范围。
3. 定义 Fiscal Classifications、Tax Zones、Determining Factor Sets/Conditions/Rules。
4. 定义 Tax Accounts、Recovery Rates、Exemptions、Thresholds、Inclusive/Compound Tax。
5. 按 AP/AR/PO/OM、手工/接口、Credit/Return、预付、外币、不同地址组合建立测试矩阵。

## SQL

```sql
-- 税行与源单据；ENTITY/EVENT_CLASS 按产品解读
SELECT zxl.tax_line_id, zxl.application_id,
       zxl.entity_code, zxl.event_class_code,
       zxl.trx_id, zxl.trx_line_id, zxl.trx_level_type,
       zxl.tax_regime_code, zxl.tax, zxl.tax_status_code,
       zxl.tax_rate_code, zxl.tax_rate, zxl.taxable_amt,
       zxl.tax_amt, zxl.tax_amt_funcl_curr,
       zxl.cancel_flag, zxl.delete_flag
  FROM zx_lines zxl
 WHERE zxl.application_id = :p_application_id
   AND zxl.trx_id = :p_trx_id
 ORDER BY zxl.tax_line_id;

SELECT tax_regime_code, tax, tax_status_code,
       tax_rate_code, percentage_rate,
       effective_from, effective_to, active_flag
  FROM zx_rates_b
 WHERE tax_regime_code = :p_tax_regime_code
   AND tax = :p_tax
 ORDER BY effective_from;
```

## 排查方法

1. 锁定 Application/Entity/Event Class/Trx ID/Line ID，不只用发票号查 ZX。
2. 重建税确定因素快照：Owner/LE/OU、Date、Party Registrations、Locations、Product/Party Fiscal Classification。
3. 按 Applicable Tax → Place of Supply → Status → Rate → Basis → Recovery 的顺序查规则、优先级和默认。
4. 比较正常/异常单据的所有确定因素，而不是直接比较最终 Tax Rate。

常见问题：税不计算通常是 Owner/Regime Applicability/Date/Place of Supply 不匹配；税率错通常是 Status/Rate Rule 优先级或 Registration/Class 不同；进项税不可抵扣要检查 Recovery Rule/Rate/Account；会计错要区分 ZX Tax Line 正确但 SLA 账户派生错的情况。

## 关联

- [AP Invoices](../02-ap/invoices.md)
- [AR Transactions](../03-ar/transactions.md)
- [SLA](../01-common/sla.md)

## 官方参考

- [Oracle Financials Implementation Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48783/T348488T348491.htm)
