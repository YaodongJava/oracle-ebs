# Treasury、现金头寸与银行主数据治理

## 适用范围

Oracle Treasury 是可选产品，处理资金交易、交易对手、限额、结算、估值和风险暴露；Cash Management 处理银行对账、现金头寸及预测。即使未部署 Treasury，内部银行账户、账户用途、签字权限、支付文件和外部对账单也必须作为跨财务模块的受控主数据。

## 治理模型

| 对象 | 数据所有者 | 变更控制 | 对账责任 |
| --- | --- | --- | --- |
| Bank / Branch | 财资主数据团队 | 银行证明、双人复核、有效期 | 银行档案与账户清单 |
| Internal Account / Use | 法人财务与资金 | 所有者、用途、币种、模块授权 | AP/AR/CE/IBY 使用范围 |
| Counterparty / Limit | 财资风险团队 | KYC/信用审批、额度与有效期 | 敞口与限额报表 |
| Statement / Transaction Code | 资金运营 | 文件格式、代码映射、测试回放 | 银行余额、未达项、自动核对率 |

## 现金头寸与预测

- 区分已确认银行余额、预计收款/付款、Treasury 结算、未达项和内部资金归集；不可将预测值当作已核对可用余额。
- 预测数据应携带来源、日期、币种、账户、置信状态和相关业务键，支持按账户/法人/币种/日期重算。
- 银行文件接口需要文件级哈希或唯一标识、防重复导入、原始文件受控留存、解析错误隔离和可审计重放。

## 只读诊断 SQL

```sql
-- 银行账户对象的可用列在不同补丁级别可能不同；先校验数据字典。
select cba.bank_account_id,
       cba.bank_account_name,
       cba.currency_code,
       cba.start_date,
       cba.end_date
  from ce_bank_accounts cba
 where cba.bank_account_id = :p_bank_account_id;

-- 对账单行诊断必须缩小到账户、日期或对账单，避免全表扫描。
select csl.statement_line_id,
       csl.trx_date,
       csl.amount,
       csl.trx_code,
       csl.reconciliation_status
  from ce_statement_lines csl
 where csl.statement_header_id = :p_statement_header_id
 order by csl.line_number;
```

## 官方参考

- [Oracle Treasury Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
- [Oracle Cash Management Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
