# Oracle EBS R12.2.x 企业结构与多组织（Multi-Org / MOAC）

## 1. 文档目标与适用范围

本文面向 Oracle E-Business Suite R12.2.x，用于设计、配置和排查以下企业结构：

- Business Group（业务组）
- Ledger（账簿，R11i 中称 Set of Books）
- Legal Entity（法人实体）
- Operating Unit（业务实体，简称 OU）
- Inventory Organization（库存组织，简称 IO）
- MOAC（Multiple Organizations Access Control，多组织访问控制）

> **SQL 安全说明**：本文 SQL 默认为只读诊断 SQL，建议使用 APPS 只读账号执行。不要在生产库直接更新 `HR_*`、`XLE_*`、`GL_*`、`FND_*` 或业务表；配置变更应通过标准 Form/OAF 页面、公开 API 或经 Oracle Support 确认的数据修复方案完成。

---

## 2. 核心概念与层级

```text
EBS Instance
└── Business Group（HR 人员与组织数据边界）
    ├── Primary Ledger（COA + 币种 + 会计日历 + SLA 方法）
    │   ├── Legal Entity A（法定/税务报告主体）
    │   │   ├── Operating Unit A1（AP/AR/PO/OM 交易数据边界）
    │   │   │   ├── Inventory Organization A11
    │   │   │   └── Inventory Organization A12
    │   │   └── Operating Unit A2
    │   └── Legal Entity B
    └── Other Ledger / Legal Entity / OU ...
```

这是逻辑关系，不表示所有层级都由同一张父子表存储。EBS 会分别在 HR、GL、XLE 和 INV 数据模型中保存组织定义及关系。

### 2.1 各层级的作用

| 层级 | 业务含义 | 主要数据边界 | 常用字段/ID |
| --- | --- | --- | --- |
| Business Group | HR 上的最高层人员与组织分区，决定立法代码、HR 弹性域等 | 人员、岗位、组织、HR 安全 | `BUSINESS_GROUP_ID` |
| Ledger | 共享科目表、本位币、会计日历和 SLA 方法的财务报告主体 | GL 日记账和余额；由 Data Access Set 控制 | `LEDGER_ID` |
| Legal Entity | 依法设立、承担税务和法定报告义务的实体 | 法人登记、税号、法定地址、法人与账簿关系 | `LEGAL_ENTITY_ID` |
| Operating Unit | 使用 AP、AR、PO、OM、CE 等子模块的业务单元 | 子账模块交易，通常以 `ORG_ID` 隔离 | `ORG_ID` / `ORGANIZATION_ID` |
| Inventory Organization | 记录物料、库存、WIP、BOM 和制造交易的组织 | 库存与制造数据 | `ORGANIZATION_ID` |

### 2.2 容易混淆的边界

1. **OU 不等于法人**：OU 是子账业务和数据权限边界，Legal Entity 是法定主体。OU 通过 Primary Ledger 和 Default Legal Context 与法人关联。
2. **OU 权限不等于 Ledger 权限**：MOAC 控制 OU；GL Data Access Set 控制 Ledger/平衡段值。一个职责能查到 AP 发票，不代表它一定有权查看相应 GL 账簿数据。
3. **OU 不等于 Inventory Organization**：OU 下可有多个 IO；IO 另有自己的组织代码、库存参数和物料主组织关系。
4. **`ORG_ID` 的含义取决于表**：在 AP/AR/PO/OM 中通常表示 OU；在 INV 表中更常见的是 `ORGANIZATION_ID`，通常表示 IO。不能只看字段名猜测。
5. **HR Organization 是通用容器**：同一个 `HR_ALL_ORGANIZATION_UNITS` 组织可被赋予一个或多个分类；具体属性保存在组织信息表或专用模块表中。

---

## 3. R12.2 MOAC 工作原理

### 3.1 从职责到数据的链路

```text
User
  → Responsibility
    → HR: Business Group
    → MO: Security Profile（多 OU）
       或 MO: Operating Unit（单 OU，兼容方式）
    → MO: Default Operating Unit（可选，仅用于默认值）
      → 会话初始化 MOAC 上下文
        → 子账页面/报表/API 只能处理授权 OU
```

- `MO: Security Profile` 可授权一个或多个 OU，是 R12 共享服务模式的核心。
- `MO: Operating Unit` 只提供单 OU 访问。
- 如果已设置 `MO: Security Profile`，`MO: Operating Unit` 会被忽略。
- `MO: Default Operating Unit` 只决定录入时的默认 OU，**不会扩大授权范围**。
- 标准 Security Profile 限于同一 Business Group；跨 Business Group 访问应使用 Global Security Profile。

### 3.2 访问模式

MOAC 会话通常有以下访问模式：

| 模式 | 含义 | 常见场景 |
| --- | --- | --- |
| `S` | Single，当前只有一个 OU | 单 OU 职责或程序显式选定 OU |
| `M` | Multiple，可访问安全配置文件中的多个 OU | MOAC 职责 |
| `A` | All，特殊全局模式 | 仅限经确认的标准程序/管理场景 |
| NULL | 未初始化 | SQL Developer、自定义 JDBC 连接或会话初始化失败 |

R12 标准应用会在登录并选择职责后初始化 FND 和 MOAC 上下文。定制 PL/SQL/API 如果从 SQL*Plus、JDBC 或外部调度器调用，不能假设上下文已存在。

### 3.3 `_ALL`、安全视图与 `ORG_ID`

- 多数 OU 级交易表含 `ORG_ID`，如 `AP_INVOICES_ALL`、`RA_CUSTOMER_TRX_ALL`、`PO_HEADERS_ALL`、`OE_ORDER_HEADERS_ALL`。
- 定制报表必须明确组织边界：使用当前 EBS 会话中的标准安全机制，或在经授权的管理报表中对 `ORG_ID` 显式限定。
- 不要因为对象名含 `_ALL` 就默认当前账号一定能无限制查看全部数据；同义词、VPD 策略、执行 schema 和会话上下文都会影响结果。
- 生产定制程序不应通过硬编码 `ORG_ID`、禁用策略或连接基表来绕过 MOAC。

---

## 4. 规划原则

### 4.1 什么时候需要新 Ledger

下列四项中任一核心属性不同，通常需要新 Ledger：

1. Chart of Accounts（会计科目表）
2. Accounting Calendar（会计日历）
3. Ledger Currency（账簿币种）
4. Subledger Accounting Method（子分类账会计方法）

管理报表、部门分割或地区不同，未必要新建 Ledger，可考虑平衡段、成本中心、OU 或管理维度。

### 4.2 什么时候需要新 Legal Entity

当存在独立法定注册、税号、法定报表、合同主体或法律责任时，应考虑新 Legal Entity。不要仅为了权限隔离而创建虚假法人。

### 4.3 什么时候需要新 OU

当 AP/AR/PO/OM 需要独立的交易类型、单据编号、税务默认、银行/收付款配置、采购或订单管理策略，或需要强 OU 级交易数据隔离时，可考虑新 OU。新建 OU 会增加设置、对账、月结和运维成本，不应将每个部门都设为 OU。

### 4.4 什么时候需要新 Inventory Organization

当需要独立追踪库存数量、库位、物料状态、成本、制造或供应计划时，应设置 IO。纯交易型 OU 不一定需要自己的 IO；但 PO Receiving、INV、WIP 等功能通常需要库存组织。

---

## 5. 标准配置顺序

> 菜单名称会因语言、产品安装和自定义职责而有差异，以当前实例的 Navigator 为准。

### 步骤 1：设计企业结构

在配置前形成经业务、财务、税务、HR 和信息安全共同确认的映射表：

| Business Group | Ledger | Legal Entity | Balancing Segment | OU | Inventory Org | 负责人 |
| --- | --- | --- | --- | --- | --- | --- |
| ... | ... | ... | ... | ... | ... | ... |

同时确认组织编码、名称、启用日期、法定地址、税号、账簿四要素及历史数据迁移边界。

### 步骤 2：定义 Ledger 和 Accounting Setup

常用路径：

```text
General Ledger 超级用户
  → Setup
  → Financials
  → Accounting Setup Manager
  → Accounting Setups
```

配置 Primary Ledger，并根据需要关联 Legal Entity、Balancing Segment Value、Reporting Currency、Secondary Ledger、SLA 选项、公司间/公司内平衡规则等。

### 步骤 3：定义 Legal Entity

通过 Legal Entity Configurator 或 Accounting Setup Manager 定义法人、法定地址、注册地、税务登记和相关联系。确保法人与 Primary Ledger 及平衡段值的设计一致。

### 步骤 4：定义 Location

常用路径：

```text
HRMS / Inventory 相关职责
  → Work Structures
  → Location
```

组织尤其是库存组织通常需要有效 Location。核对地址、国家/地区、启用日期以及 Location 是否已被其他 IO 占用。

### 步骤 5：定义 OU

可在 Define Organization 窗口定义，也可通过 Accounting Setup Manager 建立。OU 至少需要：

- Organization Classification = Operating Unit
- Primary Ledger
- Default Legal Context
- 正确的 Business Group 和有效日期

若在 Define Organization 中建立，仍应在 Accounting Setup Manager 中检查其账簿/法人关系。

### 步骤 6：定义 Inventory Organization

先建立组织与 Inventory Organization 分类，设置 Accounting Information：

- Primary Ledger
- Legal Entity
- Operating Unit

再完成 Inventory Parameters、Organization Code、Master Organization、账户、日历、子库和库位等 INV 设置。不要通过复制表数据建立 IO。

### 步骤 7：完成各模块 OU 级设置

常见设置包括：

- AP：Financial Options、Payables Options、发票选项、付款、银行账户用途。
- AR：System Options、Transaction Types、Transaction Sources、Receivables Activities、AutoAccounting。
- PO：Financial Options、Purchasing Options、Receiving Options、Document Types。
- OM：OM System Parameters、Transaction Types、Shipping Parameters。
- CE/IBY/EBTax：银行账户使用权、付款处理配置、税务法规和法人/OU 适用范围。

### 步骤 8：定义 Security Profile / Global Security Profile

常用路径：

```text
Human Resources
  → Security
  → Profile
```

选择直接列出组织或使用组织层级。遵循最小权限，将交易处理职责和跨 OU 查询/共享服务职责分开。

### 步骤 9：运行 Security List Maintenance

创建或修改 Security Profile/组织层级后，提交：

```text
Security List Maintenance（程序短名常见为 PERSELM）
```

等待请求正常完成并检查日志。遗漏该步骤是“配置文件已加 OU，但用户仍看不到”的常见原因。

### 步骤 10：设置 Profile Options

使用 System Administrator 职责，建议优先在 Responsibility 层设置：

| Profile | 用途 | 建议 |
| --- | --- | --- |
| `HR: Business Group` | 职责所属 Business Group | 与普通 Security Profile 中的 BG 一致 |
| `MO: Security Profile` | 授权的一个或多个 OU | R12.2 多 OU 首选 |
| `MO: Default Operating Unit` | 新建交易时默认 OU | 必须属于上述授权集 |
| `MO: Operating Unit` | 单 OU 授权 | 未设 `MO: Security Profile` 时使用 |
| `GL: Data Access Set` | GL 账簿/平衡段访问 | 与 OU 权限联合测试 |

设置后要求用户退出并重新登录，至少重新进入职责，以创建新会话上下文。

### 步骤 11：执行端到端验证

1. 每个 OU 分别建立测试交易。
2. 验证单 OU 和多 OU 职责的查询、新增、修改和报表范围。
3. 验证默认 OU 与 LOV 范围。
4. 测试并发请求参数和输出是否跨 OU。
5. 跟踪 SLA 会计、GL 转入、平衡段值和法人归属。
6. 验证未授权用户不能通过 Form、OAF、报表、Web ADI 或定制接口绕过权限。

---

## 6. 常用只读 SQL

### 6.1 查询 OU、Ledger 和默认 Legal Entity

```sql
SELECT hou.organization_id              AS org_id,
       hou.name                         AS operating_unit,
       hou.short_code,
       hou.business_group_id,
       hou.set_of_books_id              AS ledger_id,
       gl.name                           AS ledger_name,
       gl.currency_code,
       gl.chart_of_accounts_id,
       hou.default_legal_context_id      AS legal_entity_id,
       xep.name                          AS legal_entity_name,
       xep.legal_entity_identifier,
       hou.date_from,
       hou.date_to
  FROM hr_operating_units hou
  LEFT JOIN gl_ledgers gl
    ON gl.ledger_id = hou.set_of_books_id
  LEFT JOIN xle_entity_profiles xep
    ON xep.legal_entity_id = hou.default_legal_context_id
 ORDER BY gl.name, hou.name;
```

> `SET_OF_BOOKS_ID` 是兼容历史命名，在 R12 的 OU 视图中实际对应 Primary Ledger ID。不要把它误解为 R11i 独立账簿对象。

### 6.2 查询 Ledger 四要素

```sql
SELECT gl.ledger_id,
       gl.name                  AS ledger_name,
       gl.short_name,
       gl.ledger_category_code,
       gl.currency_code,
       gl.chart_of_accounts_id,
       gl.period_set_name,
       gl.accounted_period_type,
       gl.sla_accounting_method_code,
       gl.sla_accounting_method_type,
       gl.complete_flag
  FROM gl_ledgers gl
 WHERE gl.ledger_category_code = 'PRIMARY'
 ORDER BY gl.name;
```

### 6.3 查询 Legal Entity

```sql
SELECT xep.legal_entity_id,
       xep.name AS legal_entity_name,
       xep.legal_entity_identifier,
       xep.transacting_entity_flag,
       xep.effective_from,
       xep.effective_to
  FROM xle_entity_profiles xep
 ORDER BY xep.name;
```

### 6.4 查询 Inventory Organization 及所属 OU

```sql
SELECT ood.organization_id,
       ood.organization_code,
       ood.organization_name,
       ood.operating_unit              AS org_id,
       hou.name                        AS operating_unit,
       ood.set_of_books_id             AS ledger_id,
       gl.name                         AS ledger_name,
       ood.legal_entity                AS legal_entity_id,
       xep.name                        AS legal_entity_name,
       ood.master_organization_id,
       ood.disable_date
  FROM org_organization_definitions ood
  LEFT JOIN hr_operating_units hou
    ON hou.organization_id = ood.operating_unit
  LEFT JOIN gl_ledgers gl
    ON gl.ledger_id = ood.set_of_books_id
  LEFT JOIN xle_entity_profiles xep
    ON xep.legal_entity_id = ood.legal_entity
 ORDER BY hou.name, ood.organization_code;
```

### 6.5 检查组织分类及有效期

```sql
SELECT haou.organization_id,
       haou.name,
       haou.business_group_id,
       haou.date_from,
       haou.date_to,
       hoi.org_information1 AS classification_code,
       hoi.org_information2 AS enabled_flag
  FROM hr_all_organization_units haou
  JOIN hr_organization_information hoi
    ON hoi.organization_id = haou.organization_id
   AND hoi.org_information_context = 'CLASS'
 ORDER BY haou.name, hoi.org_information1;
```

> 不同补丁级别下 HR 分类视图的可用列可能不同。如果上述 SQL 提示列不存在，请先用 `ALL_TAB_COLUMNS` 检查当前实例定义，不要盲目修改数据。

### 6.6 查询 Profile Option 在各层级的显式设置

```sql
SELECT fpo.user_profile_option_name,
       fpov.level_id,
       CASE fpov.level_id
         WHEN 10001 THEN 'SITE'
         WHEN 10002 THEN 'APPLICATION'
         WHEN 10003 THEN 'RESPONSIBILITY'
         WHEN 10004 THEN 'USER'
         ELSE TO_CHAR(fpov.level_id)
       END AS level_name,
       fpov.level_value,
       CASE fpov.level_id
         WHEN 10002 THEN fa.application_name
         WHEN 10003 THEN frv.responsibility_name
         WHEN 10004 THEN fu.user_name
       END AS level_value_name,
       fpov.profile_option_value
  FROM fnd_profile_options_vl fpo
  JOIN fnd_profile_option_values fpov
    ON fpov.application_id = fpo.application_id
   AND fpov.profile_option_id = fpo.profile_option_id
  LEFT JOIN fnd_application_vl fa
    ON fpov.level_id = 10002
   AND fa.application_id = fpov.level_value
  LEFT JOIN fnd_responsibility_vl frv
    ON fpov.level_id = 10003
   AND frv.responsibility_id = fpov.level_value
   AND frv.application_id = fpov.level_value_application_id
  LEFT JOIN fnd_user fu
    ON fpov.level_id = 10004
   AND fu.user_id = fpov.level_value
 WHERE fpo.profile_option_name IN
       ('ORG_ID', 'DEFAULT_ORG_ID', 'XLA_MO_SECURITY_PROFILE_LEVEL',
        'PER_BUSINESS_GROUP_ID', 'GL_ACCESS_SET_ID')
 ORDER BY fpo.user_profile_option_name,
          fpov.level_id,
          level_value_name;
```

> 页面显示名与内部 `PROFILE_OPTION_NAME` 不同。上述内部名称在标准 R12 中常见，实际环境应先通过下面 SQL 确认：

```sql
SELECT profile_option_name,
       user_profile_option_name
  FROM fnd_profile_options_vl
 WHERE UPPER(user_profile_option_name) IN
       ('MO: OPERATING UNIT',
        'MO: DEFAULT OPERATING UNIT',
        'MO: SECURITY PROFILE',
        'HR: BUSINESS GROUP',
        'GL: DATA ACCESS SET')
 ORDER BY user_profile_option_name;
```

### 6.7 查看当前 EBS 会话上下文

```sql
SELECT fnd_global.user_id                       AS user_id,
       fnd_global.user_name                     AS user_name,
       fnd_global.resp_id                       AS responsibility_id,
       fnd_global.resp_appl_id                  AS responsibility_application_id,
       fnd_profile.value('PER_BUSINESS_GROUP_ID') AS business_group_id,
       fnd_profile.value('ORG_ID')              AS mo_operating_unit,
       fnd_profile.value('DEFAULT_ORG_ID')      AS default_operating_unit,
       fnd_profile.value('XLA_MO_SECURITY_PROFILE_LEVEL') AS security_profile_id,
       mo_global.get_access_mode                AS mo_access_mode,
       mo_global.get_current_org_id             AS current_org_id
  FROM dual;
```

`FND_GLOBAL.RESP_ID = -1` 或关键 Profile 为 NULL，往往说明当前不是一个已正确初始化的 EBS 会话。

### 6.8 查看当前会话可访问 OU

```sql
SELECT organization_id,
       name
  FROM mo_glob_org_access_tmp
 ORDER BY name;
```

`MO_GLOB_ORG_ACCESS_TMP` 是会话级临时数据。在另一个 SQL 会话查询，不会看到 EBS Forms/OAF 会话的内容；未先初始化 MOAC 时返回空集也是正常的。

### 6.9 从 OU 汇总主要子账交易量

```sql
SELECT hou.organization_id AS org_id,
       hou.name AS operating_unit,
       (SELECT COUNT(*)
          FROM ap_invoices_all aia
         WHERE aia.org_id = hou.organization_id) AS ap_invoice_count,
       (SELECT COUNT(*)
          FROM ra_customer_trx_all rcta
         WHERE rcta.org_id = hou.organization_id) AS ar_trx_count,
       (SELECT COUNT(*)
          FROM po_headers_all pha
         WHERE pha.org_id = hou.organization_id) AS po_count,
       (SELECT COUNT(*)
          FROM oe_order_headers_all ooha
         WHERE ooha.org_id = hou.organization_id) AS order_count
  FROM hr_operating_units hou
 ORDER BY hou.name;
```

> 该 SQL 会对大表执行多次计数，仅适合在测试库或经 DBA 确认后使用。生产库建议增加日期/单据范围，并检查执行计划。

### 6.10 找出无效 `ORG_ID` 的交易（数据质量）

```sql
SELECT 'AP_INVOICES_ALL' AS table_name,
       aia.org_id,
       COUNT(*) AS row_count
  FROM ap_invoices_all aia
 WHERE NOT EXISTS
       (SELECT 1
          FROM hr_operating_units hou
         WHERE hou.organization_id = aia.org_id)
 GROUP BY aia.org_id
UNION ALL
SELECT 'RA_CUSTOMER_TRX_ALL',
       rcta.org_id,
       COUNT(*)
  FROM ra_customer_trx_all rcta
 WHERE NOT EXISTS
       (SELECT 1
          FROM hr_operating_units hou
         WHERE hou.organization_id = rcta.org_id)
 GROUP BY rcta.org_id;
```

如果结果不为空，先检查是否为历史升级数据、失效 OU、定制导入或查询权限造成的假阳性，再向 DBA/Oracle Support 提供样例数据。

### 6.11 查询并发请求的 OU 参数线索

```sql
SELECT r.request_id,
       r.phase_code,
       r.status_code,
       r.actual_start_date,
       r.actual_completion_date,
       r.responsibility_id,
       r.responsibility_application_id,
       r.argument_text
  FROM fnd_concurrent_requests r
 WHERE r.request_id = :p_request_id;
```

`ARGUMENT_TEXT` 仅是参数快照，不能通过位置盲猜 OU。应在程序定义的 Parameters 窗口确认参数顺序和 Value Set，并联合请求日志分析。

### 6.12 检查当前表/视图列定义

```sql
SELECT owner,
       table_name,
       column_id,
       column_name,
       data_type,
       data_length
  FROM all_tab_columns
 WHERE table_name IN
       ('HR_OPERATING_UNITS',
        'ORG_ORGANIZATION_DEFINITIONS',
        'XLE_ENTITY_PROFILES',
        'GL_LEDGERS')
 ORDER BY table_name, column_id;
```

这是跨 R12.2 补丁级别或客户定制环境排查 `ORA-00904: invalid identifier` 时的第一步。

---

## 7. 定制程序的上下文初始化

### 7.1 诊断会话示例

以下仅用于开发/测试环境复现 EBS 会话，不应将用户、职责和 OU ID 硬编码到生产接口：

```sql
DECLARE
  l_user_id      NUMBER := :p_user_id;
  l_resp_id      NUMBER := :p_resp_id;
  l_resp_appl_id NUMBER := :p_resp_appl_id;
  l_org_id       NUMBER := :p_org_id;
BEGIN
  -- 初始化 EBS 用户和职责上下文
  fnd_global.apps_initialize(
    user_id      => l_user_id,
    resp_id      => l_resp_id,
    resp_appl_id => l_resp_appl_id
  );

  -- 参数为应用短名；AP 常见为 SQLAP
  mo_global.init('SQLAP');

  -- 诊断时选定单 OU。先确认该 OU 属于职责授权范围。
  mo_global.set_policy_context('S', l_org_id);
END;
/
```

注意：

- 应用短名必须与实际模块一致，不要对所有模块固定使用 `SQLAP`。
- `FND_GLOBAL.APPS_INITIALIZE` 只应使用真实、有权且未失效的 User/Responsibility/Application ID。
- 先初始化 FND 上下文，再初始化 MOAC。
- 并发程序由 EBS Concurrent Manager 初始化用户和职责上下文；仍应按开发指南处理 OU 参数和 MOAC，不要随意覆盖会话权限。

### 7.2 定制 SQL/PLSQL 的安全原则

1. 交易接口要求显式传入 OU，并验证它在当前职责授权范围内。
2. 所有接口表、错误表和日志表应保存 `ORG_ID`，避免后续无法对账。
3. 不将 OU 名称作为唯一键；使用 ID，并在输出中同时展示名称。
4. 不仅依赖前端 LOV 过滤；数据库 API 层也要验证权限。
5. 不在连接池中泄漏上一个请求的 EBS/MOAC 会话状态；每次借出连接时显式初始化，归还时清理。
6. 对跨 OU 报表记录操作用户、职责、参数、请求 ID 和导出时间，便于审计。

---

## 8. 常见问题与排查

### 8.1 责任中看不到 OU

**现象**：OU LOV 为空、进入页面报无可访问组织，或只看到部分 OU。

**按顺序检查**：

1. 用户分配的职责是否有效，开始/结束日期是否正确。
2. `MO: Security Profile` 的最终值是否在 User 层被意外覆盖。
3. Security Profile 是否包含目标 OU，组织层级版本和日期是否有效。
4. 普通 Security Profile 的 Business Group 是否与 `HR: Business Group` 一致；跨 BG 是否应使用 Global Security Profile。
5. 修改后是否成功运行 Security List Maintenance。
6. 用户是否已退出重登，旧 Forms/OAF 会话是否仍在使用旧上下文。
7. OU 和组织分类是否在当前日期有效。

### 8.2 只能看到一个 OU

**常见原因**：

- 只设置了 `MO: Operating Unit`，没有设置 `MO: Security Profile`。
- Security Profile 本身只包含一个 OU。
- 程序或页面在会话中将 policy context 设成了 Single。
- 特定产品功能本身不支持跨 OU 处理，或报表参数将范围限制到一个 OU。

先用第 6.7、6.8 节 SQL 判断是职责配置问题，还是特定程序行为。

### 8.3 默认 OU 不正确或不生效

1. 查看 User 层是否覆盖 `MO: Default Operating Unit`。
2. 默认 OU 必须属于 `MO: Security Profile` 的 OU 集合。
3. 某些页面会优先使用业务单据、客户/供应商地点、用户首选项或程序参数的 OU，并非所有字段都直接使用 Profile 默认值。
4. 重新登录后再测试。

### 8.4 新建 OU 后 AP/AR/PO/OM 无法录入交易

分两层排查：

**组织层**

- OU 分类是否启用。
- Primary Ledger 和 Default Legal Context 是否正确。
- 账簿、OU、法人和平衡段值的关系是否符合 Accounting Setup。
- 当前职责是否获得 OU 权限。

**模块层**

- AP/PO Financial Options 和模块 Options 是否完整。
- AR System Options、Transaction Source/Type、AutoAccounting 是否已按 OU 设置。
- OM System Parameters、Shipping Parameters 是否完整。
- EBTax、IBY、银行账户用途和文档序列是否已分配。
- 会计期间和相关子模块期间是否已打开。

### 8.5 并发报表页面有数据，输出为空

1. 检查请求由哪个职责提交，不要只看用户。
2. 检查程序的 OU/Reporting Level 参数和 Value Set。
3. 检查请求日志中的 `ORG_ID`、Ledger ID、日期范围和数据权限提示。
4. 确认定制报表是否正确初始化 MOAC，是否错把 `MO: Default Operating Unit` 当作唯一授权 OU。
5. 如为 BI Publisher，同时检查 Data Template 参数映射、Before Data Trigger 和执行 schema。

### 8.6 SQL 查询结果与应用页面不一致

通常由以下原因造成：

- SQL 会话没有 FND/MOAC 上下文。
- SQL 使用的 schema/同义词与标准应用不同。
- 页面还应用了状态、有效日期、数据访问集或产品级安全规则。
- SQL 遗漏 `ORG_ID`、`LEDGER_ID`、日期有效性或语言条件。
- 页面使用物化结果、摘要表或已缓存的会话数据。

诊断时记录同一用户、职责、时间点、OU、参数和具体单据 ID，再做对比。

### 8.7 用户能看数据，但不能创建或更新

MOAC 只是数据访问的一层。继续检查：

- Responsibility 的 Menu/Function Exclusion。
- Form/OAF 的个性化和只读设置。
- 单据状态、期间状态、审批状态和业务规则。
- Ledger/Data Access Set 是否仅有 Read Only 权限。
- 子模块职责是否只授予查询功能。
- 自定义代码是否对用户、职责或 OU 进行了额外限制。

### 8.8 法人、OU 或 Ledger 关联错误

不要直接更新 `HR_OPERATING_UNITS` 或组织信息表。先：

1. 停止在错误 OU 继续建立交易。
2. 导出组织、Accounting Setup、法人、平衡段值和已有交易数量证据。
3. 判断是新建错误且无交易，还是已产生会计/税务/付款数据。
4. 在测试环境验证标准页面是否允许更正。
5. 已有生产交易时，建议建立 Oracle Support SR，按数据修复方案处理。

### 8.9 组织已失效但仍出现，或提前消失

联合检查：

- `HR_ALL_ORGANIZATION_UNITS.DATE_FROM/DATE_TO`
- 组织分类的启用状态
- 组织层级版本的有效期
- Security Profile 的有效日期规则
- EBS 会话日期与数据库日期
- 修改后是否运行 Security List Maintenance 并重新登录

### 8.10 性能问题：跨 OU 查询很慢

1. 确认查询对 `ORG_ID`、业务日期和主键有选择性条件。
2. 避免对大表先跨组织联接、最后才过滤 OU。
3. 使用绑定变量，不拼接 OU 列表。
4. 检查统计信息、执行计划和数据倾斜，不在未评估时盲目加 Hint/索引。
5. 检查定制视图是否重复调用 MOAC 策略函数，是否存在隐式数据类型转换。
6. 用 SQL Monitor/AWR/ASH 需遵循数据库许可和 DBA 流程。

---

## 9. 实施检查清单

### 9.1 上线前

- [ ] 企业结构映射已由业务、财务、税务、HR 和安全团队签字。
- [ ] Ledger 四要素、Legal Entity 和 Balancing Segment 映射已确认。
- [ ] OU 的 Primary Ledger 和 Default Legal Context 正确。
- [ ] IO 的 Ledger、Legal Entity、OU、Master Org 和库存参数正确。
- [ ] AP/AR/PO/OM/CE/EBTax/IBY 等 OU 级设置已完成。
- [ ] Security Profile 遵循最小权限，跨 BG 场景已使用 Global Security Profile。
- [ ] Security List Maintenance 已成功完成。
- [ ] Profile Option 的 Site/Application/Responsibility/User 覆盖关系已审查。
- [ ] MOAC 权限与 GL Data Access Set 已做交叉验证。
- [ ] 每个 OU 已完成 P2P、O2C、会计、税务和月结的端到端测试。
- [ ] 定制报表、接口、Web ADI 和 API 已测试未授权 OU 的拒绝访问。
- [ ] 已准备回退方案；回退不依赖直接 DML 修改 EBS 基表。

### 9.2 日常运维

- [ ] 定期审计用户层 Profile 覆盖。
- [ ] 定期审计跨 OU 共享服务职责和导出权限。
- [ ] 组织层级或 Security Profile 变更后运行 Security List Maintenance。
- [ ] 定期检查即将失效的组织、Location 和职责。
- [ ] 保留组织变更申请、测试证据、并发请求 ID 和发布记录。
- [ ] 禁止定制程序通过基表 DML 维护组织和安全数据。

---

## 10. 术语与关键对象速查

| 类别 | 常用对象 | 用途 |
| --- | --- | --- |
| OU | `HR_OPERATING_UNITS` | OU 及 Ledger/默认 Legal Entity 的常用视图 |
| HR 组织 | `HR_ALL_ORGANIZATION_UNITS` | 组织基本信息和有效期 |
| 组织扩展 | `HR_ORGANIZATION_INFORMATION` | 组织分类和扩展属性 |
| Ledger | `GL_LEDGERS` | 账簿四要素和类型 |
| Legal Entity | `XLE_ENTITY_PROFILES` | 法人实体基本信息 |
| Inventory Org | `ORG_ORGANIZATION_DEFINITIONS` | IO 与 OU/Ledger/LE 的便利查询视图 |
| Profile 定义 | `FND_PROFILE_OPTIONS_VL` | Profile 内部名和显示名 |
| Profile 取值 | `FND_PROFILE_OPTION_VALUES` | Site/Application/Responsibility/User 层显式设置 |
| 运行时 Profile | `FND_PROFILE.VALUE` | 获取当前会话最终值 |
| EBS 会话 | `FND_GLOBAL` | 当前 User/Responsibility/Application 上下文 |
| MOAC 会话 | `MO_GLOBAL` | 初始化、访问模式和当前 OU |
| MOAC 临时数据 | `MO_GLOB_ORG_ACCESS_TMP` | 当前会话授权 OU 集合 |

> 视图和列在不同 R12.2 RU/RUP、产品安装组合或定制环境中可能存在差异。以当前实例 `ALL_OBJECTS`、`ALL_TAB_COLUMNS`、eTRM 和 Oracle Support 为准。

---

## 11. 官方参考资料

- [Oracle E-Business Suite Multiple Organizations Implementation Guide, Release 12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48833/T443823T443827.htm)
- [Oracle E-Business Suite Multiple Organizations 架构与术语](https://docs.oracle.com/cd/E26401_01/doc.122/e48833/T443823T443826.htm)
- [Oracle E-Business Suite Concepts: Multiple Organization Architecture](https://docs.oracle.com/cd/E26401_01/doc.122/e22949/T120505T120523.htm)
- [Oracle Financials Concepts Guide: Enterprise Structures](https://docs.oracle.com/cd/E26401_01/doc.122/e48836/T433149T433153.htm)
- [Oracle E-Business Suite Release 12.2 Documentation Library](https://docs.oracle.com/cd/E26401_01/index.htm)

Oracle 文档给出产品原理和标准实施步骤；具体补丁级别的已知问题、数据修复和性能建议，应进一步在 My Oracle Support 中根据准确版本、产品、错误栈和补丁水平检索。
