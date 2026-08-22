# Legal Entity Configurator

## 适用范围与业务说明
Legal Entity 表示法定、税务和财务责任主体；Establishment、Registration、Jurisdiction 和 Legal Address 补充法定地点与登记。

## 配置与实施要点
维护法人、登记、地址、管辖和生效日期；受控关联 Ledger、Balancing Segment、OU、银行和税务登记；设计跨法人交易与消除。

## 核心对象与诊断范围
XLE Entity、Registration、Ledger、Balancing Segment、Tax Registration。查询前在目标实例使用数据字典复核对象、列、状态值和访问权限；大表查询必须限制组织、账簿、日期或主键。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见风险与排查
用 OU 代替法人；变更法人/税号/地址时未评估 EBTax、发票、付款和本地化。 排查时先确认业务状态、组织/账簿/期间和权限上下文，再逐层检查接口、会计和报表结果。

## 关联与官方依据
[Financials Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48783/toc.htm)
