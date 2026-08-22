# COA 与会计弹性域

## 适用范围与业务说明
Accounting Flexfield 由多个段组成；Balancing、Natural Account、Cost Center 等限定符影响平衡、报告、安全和会计派生。

## 配置与实施要点
设计段、值集、层级、有效期、属性和限定符；定义/Freeze/Compile Flexfield；配置交叉验证和安全规则；建立外部编码映射。

## 核心对象与诊断范围
FND_ID_FLEX_STRUCTURES、FND_FLEX_VALUE_SETS、FND_FLEX_VALUES、GL_CODE_COMBINATIONS。查询前在目标实例使用数据字典复核对象、列、状态值和访问权限；大表查询必须限制组织、账簿、日期或主键。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见风险与排查
用组合显示文本当唯一业务键；组合无效时未检查值有效期、CVR、安全规则和允许过账。 排查时先确认业务状态、组织/账簿/期间和权限上下文，再逐层检查接口、会计和报表结果。

## 关联与官方依据
[既有 COA 文档](../../01-common/coa.md)
