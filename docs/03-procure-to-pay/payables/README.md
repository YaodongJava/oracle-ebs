# Procure to Pay： payables

## 业务定位
本专题是 Procure to Pay 中的 payables 子域。覆盖供应商、采购/收货、应付发票、Payments、iExpenses、供应商协同和 P2P 内控，强调 PO/Receipt/Invoice/Payment/CE/GL 的闭环。

## 设计与配置
确认供应商与地点、采购匹配、容差/Hold、税务、付款方式、银行账户、审批和会计期间；每批接口必须有业务唯一键、控制总额和重跑策略。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

## 数据、接口与会计追溯
PO_HEADERS_ALL、PO_LINES_ALL、RCV_TRANSACTIONS、AP_INVOICES_ALL、AP_INVOICE_LINES_ALL、AP_INVOICE_DISTRIBUTIONS_ALL、AP_PAYMENT_SCHEDULES_ALL、AP_CHECKS_ALL、XLA_AE_HEADERS。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见问题与排查
重复供应商或发票；付款文件成功被误判为银行付款成功；收货应计、AP 负债、付款和 GL 未按期间对账。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

## 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

## 关联与官方依据
[本知识域入口](../README.md)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
