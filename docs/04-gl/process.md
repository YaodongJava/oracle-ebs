# General Ledger 账簿、日记账与过账流程

## 架构

```text
Subledger/SLA → GL_INTERFACE → Journal Import
Manual/Web ADI/Recurring → Journal Batch/Header/Lines
→ Approval → Post → GL_BALANCES → Reporting/Close
```

Ledger 由 COA、Currency、Calendar 和 SLA Method 组成。Primary/Secondary Ledger 表示不同会计表述，Reporting Currency 表示币种表述，Ledger Set 用于对多账簿统一开关期和报表。Data Access Set 决定职责对 Ledger/平衡段的读写权限。

## 配置

1. 定义 COA、Calendar、Currency/Rate，在 Accounting Setup Manager 建立 Ledger。
2. 配置 Legal Entity/Balancing Segment、Secondary Ledger/Reporting Currency、SLA、Intercompany/Intracompany。
3. 定义 Journal Source/Category、Suspense/Rounding/Retained Earnings、Document Sequence、Approval/AutoPost。
4. 定义 Data Access Set、Ledger Set、账户安全与 FSG/BI 报表。

## SQL

```sql
SELECT gjb.je_batch_id, gjb.name batch_name, gjb.status batch_status,
       gjh.je_header_id, gjh.name journal_name, gjh.status,
       gjh.ledger_id, gjh.period_name, gjh.je_source,
       gjh.je_category, gjh.currency_code, gjh.actual_flag
  FROM gl_je_batches gjb
  JOIN gl_je_headers gjh ON gjh.je_batch_id = gjb.je_batch_id
 WHERE gjh.je_header_id = :p_je_header_id;

SELECT gjl.je_line_num, gjl.code_combination_id,
       gjl.entered_dr, gjl.entered_cr,
       gjl.accounted_dr, gjl.accounted_cr,
       gjl.description, gjl.status
  FROM gl_je_lines gjl
 WHERE gjl.je_header_id = :p_je_header_id
 ORDER BY gjl.je_line_num;
```

## 排查

- Import 失败：查 `GL_INTERFACE.STATUS/STATUS_DESCRIPTION`、Ledger/Period/Currency/CCID、Source/Group ID。
- Journal 不平：分别检查 Entered/Accounted 借贷、Currency/Rate、Suspense/Rounding 设置和平衡段。
- 不能 Post：检查 Batch/Header Status、Approval、Period、Data Access Set Write 权限、账户有效性。
- 余额不更新：查 Posting 请求日志、Journal Status、Actual Flag、Currency 和查询的 Balance Type。

## 关联

- [GL 常用表结构与字段含义](tables.md)
- [Journal 控制](journals.md)
- [SLA](../01-common/sla.md)
- [GL 结账](close-reports.md)
