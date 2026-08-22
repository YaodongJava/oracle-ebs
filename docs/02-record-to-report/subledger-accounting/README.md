# Record to Report： subledger-accounting

## 业务定位
本专题是 Record to Report 中的 subledger-accounting 子域。覆盖总账、SLA、FAH、AGIS、预算、合并、二级账簿和月结，目标是使每个业务事件可从来源交易追溯至已过账日记账和报告。

## 设计与配置
先确认 Ledger、期间、Data Access、会计方法和来源系统；再验证事件、分录、传输、Journal Import、Posting、重估/折算/合并和报告。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

## 数据、接口与会计追溯
GL_LEDGERS、GL_JE_BATCHES、GL_JE_HEADERS、GL_JE_LINES、GL_BALANCES、XLA_TRANSACTION_ENTITIES、XLA_EVENTS、XLA_AE_HEADERS、XLA_AE_LINES。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见问题与排查
子账差异被手工 GL 分录掩盖；未区分 Create Accounting、Transfer、Import 与 Posting 断点；跨 Ledger 汇率/转换规则未对账。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

## 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

## 关联与官方依据
[本知识域入口](../README.md)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
