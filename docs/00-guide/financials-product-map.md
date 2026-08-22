# 财务产品地图与边界

## 适用范围与业务说明
按 Record to Report、Procure to Pay、Credit to Cash、Acquire to Retire、Cash and Tax、Cost Accounting 映射产品及相邻边界。

## 配置与实施要点
明确每一业务能力的产品负责人、数据所有者、会计责任、接口来源和关账责任；将可选产品单独标记。

## 核心对象与诊断范围
GL、SLA、AP、AR、FA、CE、IBY、EBTax、Projects、Inventory、WIP、Costing。查询前在目标实例使用数据字典复核对象、列、状态值和访问权限；大表查询必须限制组织、账簿、日期或主键。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见风险与排查
把 IBY 当作 AP 负债层，或把 CE 当作支付执行层；重复维护 SLA 的通用规则。 排查时先确认业务状态、组织/账簿/期间和权限上下文，再逐层检查接口、会计和报表结果。

## 关联与官方依据
[Oracle Financials](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
