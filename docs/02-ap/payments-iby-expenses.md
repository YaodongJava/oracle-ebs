# Oracle Payments（IBY）与 Internet Expenses

## 业务定位

AP 负责确认负债和生成待付款项；Oracle Payments 负责付款方式、付款流程请求（PPR）、付款指令、格式、传输和回执。Internet Expenses 将员工费用、公司卡和现金预支按费用政策审批后导入 AP。三者必须以付款业务键、员工/供应商身份和银行对账结果形成闭环。

## 配置主线

1. 定义内部银行账户及用途，并按法人、OU、付款组织和币种限制可用范围。
2. 配置 Payment Method、Payment Process Profile、Payment Document、Format、Transmission Protocol 与外部银行账户。
3. 设置付款审批、Positive Pay、付款文件传输、ACK/拒绝回执和作废/重发的责任分离。
4. iExpenses 配置费用类型、日限额、Per Diem、Mileage、现金预支、公司卡、审批和 AP 导入参数。
5. 使用测试供应商/员工验证“选择发票 → PPR → 文件/回执 → AP 付款 → CE 核对 → XLA/GL”完整路径。

## 状态与控制点

| 阶段 | 需要保留的证据 | 不可直接假设 |
| --- | --- | --- |
| PPR 建议付款 | 发票选择条件、排除原因、审批 | `PAYMENT_STATUS_FLAG` 不等于银行已付款 |
| Payment Instruction | 指令号、格式文件、付款方式、金额/币种控制总额 | 文件生成成功不等于已传输/已受理 |
| 银行回执 | ACK/Reject、银行流水号、拒绝原因 | 银行成功不等于 CE 已自动核对 |
| Employee Expense | 员工、费用行、政策校验、审批、公司卡匹配 | 报销批准不等于 AP 已导入或付款 |

## 只读诊断 SQL

```sql
-- 在目标实例先通过 ALL_TAB_COLUMNS 校验 IBY 对象与列；按付款对象追溯 AP 发票。
select ai.invoice_id,
       ai.invoice_num,
       ai.invoice_amount,
       aps.payment_num,
       aps.amount_remaining,
       aps.payment_status_flag
  from ap_invoices_all ai
  join ap_payment_schedules_all aps
    on aps.invoice_id = ai.invoice_id
 where ai.org_id = :p_org_id
   and ai.invoice_num = :p_invoice_num;

-- 付款完成后再通过 AP 付款与 CE 对账状态分层核实，不以单一状态替代全链结论。
select ac.check_id,
       ac.check_number,
       ac.amount,
       ac.status_lookup_code,
       ac.payment_date,
       ac.ce_bank_acct_use_id
  from ap_checks_all ac
 where ac.org_id = :p_org_id
   and ac.check_number = :p_payment_number;
```

## 常见排查顺序

1. 发票是否已验证、未冻结、到期且付款方式/供应商地点/银行账户均满足选择规则。
2. PPR 是否因付款文档、格式、账户用途、币种、最小/最大金额或审批被排除。
3. 付款指令是否完成格式化与传输，银行回执是否已被受控处理；拒绝项以作废/重发或标准流程处理。
4. 检查 AP 付款、CE 外部交易/对账单行、SLA 和 GL 是否在相同会计期间可追溯。

## 实施边界

支付文件、账号、证书、私钥、卡号和个人报销附件属于敏感数据。示例日志仅保留掩码、哈希、相关号和错误码。付款和报销的写入必须使用页面、标准接口/API 或受支持的并发程序。

## 官方参考

- [Oracle Payments Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
- [Oracle Internet Expenses Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
