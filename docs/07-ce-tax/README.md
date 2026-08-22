# 现金管理、付款与税务（CE / IBY / EBTax）

本目录覆盖银行账户与用途、银行对账单、自动核对、现金预测、Oracle Payments 支付链和 E-Business Tax 税务确定。银行账户是跨 AP、AR、CE、IBY、Treasury 和 GL 的公共主数据，权限、加密、审批、回执与对账不可分割。

## 专题导航

- [银行、账户、对账单与自动核对](cash-management.md)
- [现金预测、清算与银行接口](cash-forecast-interfaces.md)
- [Treasury、现金头寸与银行主数据治理](treasury-bank-governance.md)
- [税种、税率、规则与排错](ebtax.md)
- [税务报告、本地化与合规控制](tax-reporting-localization.md)
- [表结构](tables.md)
- [银行对账单、支付、税务接口实现](interfaces.md)

## 运行控制

| 领域 | 必须对账的对象 | 关键例外 |
| --- | --- | --- |
| CE | 银行对账单余额、已核对/未核对 AP/AR/Treasury/GL 交易 | 重复导入、日期错位、银行交易代码映射错误 |
| IBY | PPR、付款指令、支付文件、传输、ACK、作废/重发 | 文件已发出但状态未回写、回执与付款状态不一致 |
| EBTax | 交易税行、税务登记、税率、Recoverability、税务报告 | 税务决定因素缺失、Tax Regime/Status/Rate 不适用、反向计税错误 |

## 安全边界

银行账号、证书、密钥、支付文件和税务身份信息不得写入示例或日志。接口只保存必要的掩码、哈希、外部业务键和审计相关号；敏感明细的查询权限应受职责和数据库最小授权约束。

## 官方依据

- [Oracle Cash Management Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
- [Oracle E-Business Tax Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
