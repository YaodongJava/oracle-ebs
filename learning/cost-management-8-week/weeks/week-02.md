# 第 2 周：成本模型与基础设置

> 本周定位：理解“组织采用哪种计价逻辑、用哪套成本版本、成本由哪些要素构成”。贯穿案例固定采用离散制造和标准成本；其他成本方法只做对比。

## 学习目标

完成本周后，你应能够：

1. 比较 Standard、Average、FIFO/LIFO 和 Periodic 的计价逻辑与主要风险。
2. 解释为什么成本方法是库存组织的高风险基础决策。
3. 准确区分 Cost Type、Cost Element 和 Cost Group。
4. 说明 Frozen、Pending 和自定义模拟成本类型在标准成本生命周期中的角色。
5. 识别五大成本要素及其典型来源。
6. 产出一份无需系统环境即可评审的成本基础配置工作簿。

## 6–8 小时时间安排（建议 7 小时）

| 环节 | 时间 | 产出 |
| --- | ---: | --- |
| 阅读成本方法与设置资料 | 1.5 小时 | 成本方法速记 |
| 编制成本方法对比表 | 1 小时 | 四类方法决策矩阵 |
| 学习 Cost Type/Element/Group | 1.5 小时 | 三类对象辨析表 |
| 编制案例配置工作簿 | 2 小时 | 组织、成本类型、要素和控制页 |
| 自测与错题订正 | 0.5 小时 | 自测记录 |
| 口头评审与复盘 | 0.5 小时 | 选型说明 |

## 必读本地资料

1. [成本模块导航](../../../docs/07-cost-accounting.md#src-docs-06-cost-readme)：理解本模块的范围和高风险业务事件。
2. [标准、平均与周期成本](../../../docs/07-cost-accounting.md#src-docs-06-cost-costing-methods)：重点阅读方法对比和关键原理。
3. [成本组织、成本类型与成本组设置](../../../docs/07-cost-accounting.md#src-docs-06-cost-setup)：重点阅读核心模型和设置顺序。
4. [物料、资源、间接费与成本更新](../../../docs/07-cost-accounting.md#src-docs-06-cost-cost-elements)：本周先掌握五大要素，卷积细节留到第 3 周。

## 概念讲解

### 1. 成本方法回答“业务事务用什么价值计量”

| 成本方法 | 价值基础 | 典型特点 | 本阶段关注的主要风险 |
| --- | --- | --- | --- |
| Standard | 当前生效的 Frozen 标准成本 | 交易按标准计价，实际与标准差异单独分析 | 标准过时、更新影响未评审、差异账户失控 |
| Average | 交易后形成的加权平均成本 | 收货、完工或调整可能改变平均成本 | 负库存、补录日期和交易顺序造成跳变 |
| FIFO/LIFO | 成本层 | 按规则消耗成本层并保留层次 | 层数量/金额异常、追溯交易影响层次 |
| Periodic | 期间计算链 | 期末按期间交易和货值计算 | 交易未成本、期间范围不完整、处理顺序错误 |

成本方法在库存组织层面确定。组织发生交易后通常不能直接切换方法；实际项目如需改变，应评估新组织和数据迁移方案，并以目标实例和 Oracle 支持意见为准。本课程不设计成本方法切换。

### 2. 三个 Cost 概念解决三个不同问题

| 概念 | 它回答的问题 | 例子 | 常见误区 |
| --- | --- | --- | --- |
| Cost Type | “这是哪一套成本版本？” | Frozen、Pending、SIM-AUG | 把它当成成本要素或会计期间 |
| Cost Element | “这笔成本的经济性质是什么？” | Material、Resource、Overhead | 把物料 A、某台机器当作要素本身 |
| Cost Group | “同一库存组织内按哪个估价分区管理？” | 默认成本组、项目/WMS 场景成本组 | 把它当成 Frozen/Pending 版本 |

本课程案例使用一个默认成本组，不展开项目制造或 WMS。功能顾问仍须知道：Cost Group 是估价分区维度，不等于 Cost Type。

### 3. 标准成本中的成本类型

- **Frozen**：当前生效、供标准成本事务计价使用的成本。
- **Pending**：准备下一次标准成本更新的待生效成本。
- **自定义模拟成本类型**：用于情景测算和方案比较，如 `SIM-AUG`。模拟成本不会因为被计算出来就自动进入 Frozen。

推荐的纸面控制链为：自定义模拟 → 评审 → 准备 Pending → Cost Rollup → 对比 Pending/Frozen → 审批 → Standard Cost Update。第 4 周将完整学习该生命周期。

实际采购价也不会自动改写 Frozen。例如 A 的标准成本为 10、采购价格为 11；在标准成本法下，应保留标准并通过采购价格等差异解释偏差，除非企业经过正式标准成本更新决定把新标准改为 11。

### 4. 五大成本要素

| Cost Element | 含义 | 典型来源 |
| --- | --- | --- |
| Material | 直接材料或采购物料价值 | 买入物料成本、BOM 组件成本 |
| Material Overhead | 与物料相关的附加成本 | 按物料、收货或其他基准吸收的物料附加费 |
| Resource | 工艺路线资源成本 | Routing Resource Usage × Resource Rate |
| Outside Processing（OSP） | 外协工序成本 | 外协资源、采购与接收相关处理 |
| Overhead | 制造间接费 | Department/Resource 关联、Basis Type、Rate、Activity |

“五大要素”是成本汇总分类；具体物料子要素、资源、间接费和活动是更细的成本来源。一个成品的 Resource 成本可同时包含本层资源和从下层半成品卷积上来的资源。

### 5. 纸面配置的先后关系

```text
组织与 Ledger 上下文
  → 库存组织成本方法及估价账户
  → Cost Type
  → Activity / Resource / Overhead / Department 关系
  → Buy Item Cost 与 Make Item BOM/Routing
  → Rollup / Review / Update
  → 采购、库存、WIP、会计与月结测试
```

先明确方法和组织边界，再设计成本明细。不能在尚未决定成本方法时直接从某个物料单价开始“配成本”。

## 贯穿案例任务

### 选型结论

为 MFG_CN 选择 **Standard Costing**，理由如下：

- 企业采用重复、稳定的离散制造结构，需要用标准与实际差异支持责任分析。
- 原料、资源和间接费均可建立批准后的标准。
- 本学习案例需要演练 Pending/Frozen、成本卷积、WIP 差异和标准成本更新。

Average 仅作为备选方法说明，不进入案例配置；FIFO/LIFO、Periodic 不进入后续案例。

### 基线成本版本

| 物料 | Buy/Make | Frozen 标准成本 | 本周成本要素说明 |
| --- | --- | ---: | --- |
| A | Buy | 10 | Material |
| B | Buy | 4 | Material |
| S | Make | 20 | Material 12 + Resource 6 + Overhead 2 |
| F | Make | 52 | 第 3 周展开本层/下层明细 |

本周把这些数值视为经批准的案例基线，不模拟系统录入。另建 `SIM-AUG` 成本类型，留作第 4 周把 A 从 10 模拟调整到 11。

### 配置工作簿模板

至少建立以下五个页签或五张表：

1. **组织与方法**：Ledger、OU、库存组织、成本方法、选择理由、禁止直接切换的控制说明。
2. **成本类型**：Frozen、Pending、SIM-AUG 的用途、维护责任人、可更新性、输入和输出。
3. **成本要素来源**：五大要素、案例是否使用、来源对象、责任部门。
4. **基础对象**：A/B/S/F 的 Buy/Make、UOM、成本来源；资源、部门、间接费和活动的纸面编码。
5. **控制与测试**：审批、冻结窗口、Rollup 检查、更新前后对账、必要测试场景。

## 作业

### 作业 1：成本方法对比与选型说明

提交一页对比表，至少比较：

- 计价基础；
- 哪些交易会改变单位成本；
- 差异如何呈现；
- 负库存/交易顺序风险；
- 是否适合本案例。

表后用不超过 200 字说明为何选择 Standard，以及选择它后管理层必须承担哪些标准维护和差异分析责任。

### 作业 2：基础配置工作簿

按上述五个页签完成纸面工作簿。每个对象必须有“业务含义、案例值、责任人、依赖、验证方式”，不要求菜单路径或数据库字段。

### 作业 3：三概念辨析卡

分别用一句定义、一个案例值和一个反例说明 Cost Type、Cost Element、Cost Group。要求其他学习者随机抽一张卡后，能在 30 秒内判断它属于哪一类。

## 自测（9 题）

1. Standard 与 Average 的单位成本在日常交易中如何不同？
2. 为什么已有交易的库存组织不能把成本方法切换当作普通参数修改？
3. Frozen 和 Pending 分别承担什么角色？
4. 自定义模拟成本计算完成后，是否会自动替代 Frozen？
5. Cost Type 与 Cost Element 的区别是什么？
6. Cost Group 为什么不等于 Cost Type？
7. 列出五大成本要素。
8. A 的标准成本为 10、采购价为 11，采购完成后 Frozen 是否应自动变为 11？
9. Resource 与 Overhead 各自通常从哪些基础对象取得成本？

### 参考答案

1. Standard 日常按 Frozen 标准计价并记录差异；Average 会因符合条件的入库、完工或调整重新形成平均成本。
2. 成本方法决定历史交易、估价和会计逻辑，直接切换会破坏一致性，通常需新组织/迁移方案和正式评估。
3. Frozen 是当前生效标准；Pending 是下一次更新的待生效标准。
4. 不会；仍需评审、准备 Pending 和受控的 Standard Cost Update。
5. Cost Type 是一套成本版本；Cost Element 是成本的经济性质分类。
6. Cost Group 用于估价分区，而 Cost Type 用于区分成本版本。
7. Material、Material Overhead、Resource、Outside Processing、Overhead。
8. 不应自动改变；差额先按标准成本业务的差异逻辑处理，是否改标准须走正式更新。
9. Resource 通常来自 Routing 的资源用量和费率；Overhead 来自部门/资源关联、Basis Type、Rate 和 Activity。

## 验收标准

- [ ] 总投入在 6–8 小时内，完成阅读、案例、作业和自测。
- [ ] 对比表覆盖 Standard、Average、FIFO/LIFO、Periodic，且没有把 Periodic 说成“查询当前 Item Cost”。
- [ ] 能在 30 秒内准确区分 Cost Type、Cost Element、Cost Group。
- [ ] 能完整列出五大成本要素，并为案例中的 Material、Resource、Overhead 指出来源。
- [ ] 工作簿包含五类页面及责任人、依赖和验证方式，不虚构菜单或系统执行结果。
- [ ] 选型说明明确采用 Standard，并说明标准维护、差异分析和更新控制责任。
- [ ] 自测至少答对 8 题；所有错题已订正。
