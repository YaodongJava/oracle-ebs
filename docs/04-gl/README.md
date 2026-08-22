# Oracle General Ledger（GL / Record to Report）

GL 是法人/账簿层面的最终会计记录与报告层。本目录覆盖日记账、预算控制、重估/折算/合并、报表、Journal Import 及期间关闭；SLA 规则的权威正文见 `01-common/sla.md`。

## 专题导航

- [账簿、日记账与过账流程](process.md)
- [日记账来源、类别、审批与自动过账](journals.md)
- [预算与资金控制](budgetary-control.md)
- [合并、重估、折算与重复日记账](consolidation-revaluation.md)
- [月结、年结与报表](close-reports.md)
- [FSG、Smart View、Web ADI 与导入](reporting-interfaces.md)
- [SLA、FAH 与 AGIS](sla-fah-agis.md)
- [表结构](tables.md)
- [`GL_INTERFACE` 实现](interfaces.md)

## 设计与关账原则

1. 先锁定 COA、日历、币种、会计方法和 Ledger，再配置 Ledger Set、Data Access Set、二级账簿及 Reporting Currency。
2. 任何子账余额差异先在子账/SLA 排除，确认已生成、传输和导入后再检查 GL；不要以手工总账分录掩盖子账差异。
3. 月结采用“子账关闭 → SLA/GL 传输 → Journal Import/Posting → 重估/折算/合并 → 报表与签字”的受控顺序。
4. 预算控制、自动平衡、悬挂账户、公司间与 Journal Approval 的例外须进入持续监控和审批流程。

## 官方依据

- [Oracle General Ledger Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
- [Oracle Financials Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48783/toc.htm)
