# Record to Report：子账到总账的关账编排

## 关账不是单一模块动作

月结需要把业务截止、接口处理、子账会计、GL 导入/过账、重估/折算/合并、报表、对账和签字作为有依赖的受控流程。各组织、Ledger、币种和产品可有不同日历；应以关账日历和责任矩阵明确顺序。

## 推荐依赖图

```text
业务交易截止与接口冻结
  → AP / AR / FA / Projects / Cost / CE 等子账处理完成
  → Create Accounting（Final）与 SLA 例外清零
  → Transfer to GL → Journal Import → Post
  → Revaluation / Translation / Consolidation（如适用）
  → 子账-GL 对账、银行/库存/资产/项目对账
  → 管理/法定报表 → 业务与财务签字 → 关闭期间
```

## 控制矩阵

| 控制 | 责任方 | 合格证据 | 失败处理 |
| --- | --- | --- | --- |
| 业务截止 | 业务模块负责人 | 截止时间、未处理业务清单、接口批次状态 | 延后、隔离或按政策进入下一期间 |
| SLA 完整性 | 财务系统负责人 | 未会计事件、错误、Draft/Final、传输状态 | 纠正规则/交易后受控重跑 |
| GL 完整性 | 总账负责人 | Import/Posting 例外、未过账批次、余额报表 | 修复来源或批准手工调整并保留追溯 |
| 对账 | 模块控制人 | AP/AR/FA/CST/CE/PA 与 GL 差异说明 | 按主键、事件、分录、期间逐层定位 |
| 签字与锁期 | 财务负责人 | 关账包、例外批准、报表版本 | 禁止绕过流程直接开关已签字期间 |

## 跨模块诊断 SQL

```sql
-- GL 未过账批次应按 Ledger/期间审查，避免扫描全部历史数据。
select gjh.je_header_id,
       gjh.name,
       gjh.period_name,
       gjh.status,
       gjh.posted_date,
       gjb.name batch_name
  from gl_je_headers gjh
  join gl_je_batches gjb
    on gjb.je_batch_id = gjh.je_batch_id
 where gjh.ledger_id = :p_ledger_id
   and gjh.period_name = :p_period_name
   and nvl(gjh.status, 'U') <> 'P';
```

## 排错原则

先确认差异属于业务、会计、传输、导入、过账、汇率/折算还是报告口径；再按交易主键、`EVENT_ID`、`AE_HEADER_ID`、`GL_SL_LINK_ID` 和 `JE_HEADER_ID` 分层定位。禁止用总账手工分录长期遮蔽子账问题。

## 官方参考

- [Oracle Financials Concepts Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48836/toc.htm)
- [Oracle Financials Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48783/toc.htm)
