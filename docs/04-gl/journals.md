# GL 日记账来源、类别、审批与自动过账

## 原理与控制

Journal Source 标识产生系统，控制 Import、Freeze Source、Journal References 等；Category 标识业务性质。Batch 是审批/过账单位，Header 定义 Ledger/Period/Currency/Source/Category，Line 保存账户和借贷。

- Journal Approval 通常依据 Ledger、Source/Category、Amount 和职权配置 Workflow/AME。
- AutoPost Criteria Set 按 Ledger、Source、Category、Balance Type、Period 筛选批次。
- Reversal 可按 Category 默认 Method/Period；Switch Dr/Cr 与 Change Sign 的结果不同。
- Source Freeze 可防止在 GL 修改来自子账的日记账，保持审计链。

## SQL

```sql
SELECT gjh.je_header_id, gjh.je_batch_id, gjh.name,
       gjh.je_source, gjh.je_category, gjh.status,
       gjh.period_name, gjh.currency_code,
       gjh.running_total_dr, gjh.running_total_cr,
       gjh.running_total_accounted_dr,
       gjh.running_total_accounted_cr,
       gjh.reversed_je_header_id, gjh.accrual_rev_period_name
  FROM gl_je_headers gjh
 WHERE gjh.je_header_id = :p_je_header_id;

SELECT gjb.je_batch_id, gjb.name, gjb.status,
       gjb.approval_status_code, gjb.posted_date,
       gjb.posting_run_id, gjb.request_id
  FROM gl_je_batches gjb
 WHERE gjb.je_batch_id = :p_je_batch_id;

SELECT je_source_name, user_je_source_name,
       journal_approval_flag, override_edits_flag
  FROM gl_je_sources
 ORDER BY user_je_source_name;
```

## 排查

- 审批人找不到：查员工/用户关联、职位/审批限额、Workflow/AME 规则和通知状态。
- AutoPost 没选中：比较 Criteria Set 与 Batch 的 Ledger/Source/Category/Balance Type/Period/Status。
- Reversal 不正确：检查 Reversal Method、Period、Effective Date、原日记账状态和是否已反冲。
- 子账 Journal 被修改：检查 Source Freeze、职责和审计数据；应在子账反冲并重建会计。

## 关联

- [GL 主流程](process.md)
- [Web ADI/Import](reporting-interfaces.md)
