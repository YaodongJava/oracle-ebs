# 职责、数据访问权限与安全配置

## 安全模型

```text
User → Responsibility → Menu → Function
                    ├→ Request Group → Concurrent Program
                    ├→ Profile Options → OU / Ledger / BG
                    └→ Data Security / Grants / Product-specific rules
```

- Responsibility 定义功能边界；Menu/Function Exclusion 决定可打开功能。
- Request Group 决定可提交的并发程序，它与页面功能权限分开。
- MOAC 控制 OU；GL Data Access Set 控制 Ledger/平衡段；HR Security Profile 控制 HR 数据。
- Profile 按 User > Responsibility > Application > Site 的优先级取最终值，用户层覆盖是常见漏权原因。
- R12.2 客刷定制不应直接修改标准权限对象，应通过自定义 Menu/Responsibility/Grant 扩展。

## 配置原则

1. 按岗位而非个人设计职责，实施最小权限和职责分离（SoD）。
2. 将交易录入、审批、付款、过账、设置和用户管理分离。
3. 尽量在 Responsibility 层设置组织和账簿 Profile，严格控制 User 层特例。
4. 为共享服务职责配置 MOAC，并对报表导出、批量更新和跨 OU 处理单独授权。
5. 使用失效日期和定期复核，不共享 EBS 账号。

## 常用 SQL

```sql
-- 用户的职责
SELECT fu.user_name, frv.responsibility_name,
       furg.start_date, furg.end_date,
       fa.application_short_name
  FROM fnd_user fu
  JOIN fnd_user_resp_groups_direct furg ON furg.user_id = fu.user_id
  JOIN fnd_responsibility_vl frv
    ON frv.responsibility_id = furg.responsibility_id
   AND frv.application_id = furg.responsibility_application_id
  JOIN fnd_application fa ON fa.application_id = frv.application_id
 WHERE fu.user_name = UPPER(:p_user_name)
 ORDER BY frv.responsibility_name;

-- 职责的菜单和请求组
SELECT frv.responsibility_name, frv.menu_id, fm.menu_name,
       frv.request_group_id, frv.data_group_id,
       frv.start_date, frv.end_date
  FROM fnd_responsibility_vl frv
  LEFT JOIN fnd_menus fm ON fm.menu_id = frv.menu_id
 WHERE frv.responsibility_id = :p_resp_id
   AND frv.application_id = :p_resp_appl_id;

-- 当前会话
SELECT fnd_global.user_id, fnd_global.resp_id,
       fnd_global.resp_appl_id,
       fnd_profile.value('ORG_ID') org_id,
       fnd_profile.value('GL_ACCESS_SET_ID') access_set_id
  FROM dual;
```

## 排查

- **菜单不可见**：查用户/职责有效期、Menu 层级、Function Exclusion、缓存和登录时使用的职责。
- **不能提交程序**：查 Request Group 及 Program/Application 分配，不要只看 Menu。
- **能查不能改**：查 Function Parameters、Data Access Set 读写权限、单据状态和个性化。
- **多看到了数据**：先查 User 层 Profile、MO Security Profile/GL Access Set，再查定制 SQL 是否绕过组织过滤。
- **授权后不生效**：检查 Security List Maintenance（如适用）、Workflow Directory Services 同步、缓存与重新登录。

## 关联文档

- [多组织 MOAC](organization.md)
- [并发程序与日志](../09-technical/concurrent-programs.md)
- [生产运维与审计](../09-technical/operations.md)
