# TCA Party Model

## 适用范围与业务说明
TCA 将 Party、Party Site、Location、Customer Account、Account Site 和 Contact 分层；供应商与客户可共享 Party 但业务账户和用途不同。

## 配置与实施要点
定义外部主数据键、去重和匹配优先级；分别维护 Party/地址/账户/地点用途/联系人/有效期；治理合并和失效。

## 核心对象与诊断范围
HZ_PARTIES、HZ_PARTY_SITES、HZ_LOCATIONS、HZ_CUST_ACCOUNTS、HZ_CUST_ACCT_SITES_ALL。查询前在目标实例使用数据字典复核对象、列、状态值和访问权限；大表查询必须限制组织、账簿、日期或主键。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见风险与排查
以显示名称作为唯一键；合并主数据前未评估开放交易、信用、税务、银行和接口映射。 排查时先确认业务状态、组织/账簿/期间和权限上下文，再逐层检查接口、会计和报表结果。

## 关联与官方依据
[AR 客户与信用](../../03-ar/customers-credit.md)
