---
layout: default
title: 元数据查询
parent: SQL
grand_parent: SQL和PPL
nav_order: 9
redirect_from:
  - /search-plugins/sql/metadata/
---

# 元数据查询

要查看有关您索引的基本元数据，请使用`SHOW` 和`DESCRIBE` 命令。

### 句法

规则`showStatement`：

![展示]({{site.url}}{{site.baseurl}}/images/showStatement.png)

规则`showFilter`：

![展览窗]({{site.url}}{{site.baseurl}}/images/showFilter.png)

### 示例1：有关索引的元数据

要查看与特定模式匹配的索引的元数据，请使用`SHOW` 命令。
使用通配符`%` 匹配所有索引：

```sql
SHOW TABLES LIKE %
```

| table_cat| table_schem| table_name| table_type| 评论| type_cat| type_schem| type_name| self_referencing_col_name| ref_generation
：--- | ：---
Docker-簇| 无效的| 帐户| 基础表| 无效的| 无效的| 无效的| 无效的| 无效的| 无效的
Docker-簇| 无效的| 员工_nest| 基础表| 无效的| 无效的| 无效的| 无效的| 无效的| 无效的


### 示例2：有关特定索引，请参见元数据

要查看带有前缀的索引名称的元数据`acc`：

```sql
SHOW TABLES LIKE acc%
```

| table_cat| table_schem| table_name| table_type| 评论| type_cat| type_schem| type_name| self_referencing_col_name| ref_generation
：--- | ：---
Docker-簇| 无效的| 帐户| 基础表| 无效的| 无效的| 无效的| 无效的| 无效的| 无效的


### 示例3：有关字段的元数据

要查看与特定模式匹配的字段名称的元数据，请使用`DESCRIBE` 命令：

```sql
DESCRIBE TABLES LIKE accounts
```

| table_cat| table_schem| table_name| column_name| 数据类型| type_name| column_size| buffer_length| DECIMAL_DIGITS| num_prec_radix| 无效| 评论| column_def| sql_data_type| SQL_DATETIME_SUB| char_octet_length| ordinal_position| is_nullable| scope_catalog| scope_schema| scope_table| source_data_type| is_autoincrement| is_generatedColumn
：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：--- | ：---
Docker-簇| 无效的| 帐户| 帐号| 无效的| 长的| 无效的| 无效的| 无效的| 10| 2| 无效的| 无效的| 无效的| 无效的| 无效的| 1|  | 无效的| 无效的| 无效的| 无效的| 不|
Docker-簇| 无效的| 帐户| 名| 无效的| 文本| 无效的| 无效的| 无效的| 10| 2| 无效的| 无效的| 无效的| 无效的| 无效的| 2|  | 无效的| 无效的| 无效的| 无效的| 不| 
Docker-簇| 无效的| 帐户| 地址| 无效的| 文本| 无效的| 无效的| 无效的| 10| 2| 无效的| 无效的| 无效的| 无效的| 无效的| 3|  | 无效的| 无效的| 无效的| 无效的| 不| 
Docker-簇| 无效的| 帐户| 平衡| 无效的| 长的| 无效的| 无效的| 无效的| 10| 2| 无效的| 无效的| 无效的| 无效的| 无效的| 4|  | 无效的| 无效的| 无效的| 无效的| 不| 
Docker-簇| 无效的| 帐户| 性别| 无效的| 文本| 无效的| 无效的| 无效的| 10| 2| 无效的| 无效的| 无效的| 无效的| 无效的| 5|  | 无效的| 无效的| 无效的| 无效的| 不| 
Docker-簇| 无效的| 帐户| 城市| 无效的| 文本| 无效的| 无效的| 无效的| 10| 2| 无效的| 无效的| 无效的| 无效的| 无效的| 6|  | 无效的| 无效的| 无效的| 无效的| 不| 
Docker-簇| 无效的| 帐户| 雇主| 无效的| 文本| 无效的| 无效的| 无效的| 10| 2| 无效的| 无效的| 无效的| 无效的| 无效的| 7|  | 无效的| 无效的| 无效的| 无效的| 不| 
Docker-簇| 无效的| 帐户| 状态| 无效的| 文本| 无效的| 无效的| 无效的| 10| 2| 无效的| 无效的| 无效的| 无效的| 无效的| 8|  | 无效的| 无效的| 无效的| 无效的| 不| 
Docker-簇| 无效的| 帐户| 年龄| 无效的| 长的| 无效的| 无效的| 无效的| 10| 2| 无效的| 无效的| 无效的| 无效的| 无效的| 9|  | 无效的| 无效的| 无效的| 无效的| 不| 
Docker-簇| 无效的| 帐户| 电子邮件| 无效的| 文本| 无效的| 无效的| 无效的| 10| 2| 无效的| 无效的| 无效的| 无效的| 无效的| 10|  | 无效的| 无效的| 无效的| 无效的| 不| 
Docker-簇| 无效的| 帐户| 姓| 无效的| 文本| 无效的| 无效的| 无效的| 10| 2| 无效的| 无效的| 无效的| 无效的| 无效的| 11|  | 无效的| 无效的| 无效的| 无效的| 不| 

