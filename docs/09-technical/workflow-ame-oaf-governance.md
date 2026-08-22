# Workflow、AME、OAF/Forms 与配置迁移治理

## 分工

- Oracle Workflow 负责业务流程、通知、活动、Business Event 与后台引擎处理。
- AME 负责规则化审批人清单和条件；业务单据仍由各产品的审批集成点驱动。
- OAF/Forms Personalization 用于受支持的界面行为调整；复杂定制需要遵守 R12.2 EBR/ADOP、扩展点、安全和回归要求。

## 实施与发布顺序

1. 用业务流程图定义事件、状态、审批角色、超时、代理、拒绝/撤回和异常补偿。
2. 将 AME 条件、属性、规则、动作类型和测试样例纳入版本控制；覆盖金额、组织、币种、项目、税务等边界值。
3. 区分 Personalization 与代码扩展：优先使用页面/Forms Personalization；代码仅使用受支持扩展点。
4. 使用 FNDLOAD、WFLOAD、OAF/Forms 发布工件或 Oracle 受支持工具迁移，并在 ADOP 流程中完成多环境回归。

## 诊断 SQL

```sql
-- Workflow 项目状态按业务键收缩；敏感内容和通知正文不应随意导出。
select wi.item_type,
       wi.item_key,
       wi.root_activity,
       wi.begin_date,
       wi.end_date
  from wf_items wi
 where wi.item_type = :p_item_type
   and wi.item_key = :p_item_key;

-- 并发/Workflow 诊断前先确认请求和业务单据的关联，避免仅按用户名全库检索。
select wias.item_type,
       wias.item_key,
       wias.process_activity,
       wias.activity_status,
       wias.begin_date,
       wias.end_date
  from wf_item_activity_statuses wias
 where wias.item_type = :p_item_type
   and wias.item_key = :p_item_key
 order by wias.begin_date;
```

## 常见问题

- 审批人不正确：先检查交易属性和 AME 条件输入，再检查职责/人员/组织层级；不要直接修改已运行 Workflow 状态表。
- 通知未发：分辨 Workflow 状态、Background Engine、Mailer、邮件通道和外部 SMTP 断点。
- 页面修改在补丁后消失：检查是否使用受支持 Personalization/扩展及其迁移工件，避免运行文件系统临时改动。

## 官方参考

- [Oracle E-Business Suite Technology Documentation](https://docs.oracle.com/cd/E26401_01/nav/technology.htm)
- [Oracle E-Business Suite Maintenance Guide](https://docs.oracle.com/cd/E26401_01/doc.122/e22954/toc.htm)
