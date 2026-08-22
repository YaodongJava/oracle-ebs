# Oracle EBS R12.2.x 知识图谱

## 使用方法

本目录以“企业结构与共享设置 → 子账交易 → SLA 会计 → GL 过账与报表 → 期间结账”为主线，同时以 P2P、O2C、库存/制造成本和项目资产化为端到端视角。

```text
Organization / COA / Calendar / Security
                 │
Supplier → PO → Receipt → AP → Payment ─┐
                                            ├→ SLA → GL → Reporting / Close
Customer → OM → Shipping → AR → Receipt ─┘
                 │
          Inventory ↔ WIP ↔ Cost
                 │
              Projects → Assets
```

## 阅读路径

- 实施人员：先读 `01-common`，再读业务模块的 `process.md` 和专题文档。
- 财务顾问：沿子账业务、SLA、GL、对账和结账链路阅读。
- 技术顾问：结合 `09-technical/data-model.md`、接口文档和各章的核心表/SQL。
- 运维人员：优先使用每章“排查顺序”，再执行 SQL；不直接 DML EBS 基表。

## 模块数据字典

| 模块 | 数据字典 | 覆盖内容 |
| --- | --- | --- |
| 财务公共 | [01-common/tables.md](01-common/tables.md) | Organization、Ledger、COA、Period、FND、SLA |
| AP | [02-ap/tables.md](02-ap/tables.md) | Supplier、Invoice、Distribution、Hold、Payment、Interface |
| AR/TCA | [03-ar/tables.md](03-ar/tables.md) | Customer、Transaction、Schedule、Receipt、Application、AutoInvoice |
| GL | [04-gl/tables.md](04-gl/tables.md) | Batch、Journal、Interface、Reference、Balance |
| FA | [05-fa/tables.md](05-fa/tables.md) | Asset、Book、Distribution、Depreciation、Retirement、Mass Additions |
| INV/CST/WIP | [06-cost/tables.md](06-cost/tables.md) | Item、On-hand、Material Transaction、Cost、WIP、Interface |
| CE/IBY/EBTax | [07-ce-tax/tables.md](07-ce-tax/tables.md) | Bank Account、Statement、Payment Instruction、Tax Line/Rate/Registration |
| E2E | [08-e2e/tables.md](08-e2e/tables.md) | P2P、O2C、Projects-Assets 的跨模块 ID 链 |
| FND/Workflow | [09-technical/tables.md](09-technical/tables.md) | User/Responsibility/Profile、Concurrent、Workflow |

## 模块接口实现手册

| 模块 | 实现手册 | 具体代码与业界案例 |
| --- | --- | --- |
| 财务公共 | [01-common/interfaces.md](01-common/interfaces.md) | 统一暂存表、FND/MOAC、提交/等待并发请求、Business Event |
| AP | [02-ap/interfaces.md](02-ap/interfaces.md) | 发票头行导入、PO/Receipt 匹配、Invoice Import、错误与成功对账 |
| AR | [03-ar/interfaces.md](03-ar/interfaces.md) | AutoInvoice、收入分配、AutoLockbox、ISG REST 调用 |
| GL | [04-gl/interfaces.md](04-gl/interfaces.md) | `GL_INTERFACE`、批次平衡、Journal Import、Journal 追溯 |
| FA | [05-fa/interfaces.md](05-fa/interfaces.md) | `FA_MASS_ADDITIONS`、遗留迁移、Prepare/Post 与资产对账 |
| INV/CST/WIP | [06-cost/interfaces.md](06-cost/interfaces.md) | MTI、Lot Interface、Transaction Manager、成本状态追踪 |
| CE/IBY/EBTax | [07-ce-tax/interfaces.md](07-ce-tax/interfaces.md) | 银行对账单接口、付款/ACK、Reconciliation Open Interface、税分类 |
| E2E | [08-e2e/interfaces.md](08-e2e/interfaces.md) | P2P/O2C/项目资产化、相关号、Transactional Outbox、补偿 |
| 技术集成 | [09-technical/interfaces.md](09-technical/interfaces.md) | Concurrent Worker、API 模板、ISG REST、退避重试、可观测性 |

所有写入示例仅使用 Oracle 标准 Open Interface/公开 API 或客户自定义对象，不直接修改 EBS 业务基表。上线前必须按目标 R12.2.x 实例的 Integration Repository、并发程序参数、eTRM 和 `ALL_TAB_COLUMNS` 复核签名与字段。

## SQL 约定

- SQL 默认为 APPS 视角的只读诊断样例，`:p_*` 为绑定变量。
- 大表查询必须增加 `ORG_ID`、`LEDGER_ID`、业务日期或主键范围。
- 先在测试库验证列名和执行计划；对象以当前 R12.2 补丁级别的 eTRM/`ALL_TAB_COLUMNS` 为准。
- 数据修复应使用标准页面、公开 API 或 Oracle Support 方案，不将本库 SQL 改为 UPDATE/DELETE 直接执行。

## 官方资料

- [Oracle E-Business Suite Release 12.2 Documentation Library](https://docs.oracle.com/cd/E26401_01/index.htm)
- [Oracle Financials Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48783/T348488T348491.htm)
- [Oracle Subledger Accounting Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48771/title.htm)
