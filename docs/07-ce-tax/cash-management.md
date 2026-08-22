# 现金管理：银行、银行账户、对账单与自动核对

## 模型与流程

R12 银行/分行基于 TCA，内部银行账户由 CE 管理，账户所有者/用途决定 AP/AR/Payroll/Treasury 在哪个 Legal Entity/OU 下可用。

```text
Bank Statement Import/Manual Entry
→ Header/Lines → AutoReconciliation Matching
→ Reconciled/Unreconciled/Errors
→ Cash Position + Accounting/Close
```

AutoReconciliation 根据 Transaction Code Mapping、Reference、Amount/Date Tolerance、Receipt/Payment Number 等规则匹配 AP Payment、AR Receipt、Cashflow、Bank Transfer 和手工现金交易。

## SQL

```sql
SELECT cba.bank_account_id, cba.bank_account_name,
       cba.bank_account_num, cba.currency_code,
       cba.start_date, cba.end_date,
       cba.account_owner_org_id
  FROM ce_bank_accounts cba
 WHERE cba.bank_account_id = :p_bank_account_id;

SELECT csh.statement_header_id, csh.bank_account_id,
       csh.statement_number, csh.statement_date,
       csh.currency_code, csh.control_begin_balance,
       csh.control_end_balance
  FROM ce_statement_headers csh
 WHERE csh.bank_account_id = :p_bank_account_id
 ORDER BY csh.statement_date DESC;

SELECT csl.statement_line_id, csl.line_number,
       csl.trx_date, csl.trx_type, csl.trx_code,
       csl.amount, csl.status, csl.bank_trx_number,
       csl.invoice_text
  FROM ce_statement_lines csl
 WHERE csl.statement_header_id = :p_statement_header_id
 ORDER BY csl.line_number;
```

## 排查

- 银行账户不可选：查 Owner Legal Entity、OU Use、Application Use、Currency、有效期和用户权限。
- Statement Import 错：检查 Bank Account/Number、Currency、Statement Number 唯一性、Control Balance、Transaction Code 和文件格式。
- AutoReconciliation 匹配不到：比较 Transaction Type/Code、Reference、Amount/Date/Currency、原交易状态和容差。
- 对账后 GL 不对：跟踪 AP/AR/CE 原交易、Clearing Event、SLA 和 GL Post，区分对账状态与会计状态。

## 关联

- [CE/IBY/EBTax 常用表结构与字段含义](tables.md)
- [AP Payments](../02-ap/payments.md)
- [AR Receipts](../03-ar/receipts.md)
