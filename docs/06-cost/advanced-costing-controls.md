# 高级成本控制：差异、COGS、OPM/LCM 与关账风险

## 适用边界

本专题补充离散制造/库存成本中的差异和销售成本控制，并标识 OPM、Landed Cost Management（LCM）、项目制造等可选能力。是否安装产品、组织成本方法、估价账与会计规则均须以目标实例为准。

## 管理口径

| 主题 | 管理问题 | 关键数据链 |
| --- | --- | --- |
| 采购应计/价格差异 | 收货、发票与采购价格差异为何未清 | PO/RCV → AP → SLA/GL |
| WIP 差异 | 物料、资源、间接费、外协和产出差异是否合理 | WIP Job → 事务/成本 → 关闭/差异 → GL |
| COGS | 收入与销售成本是否在正确期间配比 | OM/Shipping → INV/COGS → SLA/GL |
| LCM/OPM | 附加成本或过程制造成本是否重复/遗漏分摊 | 业务交易 → 成本层/要素 → 会计事件 |

## 期间控制

1. 关闭前确认库存事务、接收、WIP 完工/关闭和成本处理器均已完成，先解决异常事务再关闭成本期间。
2. 分别审阅库存估值、在制品、收货应计、成本差异和 COGS；金额相等不代表事务数量、期间和科目均正确。
3. 标准成本更新、成本调整和追溯交易须有冻结期、批准、影响模拟和 GL 对账；避免在已签字期间直接重算。

## SQL：成本事务定位

```sql
-- 从物料事务开始定位；以组织、物料、日期/事务号缩小范围。
select mmt.transaction_id,
       mmt.organization_id,
       mmt.inventory_item_id,
       mmt.transaction_date,
       mmt.transaction_quantity,
       mmt.transaction_type_id,
       mmt.transaction_source_type_id,
       mmt.costed_flag
  from mtl_material_transactions mmt
 where mmt.organization_id = :p_organization_id
   and mmt.inventory_item_id = :p_inventory_item_id
   and mmt.transaction_date >= :p_from_date
   and mmt.transaction_date < :p_to_date + 1
 order by mmt.transaction_date, mmt.transaction_id;
```

## 排查原则

- `COSTED_FLAG` 或处理状态只能表明某一处理阶段，不能单独证明 SLA、GL 过账或报表已正确。
- 先按事务链检查来源与数量，再检查成本要素和会计，最后核对报表；不要通过直接更新成本/库存业务表处理异常。
- 对 OPM、LCM、项目制造等产品使用其官方指南、许可证与补丁级别验证对象和并发程序，避免将离散制造对象套用到不同成本模型。

## 官方参考

- [Oracle Supply Chain Management Documentation](https://docs.oracle.com/cd/E26401_01/nav/scm.htm)
