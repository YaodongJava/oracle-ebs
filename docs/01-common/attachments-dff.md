# 附件、说明性弹性域与文档序列

## 附件

EBS 附件由实体定义、文档、附件关联和存储内容组成。常用对象为 `FND_ATTACHED_DOCUMENTS`、`FND_DOCUMENTS`、`FND_DOCUMENTS_TL`、`FND_DOCUMENT_DATATYPES`、`FND_LOBS`。附件可以是文件、URL、短文本或长文本，是否可见受实体、类别、功能和用户权限影响。

## DFF 原理

Descriptive Flexfield（DFF）用于在不修改标准表结构的情况下，使用预留 `ATTRIBUTE_CATEGORY`、`ATTRIBUTE1...N` 字段扩展业务属性。Global Segment 对所有上下文显示，Context-sensitive Segment 由上下文值决定。DFF 只是存储与校验机制，不应替代独立业务实体或高频报表模型。

## 文档序列

Document Sequence 为业务文档生成唯一、可审计的编号。通常流程为：定义 Sequence → Document Category → 按 Ledger/Application/Method/Date 分配。`Sequential Numbering` Profile 决定强制、部分或不使用。已启用并产生编号的序列不应随意修改初始值或删除分配。

## 配置检查

- DFF：确认 Title/Application/Table、上下文、Value Set、Required/Security、引用字段和编译状态。
- 附件：确认 Entity/Primary Key 映射、Category 分配、文件大小限制、MIME 类型、存储和病毒扫描策略。
- 序列：确认应用、单据类别、账簿/方法、有效日期和 Profile 值。
- R12.2 在线补丁环境中，数据库定制必须遵循 Edition-Based Redefinition 和 AD Online Patching 开发标准。

## 常用 SQL

```sql
-- 某业务实体附件元数据（PK1_VALUE 根据实体可能为字符串）
SELECT fad.attached_document_id, fad.entity_name,
       fad.pk1_value, fad.pk2_value, fad.seq_num,
       fd.document_id, fd.datatype_id, fd.category_id,
       fdt.title, fdt.description, fd.url, fd.media_id
  FROM fnd_attached_documents fad
  JOIN fnd_documents fd ON fd.document_id = fad.document_id
  LEFT JOIN fnd_documents_tl fdt
    ON fdt.document_id = fd.document_id
   AND fdt.language = USERENV('LANG')
 WHERE fad.entity_name = :p_entity_name
   AND fad.pk1_value = TO_CHAR(:p_pk1_value)
 ORDER BY fad.seq_num;

-- DFF 定义
SELECT application_table_name, descriptive_flexfield_name,
       title, freeze_flex_definition_flag
  FROM fnd_descriptive_flexs_vl
 WHERE UPPER(title) LIKE UPPER(:p_title_pattern);

-- 文档序列
SELECT name, application_id, type, initial_value,
       start_date, end_date, message_flag
  FROM fnd_document_sequences
 WHERE application_id = :p_application_id
 ORDER BY name;
```

## 排查

- **DFF 不显示**：检查功能对应 DFF Title、上下文值、段启用/有效期、编译和缓存。
- **DFF 值保存失败**：检查 Value Set 长度/类型、Required、安全规则和底层 `ATTRIBUTE` 列长度。
- **附件看不到**：比较 Entity Name、PK1...PK5、Category 和功能分配；不要只查 `FND_LOBS`。
- **文件无法下载**：检查 `MEDIA_ID`、LOB 存在性、MIME、Web 层日志和反向代理大小/超时限制。
- **序列不生成/重复**：查 Sequential Numbering Profile、Category Assignment、有效期和方法；不直接改序列表。

## 关联文档

- [R12.2 定制开发](../09-technical/customization.md)
- [权限与审计](security.md)
