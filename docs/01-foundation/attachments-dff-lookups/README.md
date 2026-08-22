# 附件、DFF 与 Lookup

## 适用范围与业务说明
附件保存受控证明资料；DFF 扩展标准对象；Lookup 管理受控代码。三者不能替代核心主数据、审批或不受控集成字段。

## 配置与实施要点
定义所有者、类型、长度、必输、敏感级别和有效期；编译 DFF 并测试页面/接口/NLS；Lookup 采用受控发布和失效。

## 核心对象与诊断范围
FND Flexfield 元数据、Lookup、附件关系表、实体主键。查询前在目标实例使用数据字典复核对象、列、状态值和访问权限；大表查询必须限制组织、账簿、日期或主键。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见风险与排查
将账号、身份证明或密钥放入通用字段；删除历史 Lookup 值导致历史交易无法解释。 排查时先确认业务状态、组织/账簿/期间和权限上下文，再逐层检查接口、会计和报表结果。

## 关联与官方依据
[既有附件专题](../../01-common/attachments-dff.md)
