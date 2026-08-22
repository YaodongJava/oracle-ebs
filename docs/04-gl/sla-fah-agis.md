# SLA、Financials Accounting Hub 与 AGIS

## 适用范围

SLA 是 EBS 子账会计的通用引擎；GL 接收其传输的日记账。Financials Accounting Hub（FAH）用于将外部业务系统的事件转换为受控的子账会计；Advanced Global Intercompany System（AGIS）处理跨法人/跨组织内部交易。两者均为独立可选产品/功能边界，须确认许可证、安装和实施范围。

## 会计链路

```text
业务交易
  → Transaction Entity / Event / Event Type
  → Accounting Method Builder（AAD/JLD/JLT/ADR/Mapping Set）
  → XLA AE Header / Line
  → Transfer to GL
  → Journal Import / GL Journal / Post
```

## 设计要点

| 主题 | 实施决定 | 控制要求 |
| --- | --- | --- |
| SLA | 会计方法、事件类型、账户规则、辅助参考、说明规则 | 不直接改已完成历史会计；规则改动须版本化、测试和审批 |
| FAH | 外部来源、事件模型、接口字段、映射、异常/重放 | 外部业务键必须唯一，可从来源交易追溯至 GL |
| AGIS | 交易类型、组织关系、内部交易账户、审批与接收规则 | 发出/接收、AP/AR、公司间与消除差异分别对账 |
| Balancing | Intercompany/Intracompany 规则、舍入、悬挂账户 | 配置变化先在隔离 Ledger/测试数据验证分录平衡 |

## 只读诊断 SQL

```sql
-- 从会计事件追踪已生成的子账分录；按事件或实体键收缩范围。
select xte.entity_code,
       xte.source_id_int_1,
       xte.transaction_number,
       xah.event_id,
       xah.ae_header_id,
       xah.accounting_entry_status_code,
       xah.gl_transfer_status_code
  from xla_transaction_entities xte
  join xla_events xev
    on xev.entity_id = xte.entity_id
  join xla_ae_headers xah
    on xah.event_id = xev.event_id
 where xte.ledger_id = :p_ledger_id
   and xte.transaction_number = :p_transaction_number;

-- 分录行到 GL 的关联应通过受支持的 XLA/GL 链审查，字段以目标 eTRM 为准。
select xal.ae_header_id,
       xal.ae_line_num,
       xal.accounting_class_code,
       xal.accounted_dr,
       xal.accounted_cr,
       xal.gl_sl_link_id
  from xla_ae_lines xal
 where xal.ae_header_id = :p_ae_header_id
 order by xal.ae_line_num;
```

## 排错顺序

1. 确认源交易、会计事件及其状态，再检查会计定义是否对该事件类型生效。
2. 区分“未创建会计”“Draft/Final 状态”“未传输 GL”“Journal Import/过账失败”四个断点。
3. 对 FAH/AGIS 先检查来源业务键、批次控制和接收方状态；不要把跨系统部分成功当作单一数据库事务回滚。

## 官方参考

- [Oracle Subledger Accounting Implementation Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e48771/title.htm)
- [Oracle Financials Accounting Hub Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
- [Oracle Advanced Global Intercompany System Documentation](https://docs.oracle.com/cd/E26401_01/nav/financials.htm)
