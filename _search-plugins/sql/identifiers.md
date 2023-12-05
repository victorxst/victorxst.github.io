---
layout: default
title: 身份标识
parent: SQL和PPL
nav_order: 6
redirect_from:
  - /search-plugins/ppl/identifiers/
---


# 身份标识

标识符是命名您的数据库对象的ID，例如索引名称，字段名称，别名等。
OpenSearch支持两种类型的标识符：常规标识符和划界标识符。

## 常规标识符

常规标识符是一串字符，以ASCII字母（下部或高层）开头。
下一个字符可以是字母，数字或下划线（_）。它不能是保留的关键字。
也不允许使用Whitespace和其他特殊字符。

OpenSearch支持以下常规标识符：

1. 标识符由点前缀`.` 符号。用于隐藏索引。例如`.opensearch-dashboards`。
2. 标识符由`@` 符号。用于LogStash摄入生成的元字段。
3. 连字符的标识符`-` 在中间。用于日期信息的索引名称。
4. 标识符带有星星`*` 展示。用于索引模式的通配符匹配。

对于常规标识符，您可以使用该名称，而无需任何背tick或逃脱字符。
在此示例中`source`，，，，`fields`，，，，`account_number`，，，，`firstname`， 和`lastname` 都是标识符。在这些中，`source` 字段是保留标识符。

```sql
SELECT account_number, firstname, lastname FROM accounts;
```

| 帐号| 名| 姓|
：--- | ：--- |
| 1| 琥珀色| 公爵
| 6| 哈蒂| 纽带
| 13| Nanette| 贝茨
| 18| 戴尔| 亚当斯


## 界定标识符

界定标识符可以包含常规标识符不允许的特殊字符。
您必须用背tick封闭定界标识符（\`\`）。Back Ticks将标识符与特殊角色区分开。

如果索引名称包括一个点（`.`）， 例如，`log-2021.01.11`，使用带有背滴的划界标识符逃脱\``日志-2021.01.11`\`。

使用划界标识符的典型示例：

1. 具有保留关键字的标识符。
2. 标识符`.` 展示。相似地，`-` 包括日期信息。
3. 具有其他特殊字符的标识符。例如，Unicode字符。

引用带有tick的索引名称：

```sql
source=`accounts` | fields `account_number`;
```

| 帐号|
：--- |
| 1|       
| 6|
| 13|
| 18|

## 案例灵敏度

标识符对病例敏感。它们必须与OpenSearch中存储的内容完全相同。

例如，如果您运行`source=Accounts`，因为实际索引名称在较低的情况下，您将获得一个索引。

