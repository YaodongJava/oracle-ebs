# 范围、版本与适用性

## 适用范围与业务说明
默认适用 Oracle E-Business Suite R12.2.x；功能可用性受安装产品、许可证、国家本地化、数据库版本、AD/TXK 补丁和客户定制影响。

## 配置与实施要点
记录 EBS Release、ATG、AD/TXK、数据库 RU/RUR、WebLogic/OHS 和节点拓扑；升级、CPU、RUP 或 ADOP 后执行财务、接口、报表和安全回归。

## 核心对象与诊断范围
FND 产品信息、Context File、已安装产品、Integration Repository、eTRM、ALL_TAB_COLUMNS。查询前在目标实例使用数据字典复核对象、列、状态值和访问权限；大表查询必须限制组织、账簿、日期或主键。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见风险与排查
仅按模块名称推断功能；把旧环境的 SQL、API 签名或页面行为直接复制到新环境。 排查时先确认业务状态、组织/账簿/期间和权限上下文，再逐层检查接口、会计和报表结果。

## 关联与官方依据
[Oracle Documentation Library](https://docs.oracle.com/cd/E26401_01/index.htm)｜[Maintenance Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22954/toc.htm)
