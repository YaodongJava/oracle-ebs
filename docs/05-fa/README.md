# Oracle Assets（FA / Acquire to Retire）

本目录覆盖资产账簿、类别、增加、资本化、折旧、调整、转移、处置、Mass Additions、会计与关账。资产来源可来自 AP、Projects/CIP、iAssets 或外部迁移；不同来源需要保留来源单据和资产编号之间的可追溯链。

## 专题导航

- [资产生命周期](process.md)
- [账簿、类别、位置与配置](setup.md)
- [增加、调整、转移、重分类与盘点](asset-transactions.md)
- [折旧、税务折旧、处置与会计](depreciation-accounting.md)
- [月结、报表、Mass Additions 与排错](close-reports-interfaces.md)
- [Projects 到 Assets：CIP 与资本化](projects-capitalization.md)
- [表结构](tables.md)
- [Mass Additions 与迁移接口](interfaces.md)

## 会计与控制重点

| 业务动作 | 需确认的决定因素 | 常见遗漏 |
| --- | --- | --- |
| Capitalize | Asset Book、Category、Date Placed in Service、成本、资产来源 | 将 CIP、费用化和可资本化支出混在同一规则中 |
| Depreciate | Method、Life、Convention、Prorate、Period | 忘记先处理资产交易或未关闭前序模块期间 |
| Transfer/Adjust | Distribution、Location、Expense Account、Cost/Reserve | 只看资产头，遗漏分配行历史和会计影响 |
| Retire | Proceeds、Removal Cost、Partial Units、Gain/Loss | 处置日期/期间不一致，或遗漏 AP/AR/CE 清算链 |

## R12.2 边界

使用 Mass Additions 或 Oracle 公开 API 处理集成与迁移；不直接更新 `FA_ADDITIONS_B`、`FA_BOOKS` 或 `FA_DISTRIBUTION_HISTORY`。资产账簿、税务账簿和折旧规则变更应完成影响分析并留存审批。

## 官方依据

- [Oracle Assets Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
