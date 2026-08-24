# 导航、学习与知识库治理

本目录是知识库的推荐起点，负责建立产品全景、角色学习路径、版本适用性、术语和生产安全边界。内容不替代 Oracle Support、许可证合同、客户变更流程、会计政策或当地法规意见。

## 新读者从这里开始

1. [顾问学习手册](consultant-handbook.md)：用一份文档建立财务功能与技术全景。
2. [按角色阅读与练习路径](reading-paths-by-role.md)：选择功能、技术、集成、测试或运维路线。
3. [财务产品地图与边界](financials-product-map.md)：分清 GL、SLA、AP、AR、FA、CE、IBY、EBTax、Projects 和 Costing 的职责。
4. [中英文术语与缩略语](../90-reference/glossary-and-acronyms.md)：查询英文全称、中文名称和使用语境。
5. 再进入 [知识图谱与模块详文](../README.md) 完成专项学习。

## 指南清单

| 文档 | 用途 |
| --- | --- |
| [顾问学习手册](consultant-handbook.md) | 企业结构、端到端流程、会计追溯、技术架构、接口、发布、测试和排障综合教程 |
| [范围、版本与适用性](scope-and-version.md) | 记录版本基线，判断产品、补丁、许可证和实例差异 |
| [财务产品地图与边界](financials-product-map.md) | 按 R2R、P2P、C2C、A2R、Cash/Tax、Costing 划分产品职责 |
| [按角色阅读路径](reading-paths-by-role.md) | 财务功能、财务技术、集成、测试、经理和运维的学习路线 |
| [按生命周期阅读路径](reading-paths-by-lifecycle.md) | Assessment、Blueprint、Build、Test、Cutover、Hypercare、Run |
| [文档、SQL 与示例规范](documentation-conventions.md) | 内容结构、术语、SQL 安全、验证和维护要求 |
| [生产安全与支持边界](safety-and-production-boundaries.md) | 只读诊断、标准入口、变更审批、回退和敏感数据控制 |
| [Oracle 官方资料与验证顺序](official-sources.md) | 从官方概念到目标实例和非生产验证的证据链 |

## 三条必须遵守的原则

1. 所有结论先标明适用 EBS/AD-TXK/数据库/产品补丁、组织、账簿和场景；经验不能自动推广到所有 R12.2.x 实例。
2. 写入使用标准页面、公开 API、Open Interface、标准并发程序或批准的 Oracle Support 方案；不直接 DML EBS 业务基表。
3. “接口/请求完成”不等于业务闭环；必须验证业务状态、数量/金额、SLA、GL、报表和外部回执。

## 推荐学习闭环

```text
概念与产品边界 → 业务流程 → 配置 → 交易与反向交易
  → SLA/GL → 报表/对账/关账 → 数据模型/接口 → 排错
  → 实施案例与官方依据 → 非生产验证
```

每个专题至少形成：流程图、配置清单、会计矩阵、状态/主键链、测试用例、对账公式、排错证据和官方来源。
