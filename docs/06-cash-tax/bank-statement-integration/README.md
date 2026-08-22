# Cash, Treasury and Tax： bank-statement-integration

## 业务定位
本专题是 Cash, Treasury and Tax 中的 bank-statement-integration 子域。覆盖现金管理、Treasury、EBTax、Tax Reporting Ledger、银行对账单、现金预测、资金归集和外部税引擎。

## 设计与配置
银行账户按法人/用途/币种受控；文件接口按文件级唯一标识和控制总额管理；税务以登记、确定因素、税率、税行、报告和会计期间逐层验证。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

## 数据、接口与会计追溯
CE_BANK_ACCOUNTS、CE_STATEMENT_HEADERS、CE_STATEMENT_LINES、IBY_PAYMENTS_ALL、ZX_LINES、ZX_PARTY_TAX_PROFILE、ZX_REGISTRATIONS。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见问题与排查
泄露账号/证书/税号；把支付状态当作银行对账完成；把通用 EBTax 配置当作本地法定申报保证。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

## 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

## 关联与官方依据
[本知识域入口](../README.md)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
