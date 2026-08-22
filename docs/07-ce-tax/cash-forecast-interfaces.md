# 付款/收款清算、现金预测与银行接口

> 银行对账单两张接口表、IBY 付款/回执和 EBTax 传参代码见 [CE/IBY/EBTax 接口实现案例](interfaces.md)。

## 现金预测

Cash Forecast Template 定义来源、时间桶、币种和包含条件。来源可包括 AP Invoices/Payments、AR Transactions/Receipts、PO/Requisitions、Payroll、Cashflows 和手工预测。Forecast 是预期流动性，不是 GL 实际现金余额。

## 银行接口设计

- 入站：Bank Statement/Lockbox/Acknowledgement，保存原始文件、Checksum、Bank/Account、Sequence、批次和重复键。
- 出站：Payment File/Positive Pay/Remittance，使用 IBY Format/PPP/Transmission，实施加密、签名、安全传输和回执。
- 不将银行账号、个人信息或私钥写入并发日志；设置文件保留/脱敏/归档策略。

## SQL

```sql
-- 未对账银行流水汇总
SELECT csh.bank_account_id, csl.trx_type, csl.status,
       COUNT(*) line_count, SUM(csl.amount) amount
  FROM ce_statement_headers csh
  JOIN ce_statement_lines csl
    ON csl.statement_header_id = csh.statement_header_id
 WHERE csh.statement_date BETWEEN :p_start_date AND :p_end_date
 GROUP BY csh.bank_account_id, csl.trx_type, csl.status;

-- 请求参数和日志线索
SELECT request_id, phase_code, status_code,
       actual_start_date, actual_completion_date,
       argument_text, logfile_name, outfile_name
  FROM fnd_concurrent_requests
 WHERE request_id = :p_request_id;
```

## 排查

- Forecast 为空：检查 Template Source、Cutoff Date/Bucket、OU/Ledger/Bank Account、Currency 和源单据状态。
- Forecast 重复：检查多来源是否同时包含 Invoice+Payment/Order+Invoice，理解预测选择规则。
- Bank File 重复：使用 Bank Account + Statement/File ID + Sequence + Checksum 幂等，不仅依赖文件名。
- 传输失败：查证书/密钥有效期、SFTP/HTTPS 连通、目录权限、文件编码、银行回执和 IBY/OPP 日志。

## 关联

- [Cash Management](cash-management.md)
- [Integration](../09-technical/integration.md)
