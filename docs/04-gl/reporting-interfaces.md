# FSG、Smart View、Web ADI 与日记账导入

## 报表与接口

- **FSG**：Row Set 定义账户/计算行，Column Set 定义期间/金额/计算列，Content Set 按段拆分，Row Order 定义排序，Display Set 控制显示。
- **Smart View**：通过已配置数据源在 Excel 查询/钻取 GL 余额，权限仍受 EBS/GL 数据访问控制。
- **Web ADI**：Integrator + Interface + Content + Layout + Mapping 将 Excel 数据验证并上传，GL Journal 最终进入 GL Interface/Import。
- **Journal Import**：按 Source/Group ID 从 `GL_INTERFACE` 生成 Batch/Header/Lines，错误行留在接口表并带 Status。

## SQL

```sql
SELECT status, ledger_id, user_je_source_name,
       user_je_category_name, accounting_date,
       currency_code, code_combination_id,
       entered_dr, entered_cr, accounted_dr, accounted_cr,
       group_id, reference1, reference4
  FROM gl_interface
 WHERE group_id = :p_group_id
 ORDER BY accounting_date;

SELECT gir.je_header_id, gir.je_line_num,
       gir.reference_1, gir.reference_2, gir.reference_3,
       gir.reference_4, gir.gl_sl_link_id, gir.gl_sl_link_table
  FROM gl_import_references gir
 WHERE gir.je_header_id = :p_je_header_id
 ORDER BY gir.je_line_num;

SELECT row_set_id, name, description
  FROM rg_report_axis_sets
 ORDER BY name;
```

## 排查

- FSG 金额不对：检查 Ledger/Currency/Amount Type、Period Offset、Row Account Assignment、Summary/Detail、Sign 和报表显示舍入。
- FSG 空白：查 Data Access Set、行列显示条件、Zero Suppression、Content Set 和账户范围。
- Web ADI 上传失败：查 Desktop Integration 配置、Excel 信任/宏、Integrator/Layout/Mapping、职责、日期格式和服务器日志。
- Journal Import 错：按 `STATUS` 解码，检查 Ledger/Period/CCID/Currency/Balance/Source；修正上游或接口，不直接改已生成 Journal。
- 子账 Drilldown 丢失：检查 Journal Source Import References 选项和 `GL_IMPORT_REFERENCES`/SLA Link。

## 关联

- [GL Process](process.md)
- [COA](../01-common/coa.md)
- [Integration](../09-technical/integration.md)

## 官方参考

- [Oracle General Ledger Implementation Guide R12.2](https://docs.oracle.com/cd/E26401_01/doc.122/e48747/toc.htm)
