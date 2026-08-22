# 导航与知识库治理

## 适用范围与业务说明
定义版本基线、阅读路径、文档治理和生产安全边界。它不替代 Oracle Support、许可证合同、客户变更流程或当地法规意见。

## 配置与实施要点
确认目标环境的 EBS、AD/TXK、数据库与已安装产品版本；按业务角色选择阅读路径；所有生产操作经过变更、回退和业务验证。

## 核心对象与诊断范围
版本快照、产品清单、配置工作簿、变更记录、验证证据。查询前在目标实例使用数据字典复核对象、列、状态值和访问权限；大表查询必须限制组织、账簿、日期或主键。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见风险与排查
把经验结论当作所有补丁级别的通用事实；未确认许可证便启用可选产品。 排查时先确认业务状态、组织/账簿/期间和权限上下文，再逐层检查接口、会计和报表结果。

## 关联与官方依据
[范围与版本](scope-and-version.md)｜[产品地图](financials-product-map.md)｜[按角色阅读](reading-paths-by-role.md)｜[按生命周期阅读](reading-paths-by-lifecycle.md)｜[文档规范](documentation-conventions.md)｜[生产安全](safety-and-production-boundaries.md)｜[官方资料](official-sources.md)
