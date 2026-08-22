# Assets and Projects： projects-foundation

## 业务定位
本专题是 Assets and Projects 中的 projects-foundation 子域。覆盖固定资产、iAssets、eAM、Projects、项目成本/开票/计划/合同/Grants、项目转资产、物业与租赁。

## 设计与配置
明确可资本化政策、项目/任务/资产类别/账簿、成本归集、资产行、Mass Additions、折旧和处置；项目成本、CIP、FA 和 GL 分别对账。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

## 数据、接口与会计追溯
FA_ADDITIONS_B、FA_BOOKS、FA_DISTRIBUTION_HISTORY、FA_DEPRN_SUMMARY、FA_MASS_ADDITIONS、PA_PROJECTS_ALL、PA_TASKS、PA_EXPENDITURE_ITEMS_ALL。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见问题与排查
把费用化和资本化成本混同；只看资产头忽略账簿/分配历史；未确认可选 Projects/Property/Lease 产品范围。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

## 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

## 关联与官方依据
[本知识域入口](../README.md)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
