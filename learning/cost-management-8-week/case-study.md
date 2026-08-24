# 贯穿案例：华星制造标准成本与月结

> 用途：本案例贯穿 8 周学习。所有公司、组织、科目和单据号均为培训数据，不代表任何真实 Oracle EBS 实例。
>
> 环境限制：当前无可操作系统，因此标记为“纸面模拟”的结果只能证明概念、计算和顾问交付能力，不能作为系统配置或程序运行成功的证据。

## 1. 案例目标

你是华星制造有限公司的 Oracle EBS R12.2 成本模块初级功能顾问。公司采用离散制造和标准成本法，需要建立一条可解释、可复核的成本链：

```text
PO → RCV → INV → BOM/Routing → CST → WIP → SLA → GL → 月结对账
```

完成案例后，应能：

1. 说明组织、成本类型、成本要素和成本组的边界。
2. 手工卷积半成品 S 和成品 F 的标准成本。
3. 解释采购价差、WIP 投入、完工和材料用量差异。
4. 区分 Processed、Costed、Accounted、Transferred、Imported、Posted。
5. 用纸面证据完成标准成本更新评审、月结和问题分析。

## 2. 组织与会计假设

| 对象 | 培训值 | 说明 |
| --- | --- | --- |
| Ledger | CN_PRIMARY | 本位币 CNY；名称仅供练习 |
| Operating Unit | CN_MFG_OU | 采购、应付业务边界 |
| Item Master Organization | ITEM_MASTER | 维护共享物料定义，不承载本案例库存交易 |
| Inventory Organization | M1 | 离散制造、标准成本法 |
| 会计期间 | AUG-26 | 假设为当前开放练习期间 |
| Cost Type | Frozen / Pending | Frozen 为现行标准，Pending 为待更新成本 |
| Cost Group | M1_DEFAULT | 本案例不启用项目制造或 WMS 成本分区 |
| 数量单位 | EA | 所有物料均按件计算 |
| 金额单位 | CNY | 除非另有说明，金额保留两位小数 |

培训科目如下。真实项目必须以目标实例的组织、子库、WIP Accounting Class 和 SLA 账户派生为准。

| 培训科目 | 用途 |
| --- | --- |
| Receiving Inspection | 收货检验价值 |
| Receiving Accrual | 收货应计 |
| Raw Material Inventory | 原材料库存估价 |
| Semi-finished Inventory | 半成品库存估价 |
| Finished Goods Inventory | 成品库存估价 |
| WIP Valuation | 在制品投入与完工价值；答题时可按成本要素拆分 |
| Purchase Price Variance | 采购价格差异（PPV） |
| Invoice Price Variance | 发票价格差异（IPV） |
| Material Usage Variance | 材料用量差异 |
| Resource Absorption | 资源吸收 |
| Overhead Absorption | 制造费用吸收 |
| Standard Cost Adjustment | 标准成本更新重估差额 |
| AP Liability | 应付账款负债 |

## 3. 物料、BOM 与工艺路线

### 3.1 Frozen 标准成本基线

| 物料 | 类型 | Buy/Make | 标准成本 | 补充信息 |
| --- | --- | --- | ---: | --- |
| A | 原料 | Buy | 10.00 | 实际采购/PO 单价 11.00 |
| B | 原料 | Buy | 4.00 | 本案例不设置采购价差 |
| S | 半成品 | Make | 20.00 | 由 B、资源和制造费用构成 |
| F | 成品 | Make | 52.00 | 由 A、S、资源和制造费用构成 |

### 3.2 BOM 与 Routing 基线

| 装配件 | 成本来源 | 标准用量/费率 | 标准金额 | 成本要素 | 层级 |
| --- | --- | ---: | ---: | --- | --- |
| S | B | 3 EA × 4.00 | 12.00 | Material | This/Lower level 由卷积明细判断；在 S 汇总中为组件材料 |
| S | 资源 RS | 1 × 6.00 | 6.00 | Resource | This level |
| S | 制造费用 OHS | 1 × 2.00 | 2.00 | Overhead | This level |
| F | A | 2 EA × 10.00 | 20.00 | Material | 组件材料 |
| F | S | 1 EA × 20.00 | 20.00 | Material/下层要素明细 | Lower level 来源 |
| F | 资源 RF | 1 × 8.00 | 8.00 | Resource | This level |
| F | 制造费用 OHF | 1 × 4.00 | 4.00 | Overhead | This level |

案例基线不使用替代 BOM/Routing、Phantom、OSP、Yield 损失或 Scrap Factor。学习者仍需在配置工作簿中说明这些因素可能怎样影响 Rollup。

## 4. 业务数据包

### 4.1 采购与库存事件

以下事件用于第 5 周会计矩阵。除明确标为“扩展测试”外，均使用 Frozen 标准成本。

| 编号 | 业务事件 | 数量/金额 | 预期分析重点 |
| --- | --- | --- | --- |
| P01 | PO 收货 A 至 Receiving | 10 EA，PO 单价 11.00 | Receiving Inspection 与 Receiving Accrual |
| P02 | 将 P01 全部交付至原料资产子库 | 10 EA | 库存按标准成本 10.00；PPV |
| P03 | AP 按 PO 价格匹配发票 | 10 EA，发票单价 11.00 | 冲销应计；IPV 为 0 |
| I01 | 杂项接收 B | 2 EA，标准成本 4.00 | 库存与杂项账户 |
| I02 | 杂项发出 B | 1 EA，标准成本 4.00 | 杂项账户与库存 |
| I03 | A 从 RM_STORE 转至 LINE_STORE | 2 EA，两个资产子库账户不同 | 组织内价值转移；总库存价值不变 |
| P04 | 收货退回供应商 | 2 EA，PO 单价 11.00 | 按原收货链反向；独立练习，不并入 WIP 数量桥 |
| P05 | 扩展：发票单价改为 12.00 | 10 EA | 与 P03 二选一，计算 IPV；不可与 P03 同时入账 |

### 4.2 WIP 工单事件

为避免数量歧义，两个工单的计划和完工数量均为 1 EA，期初 WIP 均为 0。

**工单 WS-001：生产 1 EA 半成品 S**

| 顺序 | 事件 | 实际投入/产出 |
| ---: | --- | ---: |
| 1 | 发料 B | 3 EA × 4.00 = 12.00 |
| 2 | 资源计费 RS | 6.00 |
| 3 | 制造费用吸收 OHS | 2.00 |
| 4 | 完工 S 入库 | 1 EA × 20.00 = 20.00 |
| 5 | 关闭工单 | 无差异；期末 WIP = 0 |

**工单 WF-001：生产 1 EA 成品 F**

| 顺序 | 事件 | 标准 | 实际 |
| ---: | --- | ---: | ---: |
| 1 | 发料 A | 2 EA × 10.00 = 20.00 | **3 EA × 10.00 = 30.00** |
| 2 | 发料 S | 1 EA × 20.00 = 20.00 | 1 EA × 20.00 = 20.00 |
| 3 | 资源计费 RF | 8.00 | 8.00 |
| 4 | 制造费用吸收 OHF | 4.00 | 4.00 |
| 5 | 完工 F 入库 | 1 EA × 52.00 = 52.00 | 标准成本完工 52.00 |
| 6 | 关闭工单 | 标准投入 52.00 | 实际投入 62.00；识别不利差异 10.00 |

本案例只要求解释多领 1 EA A 形成的材料用量差异。不要把 A 的采购价 11.00直接作为 WIP 发料成本；在标准成本法下，本案例发料按 Frozen 标准成本 10.00。

### 4.3 标准成本更新独立快照

第 4 周使用独立快照，不与上述连续交易做数量合并：

- Pending 中 A 从 10.00 调整为 11.00；B、资源和制造费用费率不变。
- 重新 Rollup 后，S 仍为 20.00，F 应从 52.00 变为 54.00。
- 更新生效时仅有 A 现有量 10 EA；S、F 现有量和 WIP 均为 0。
- 因而本快照的库存重估金额为 `10 EA × (11.00 − 10.00) = 10.00`。

如果自行增加 S、F 现有量或未结 WIP，必须列出数量、旧/新成本和重估公式，不得直接套用 10.00。

## 5. 八周任务

| 周次 | 必做任务 | 最低纸面证据 |
| --- | --- | --- |
| 1 | 绘制组织关系和端到端业务/会计流程；说明模块边界 | 一张带箭头、状态和责任人的流程图；一页讲解提纲 |
| 2 | 选择成本方法；填写配置工作簿；区分 Cost Type、Element、Group | 方法对比及签字决策；配置字段、理由、依赖、验证项 |
| 3 | 手工 Rollup S 与 F；标注 this-level/lower-level 来源 | 算式、成本要素明细、复核人勾选；结果必须为 S=20、F=52 |
| 4 | 编写 Standard Cost Update SOP；演算独立快照 | Pending/Frozen 对比、影响模拟、审批门禁、回退/停止条件、重估分录 |
| 5 | 完成不少于 10 个事件的会计矩阵 | 每行含数量、单价、借贷、状态、来源报告；P05 与 P03 不可同时使用 |
| 6 | 完成两张工单成本桥并解释差异 | WS-001 闭合为 0；WF-001 的不利材料用量差异为 10 |
| 7 | 形成月结清单和三类故障分析 | 待处理事务、资源费率遗漏、收货/AP 不匹配；每项有根因、处理、复核证据 |
| 8 | 汇编顾问交付物并进行 30 分钟答辩 | 完整模板包、交叉引用、版本/审批记录、答辩提纲和问题记录 |

## 6. 纸面模拟证据规则

每份纸面交付物页首应注明：`PAPER SIMULATION / 未经 EBS 实机验证`。证据至少包含：

- 文档编号、版本、编制人、复核人、日期和案例事件编号。
- 输入来源、公式、金额单位、舍入规则和假设。
- 预期系统导航/请求/报表名称，以及未来实机应截取的关键字段。
- 实际纸面结果、预期结果、差异、结论和复核签字。
- 不得伪造请求 ID、Transaction ID、会计状态或系统截图；这些字段填写 `N/A—待实机验证`。

纸面证据可证明“知道如何做和如何验”，不能证明“系统中已经做过”。获得非生产环境后，应重新执行 Rollup/Update、库存/WIP 事务、SLA/GL 追踪和模拟月结。

## 7. 参考资料

- [成本模块导航](../../docs/07-cost-accounting.md#src-docs-06-cost-readme)
- [成本设置](../../docs/07-cost-accounting.md#src-docs-06-cost-setup)
- [成本方法](../../docs/07-cost-accounting.md#src-docs-06-cost-costing-methods)
- [成本要素与 Rollup](../../docs/07-cost-accounting.md#src-docs-06-cost-cost-elements)
- [会计流](../../docs/07-cost-accounting.md#src-docs-06-cost-accounting-flow)
- [期间关闭与报表](../../docs/07-cost-accounting.md#src-docs-06-cost-period-close-reports)
- [Inventory/WIP/Cost/GL 追踪](../../docs/09-end-to-end.md#src-docs-08-e2e-inventory-wip-cost-gl)
