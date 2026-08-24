# 贯穿案例答案与复核键

> PAPER SIMULATION / 未经 EBS 实机验证
>
> 本答案用于自评和导师复核。典型分录以 [case-study.md](case-study.md) 的培训科目为准；真实实例的账户、会计行类型和生成时点必须以实际设置、SLA 和报表为准。

## 1. 核心概念答案

| 概念 | 合格答案要点 | 常见错误 |
| --- | --- | --- |
| Cost Type | 一套物料、资源、间接费成本表述；Frozen 常作现行标准，Pending/自定义类型用于模拟或更新 | 把它说成库存价值分区 |
| Cost Element | Material、Material Overhead、Resource、Outside Processing、Overhead 五类成本构成 | 把 Department 或 Cost Group 当作第六要素 |
| Cost Group | 特定项目制造/WMS 等场景下，同一组织库存的成本分区 | 把它等同 Cost Type |
| Inventory Organization | 库存和制造交易边界，并承载成本方法等关键参数 | 把 Item Master Organization 当作本案例交易组织 |
| Standard Cost | 交易通常按 Frozen 标准计价，实际与标准的偏差进入相应差异 | 用采购实际价 11 直接给 WIP 发料 A 计价 |
| Complete 与 Closed | Complete 表示完工状态；Closed 后才按规则识别/结转工单差异 | 完工后立即假设差异已全部入账 |

## 2. 第 1 周：流程与状态

合格的端到端图至少包含：

```text
PO 订单
  ↓
RCV 收货/交付 → INV 数量与估价 → BOM/Routing → CST Rollup/Update
                                      ↓
                          WIP 发料/资源/间接费/完工/关闭
                                      ↓
                             成本分录 → SLA → GL
```

六个状态必须分开说明：

| 状态 | 回答的问题 | 未来实机证据示例 |
| --- | --- | --- |
| Processed | 业务事务是否进入基表/业务处理成功 | 事务编号、数量、日期、来源 |
| Costed | 成本处理器是否完成计价 | Costed Flag、成本处理日志、成本分布 |
| Accounted | SLA 是否创建会计 | 会计事件/分录状态、AE Header/Line |
| Transferred | SLA 分录是否传送 GL | Transfer Status/Date |
| Imported | GL Import 是否生成日记账 | Journal Import 结果、批次/日记账 |
| Posted | 日记账是否已过账余额 | Posted Status、过账日期、余额 |

只看到 `Costed` 不能推断 `Posted`。纸面模拟中，这些系统标识均应写 `N/A—待实机验证`，并写明预期取证位置。

## 3. 第 2 周：成本方法选择

本案例应选择 **Standard Cost**，理由是题目要求 Frozen 标准、PPV/IPV、WIP 用量差异和 Standard Cost Update。其他方法可作为对比，但不可混入本案例计算。

| 方法 | 计价基础 | 本案例适用性 |
| --- | --- | --- |
| Standard | Frozen Standard Cost | 主线；可清楚分析价格、用量、资源和间接费差异 |
| Average | 交易后加权平均 | 仅概览；负库存、补录和交易顺序会影响成本 |
| FIFO/LIFO | 成本层 | 不在本期主线 |
| Periodic | 期末周期成本处理 | 不等于当前 Item Cost 查询；不在本期主线 |

配置工作簿中至少应锁定：M1、Standard、Frozen/Pending、五大成本要素、资源/制造费用、WIP Accounting Class、资产子库和关键估价/差异/吸收账户。M1_DEFAULT 只是本案例默认成本组，不能据此推断启用了项目成本分区。

## 4. 第 3 周：成本卷积

### 4.1 半成品 S

```text
Material = 3 × B 标准成本 4.00 = 12.00
Resource = 6.00
Overhead = 2.00
S 标准成本 = 12.00 + 6.00 + 2.00 = 20.00
```

| 成本要素 | 金额 |
| --- | ---: |
| Material | 12.00 |
| Resource | 6.00 |
| Overhead | 2.00 |
| 合计 | **20.00** |

### 4.2 成品 F

```text
A 组件 = 2 × 10.00 = 20.00
S 组件 = 1 × 20.00 = 20.00
Resource = 8.00
Overhead = 4.00
F 标准成本 = 20.00 + 20.00 + 8.00 + 4.00 = 52.00
```

| 来源 | 金额 | 解释 |
| --- | ---: | --- |
| A | 20.00 | F 的组件材料 |
| S | 20.00 | 下层装配件带入的成本；明细可继续追溯 B/RS/OHS |
| RF | 8.00 | F 本层资源 |
| OHF | 4.00 | F 本层制造费用 |
| 合计 | **52.00** | 与 Frozen 基线一致 |

如果展开 S，F 的最终来源为 B 12 + S 本层资源 6 + S 本层制造费用 2 + A 20 + F 本层资源 8 + F 本层制造费用 4 = 52。优秀答案会区分“装配件汇总显示”和“成本要素/层级明细显示”，不会把 S 的 20 全部武断地标成单一 Material Element。

## 5. 第 4 周：标准成本更新

Pending 中只把 A 从 10.00 调至 11.00：

| 物料 | Frozen | Pending Rollup | 单位变化 | 原因 |
| --- | ---: | ---: | ---: | --- |
| A | 10.00 | 11.00 | +1.00 | 手工/导入新的模拟材料标准 |
| B | 4.00 | 4.00 | 0.00 | 未变化 |
| S | 20.00 | 20.00 | 0.00 | 只依赖 B、RS、OHS |
| F | 52.00 | 54.00 | +2.00 | 每件 F 使用 2 EA A |

独立快照仅有 A 现有量 10 EA，S/F/WIP 为 0：

```text
A 库存重估 = 10 × (11.00 − 10.00) = +10.00
```

训练用典型分录：借 Raw Material Inventory 10.00，贷 Standard Cost Adjustment 10.00。若成本下降，方向相反；真实科目和行类型以目标实例为准。

合格 SOP 必须在执行 Update 前包含：冻结窗口、BOM/Routing 截止点、Pending/Frozen 对比、Rollup 日志、库存/WIP 影响模拟、账户审核、审批和停止条件。更新后必须保存程序结果、重估分录、SLA/GL 状态和对账。无环境时只能形成签字纸面包，不得填造 Request ID。

## 6. 第 5 周：业务事件会计矩阵答案

下表提供 15 个可用事件。所有借贷均为训练用典型表达；P05 是 P03 的替代扩展，二者不能同时计入同一账套情景。

| 事件 | 金额 | 借方 | 贷方 | 说明 |
| --- | ---: | --- | --- | --- |
| P01 PO 收货 A 10×11 | 110.00 | Receiving Inspection | Receiving Accrual | 按 PO 收货价值 |
| P02 交付 A 至库存 | 100.00 | Raw Material Inventory | Receiving Inspection | 标准库存价值 |
| P02 识别 PPV | 10.00 | Purchase Price Variance | Receiving Inspection | PO 11 − 标准 10，不利差异 |
| P03 AP 发票 10×11 | 110.00 | Receiving Accrual | AP Liability | 与 PO 同价，IPV=0 |
| I01 杂项接收 B 2×4 | 8.00 | Raw Material Inventory | Miscellaneous Offset | 资产增加 |
| I02 杂项发出 B 1×4 | 4.00 | Miscellaneous Offset | Raw Material Inventory | 资产减少 |
| I03 组织内资产子库转移 A 2×10 | 20.00 | LINE_STORE Inventory | RM_STORE Inventory | 总库存价值不变；以子库账户配置为前提 |
| WS-001 发料 B 3×4 | 12.00 | WIP Valuation—Material | Raw Material Inventory | 工单投入 |
| WS-001 资源 | 6.00 | WIP Valuation—Resource | Resource Absorption | 工单投入 |
| WS-001 制造费用 | 2.00 | WIP Valuation—Overhead | Overhead Absorption | 工单投入 |
| WS-001 完工 S | 20.00 | Semi-finished Inventory | WIP Valuation | 可按要素拆分贷方 |
| WF-001 发料 A 与 S | 50.00 | WIP Valuation—Material | Raw Material Inventory 30；Semi-finished Inventory 20 | 实际多领 A |
| WF-001 资源 | 8.00 | WIP Valuation—Resource | Resource Absorption | 与标准一致 |
| WF-001 制造费用 | 4.00 | WIP Valuation—Overhead | Overhead Absorption | 与标准一致 |
| WF-001 完工 F | 52.00 | Finished Goods Inventory | WIP Valuation | 按 Frozen 标准完工 |
| WF-001 关闭差异 | 10.00 | Material Usage Variance | WIP Valuation | 清零剩余 WIP；不利差异 |

P01/P02 合并复核：Receiving Inspection 借 110、贷 100+10，余额为 0。P01/P03 合并复核：Receiving Accrual 贷 110、借 110，余额为 0。

扩展 P05：若 PO 仍为 11、发票改为 12，则发票金额 120，典型答案为借 Receiving Accrual 110、借 Invoice Price Variance 10、贷 AP Liability 120。具体 IPV 生成逻辑受匹配、设置和税费等影响，实机必须取证。

P04 收货退供应商应沿原接收链反向处理。由于题目指定它是独立练习，不应把退货后的数量硬塞进 WS/WF 工单数量桥。

## 7. 第 6 周：WIP 成本桥

### 7.1 WS-001

```text
期初 WIP 0
+ Material 12
+ Resource 6
+ Overhead 2
− Completion 20
− Variance 0
= 期末 WIP 0
```

### 7.2 WF-001

```text
期初 WIP 0
+ Material：A 30 + S 20 = 50
+ Resource 8
+ Overhead 4
− Completion 52
− Variance 10
= 期末 WIP 0
```

材料用量差异：

```text
(实际用量 3 − 标准用量 2) × A Frozen 标准成本 10 = 10 不利
```

判断点：

- 差异来自多领 1 EA A，不是采购价差。
- A 的 PO 单价 11 与 Frozen 标准 10 的差在采购链识别为 PPV；本题不把它再次计入 WIP 用量差异。
- WF-001 在完工后仍有 WIP 余额 10；关闭并识别差异后才归零。

## 8. 第 7 周：月结与三类故障

合格关期顺序：冻结当期补录 → 清理 Inventory/Receiving/WIP/Cost 待处理 → 确保各类交易已 Cost 并创建会计 → 处理 WIP 完工/关闭/差异和 COGS → 对账估值/应计/WIP/COGS/差异/GL → 保存关期报表 → 关闭 Inventory Period。

### 8.1 对账公式

```text
数量：期末 On-hand = 期初 + Receipts + Completions + Transfers In
                    − Issues − Sales − Transfers Out ± Adjustments

WIP：期末 WIP = 期初 WIP + Material + Resource + OSP + Overhead
               − Completion/Return − Variance

GL：相关 GL 余额 = 成本子账 + SLA 未转 + GL 未过账 + 可识别手工调整
```

### 8.2 故障案例参考答案

| 故障 | 现象/断点 | 应检查 | 合格处理 | 复核证据 |
| --- | --- | --- | --- | --- |
| 未成本事务 | 业务事务存在但 Costed 未完成；下游无完整成本分录 | 期间、Item Cost、账户、前置事务、Cost Manager、错误解释 | 修正配置/数据来源后用标准处理流程重跑；禁止直接更新业务表 | 原错误、处理日志、成本分布、前后状态 |
| 资源费率遗漏 | Routing 有资源用量但 Rollup/工单资源金额为 0 | Costed Flag、Resource Rate/Cost Type、UOM、Department/Resource、有效日期 | 在正确成本类型补全费率，重新 Rollup/模拟；对已发生成本按批准方案处理 | 费率表、Rollup 前后对比、影响和审批 |
| 收货/AP 不匹配 | 例如收货 10×11、当期只匹配发票 9×11，Receiving Accrual 尚有贷余 11 | PO Distribution、Receipt、Invoice Distribution、数量/价格/汇率、退货更正、截止日 | 确认是时间性未开票还是异常；次期匹配或按标准应计清理流程处理 | 三方明细、账龄、责任人、后续结清记录 |

任何答案建议直接改 `MTL_*`、`WIP_*`、`CST_*` 等业务基表，均判为不通过。

## 9. 第 8 周：交付物与答辩

### 9.1 交付物完整性

- 流程图能从业务来源追到 SLA/GL，状态无跳步。
- 术语表、配置工作簿、成本测算和会计矩阵使用同一组织/物料/版本。
- SOP 有执行前门禁、执行步骤、停止条件和执行后对账。
- SIT/UAT 脚本同时覆盖正常、反向、异常和关期场景。
- 月结表能把未成本、未会计、未传 GL、未导入、未过账分开。
- 问题日志包含证据、影响、责任人、临时措施和关闭标准。

### 9.2 30 分钟答辩建议

| 时间 | 内容 |
| ---: | --- |
| 0–3 分钟 | 业务目标、范围、环境限制 |
| 3–7 分钟 | 组织和端到端流程 |
| 7–12 分钟 | 成本模型与 S/F Rollup |
| 12–17 分钟 | 采购、库存和 WIP 会计 |
| 17–21 分钟 | 标准成本更新控制 |
| 21–25 分钟 | 月结、对账和三类故障 |
| 25–28 分钟 | 测试、切换与待实机验证事项 |
| 28–30 分钟 | 结论与提问 |

## 10. 评分复核表

| 维度 | 分值 | 满分证据 |
| --- | ---: | --- |
| 成本原理与计算 | 25 | S=20、F=52、更新后 F=54、WF 差异=10，公式和层级均可复核 |
| 配置设计 | 20 | 关键配置、账户、依赖、风险、验证项和审批完整 |
| 业务及会计链 | 20 | ≥10 个事件，借贷平衡，六状态及证据分开 |
| WIP 差异与月结对账 | 20 | 两张 WIP 桥闭合；三类故障和关期清单完整 |
| 顾问交付物与表达 | 15 | 文档版本一致、可追踪、30 分钟内清晰答辩 |
| 合计 | **100** | **80 分及格** |

否决项：存在无法解释的差异、建议直接修改业务基表，或关键测试只有结论而没有输入/步骤/预期/证据。无系统环境本身不扣分，但必须如实标记纸面模拟和待实机验证项。

