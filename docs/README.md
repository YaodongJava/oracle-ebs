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

## SQL 约定

- SQL 默认为 APPS 视角的只读诊断样例，`:p_*` 为绑定变量。
- 大表查询必须增加 `ORG_ID`、`LEDGER_ID`、业务日期或主键范围。
- 先在测试库验证列名和执行计划；对象以当前 R12.2 补丁级别的 eTRM/`ALL_TAB_COLUMNS` 为准。
- 数据修复应使用标准页面、公开 API 或 Oracle Support 方案，不将本库 SQL 改为 UPDATE/DELETE 直接执行。

## 官方资料

- [Oracle E-Business Suite Release 12.2 Documentation Library](https://docs.oracle.com/cd/E26401_01/index.htm)
- [Oracle Financials Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48783/T348488T348491.htm)
- [Oracle Subledger Accounting Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48771/title.htm)
