# Credit to Cash： deductions-and-disputes

## 业务定位
本专题是 Credit to Cash 中的 deductions-and-disputes 子域。覆盖 TCA 客户、订单/发运边界、应收交易、信用、催收、自助、扣款/争议、Loans 和 C2C 内控。

## 设计与配置
以 Party/Account/Site 分层治理客户；定义交易来源/类型、AutoAccounting、收款方法、信用策略、催收和坏账审批；以交易、收款、会计和 GL 完成对账。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

## 数据、接口与会计追溯
HZ_PARTIES、HZ_CUST_ACCOUNTS、RA_CUSTOMER_TRX_ALL、RA_CUSTOMER_TRX_LINES_ALL、AR_PAYMENT_SCHEDULES_ALL、AR_CASH_RECEIPTS_ALL、AR_RECEIVABLE_APPLICATIONS_ALL、XLA_AE_HEADERS。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见问题与排查
将客户名称当唯一键；重传 AutoInvoice 造成重复交易；把账龄异常与信用/催收产品未部署混淆。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

## 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

## 关联与官方依据
[本知识域入口](../README.md)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
