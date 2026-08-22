# Cost Accounting： process-manufacturing-costing

## 业务定位
本专题是 Cost Accounting 中的 process-manufacturing-costing 子域。覆盖采购收货会计、库存、离散/过程制造、WIP、LCM、COGS、项目制造、eAM 成本和 SCM 到 SLA/GL。

## 设计与配置
按组织、物料、事务、成本期间、成本要素和会计事件控制；先清理异常事务和成本处理器，再执行关期与差异对账。
上线前以正常、跨期、外币/多组织（如适用）、拒绝、冲销/撤回、重跑和月结场景完成验证。

## 数据、接口与会计追溯
MTL_MATERIAL_TRANSACTIONS、MTL_TRANSACTIONS_INTERFACE、CST_ITEM_COSTS、CST_ITEM_COST_DETAILS、WIP_ENTITIES、WIP_DISCRETE_JOBS、XLA_AE_HEADERS。接口必须维护来源业务键、批次号、行号、状态、错误码和可重放策略；会计追溯按交易主键、事件、分录和 GL 链分层进行。

```sql
select owner, table_name, column_name, data_type
  from all_tab_columns
 where owner = upper(:p_owner)
   and table_name = upper(:p_table_name)
 order by column_id;
```

## 常见问题与排查
仅看数量或余额忽略事务链；在签字期间直接重算成本；对 OPM/LCM 使用不适用的离散制造对象。 先确认产品是否已安装、职责和组织/账簿上下文是否正确，再检查业务状态、接口批次、并发日志、SLA 和报告。

## 实施边界
不直接 DML Oracle EBS 业务表。写入使用标准页面、公开 API、Open Interface 或受控自定义对象；可选产品需确认许可证、安装和补丁范围。

## 关联与官方依据
[本知识域入口](../README.md)｜[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
