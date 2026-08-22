# Cost Accounting

## 范围与目标
覆盖采购收货会计、库存、离散/过程制造、WIP、LCM、COGS、项目制造、eAM 成本和 SCM 到 SLA/GL。

## 运行与实施控制
按组织、物料、事务、成本期间、成本要素和会计事件控制；先清理异常事务和成本处理器，再执行关期与差异对账。

## 核心数据对象
MTL_MATERIAL_TRANSACTIONS、MTL_TRANSACTIONS_INTERFACE、CST_ITEM_COSTS、CST_ITEM_COST_DETAILS、WIP_ENTITIES、WIP_DISCRETE_JOBS、XLA_AE_HEADERS。对象、列、状态和 API 签名须在目标实例 eTRM、Integration Repository 与数据字典复核。

## 与既有知识的关系
本目标目录新增详细入口；已有专题保留在 [06-cost/README](../06-cost/README.md) 并逐步迁移链接，不复制历史内容。

## 官方依据
[Oracle Financials Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
