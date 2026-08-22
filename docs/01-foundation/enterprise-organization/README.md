# 企业组织结构

## 适用范围与业务说明
Business Group、Legal Entity、Ledger、Operating Unit、Inventory Organization、Location 和组织分类分别承担人力/安全、法定、会计、交易、库存/成本和地点职责。

## 配置与实施要点
绘制法人、经营单位、库存组织、地点、银行和税务登记关系；定义组织分类/有效期；用 P2P、O2C、库存、资产样例验证。

## 核心对象与诊断范围
HR_ALL_ORGANIZATION_UNITS_F、HR_ORGANIZATION_INFORMATION_F、XLE_ENTITY_PROFILES、GL_LEDGERS、HR_LOCATIONS_ALL。查询前在目标实例使用数据字典复核对象、列、状态值和访问权限；大表查询必须限制组织、账簿、日期或主键。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见风险与排查
把 OU 当法人或把库存组织当成本中心；看不到组织时忽略有效期、分类和 Security Profile。 排查时先确认业务状态、组织/账簿/期间和权限上下文，再逐层检查接口、会计和报表结果。

## 关联与官方依据
[既有多组织详解](../../01-common/organization.md)
