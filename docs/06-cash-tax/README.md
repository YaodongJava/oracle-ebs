# Cash, Treasury and Tax

## 范围与目标
覆盖现金管理、Treasury、EBTax、Tax Reporting Ledger、银行对账单、现金预测、资金归集和外部税引擎。

## 运行与实施控制
银行账户按法人/用途/币种受控；文件接口按文件级唯一标识和控制总额管理；税务以登记、确定因素、税率、税行、报告和会计期间逐层验证。

## 核心数据对象
CE_BANK_ACCOUNTS、CE_STATEMENT_HEADERS、CE_STATEMENT_LINES、IBY_PAYMENTS_ALL、ZX_LINES、ZX_PARTY_TAX_PROFILE、ZX_REGISTRATIONS。对象、列、状态和 API 签名须在目标实例 eTRM、Integration Repository 与数据字典复核。

## 与既有知识的关系
本目标目录新增详细入口；已有专题保留在 [07-ce-tax/README](../07-ce-tax/README.md) 并逐步迁移链接，不复制历史内容。

## 官方依据
[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
