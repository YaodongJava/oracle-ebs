# 集中式银行账户模型

## 适用范围与业务说明
内部银行、分行、账户、所有者、用途与支付/收款/对账权限形成 CE/IBY 银行模型，且跨法人、OU、币种和模块受控。

## 配置与实施要点
建立银行/分行/账户/用途；配置 AP、AR、CE、IBY、Treasury 的可用范围；实施掩码、双人复核、证书保管和定期复核。

## 核心对象与诊断范围
CE_BANK_ACCOUNTS、账户使用对象、IBY 外部账户和支付对象。查询前在目标实例使用数据字典复核对象、列、状态值和访问权限；大表查询必须限制组织、账簿、日期或主键。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见风险与排查
将敏感账号输出到日志；文件生成成功便认为银行已经受理或 CE 已对账。 排查时先确认业务状态、组织/账簿/期间和权限上下文，再逐层检查接口、会计和报表结果。

## 关联与官方依据
[现金与税务目录](../../07-ce-tax/README.md)
