# 第 3 周：BOM、Routing 与成本卷积

## 学习目标

- 区分 BOM 组件、Routing 工序、资源、制造费用和成本要素。
- 手工计算半成品 S、成品 F 的 This Level/Lower Level 成本。
- 解释 Rollup 的 Pending 结果、冻结标准和版本追溯。

## 参考案例

使用 `case-study.md` 的基线：B=4，S=3×B+6 资源+2 制造费用=20；F=2×A+1×S+8 资源+4 制造费用=52。不得把采购价 11 直接替代 A 的 Frozen 标准 10。

## 练习

1. 为 S 画 BOM/Routing 图，并标出 Material、Resource、Overhead。
2. 为 F 展开一层和全部下层，分别列出 This Level 与 Lower Level。
3. 假设 B 的 Pending 成本由 4 调为 4.50，计算 S 与 F 的新 Pending 成本，注明传播路径。
4. 列出替代 BOM、Phantom、损耗率、外协加工和无效日期会影响的输入。

## 6 小时安排

1 小时阅读案例与成本要素；2 小时展开 BOM/Routing；2 小时完成卷积和敏感性分析；1 小时整理证据与问题。

## 自测

- Pending 成本是否会自动成为 Frozen？（不会，需批准并执行标准更新。）
- F 的 Lower Level 成本来自哪里？（来自 S 的已批准成本明细，按卷积层级汇总。）
- 没有实例时能否填写真实 Rollup 请求号？（不能，标记为待实机验证。）

## 交付与验收

- 一张成本卷积表：物料、用量、成本要素、金额、层级。
- 一张“输入变更→受影响装配件→需复核报表”的影响矩阵。
- 验收：金额可复算；明确 Pending 不等于 Frozen；没有编造系统请求号。

## 官方依据

- [Cost Management — Overview of Standard Costing](https://docs.oracle.com/cd/E26401_01/doc.122/e48829/T372621T373688.htm)
- [Cost Management User's Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48829/toc.htm)
