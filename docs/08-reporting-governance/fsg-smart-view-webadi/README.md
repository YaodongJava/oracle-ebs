# Reporting and Governance： fsg-smart-view-webadi

## 业务定位
本专题是 Reporting and Governance 中的 fsg-smart-view-webadi 子域。覆盖关账与对账、财务/管理报告、BI Publisher/RXi、FSG/Smart View/Web ADI、ECC、内控、审计、保留、本地化和公共部门。

## 设计与配置
先明确报告口径、数据截止、Ledger/币种/组织范围和签字责任；报告、对账、审计和数据保留使用一致的业务键与期间。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

## 数据、接口与会计追溯
GL_BALANCES、GL_JE_HEADERS、XLA_AE_HEADERS、FND_CONCURRENT_REQUESTS、FND_AUDIT_TABLES、FND_LOGINS。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见问题与排查
报表只按显示金额判断正确；将业务修复留给报表层；未确认 ECC、公共部门、本地化和诊断工具的许可证/适用范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

## 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

## 关联与官方依据
[本知识域入口](../README.md)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
