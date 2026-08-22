# Record to Report

## 范围与目标
覆盖总账、SLA、FAH、AGIS、预算、合并、二级账簿和月结，目标是使每个业务事件可从来源交易追溯至已过账日记账和报告。

## 运行与实施控制
先确认 Ledger、期间、Data Access、会计方法和来源系统；再验证事件、分录、传输、Journal Import、Posting、重估/折算/合并和报告。

## 核心数据对象
GL_LEDGERS、GL_JE_BATCHES、GL_JE_HEADERS、GL_JE_LINES、GL_BALANCES、XLA_TRANSACTION_ENTITIES、XLA_EVENTS、XLA_AE_HEADERS、XLA_AE_LINES。对象、列、状态和 API 签名须在目标实例 eTRM、Integration Repository 与数据字典复核。

## 与既有知识的关系
本目标目录新增详细入口；已有专题保留在 [04-gl/README](../04-gl/README.md) 并逐步迁移链接，不复制历史内容。

## 官方依据
[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
