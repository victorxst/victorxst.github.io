---
layout: default
title: CAT API
nav_order: 10
has_children: true
redirect_from:
  - /opensearch/catapis/
  - /opensearch/rest-api/cat/index/
---

# 猫API
**引入1.0**
{: .label .label-purple }
您可以轻松获得有关群集的基本统计信息-到-了解使用紧凑和对齐文本（CAT）API的表格格式。猫API是人类-可读的界面返回纯文本而不是传统的JSON。

使用CAT API，您可以回答问题，例如哪个节点是当选的大师，群集中的哪个状态，每个索引中有多少个文档等等。

## 例子

要查看CAT API中的可用操作，请使用以下命令：

```
GET _cat
```
{% include copy-curl.html %}

## 可选查询参数

您可以使用任何CAT API使用以下查询参数来过滤您的结果。

范围| 描述
:--- | :--- |
`v` |  通过将标头添加到列中来提供详细的输出。它还添加了一些格式，以帮助将每个列对齐在一起。本节中的所有示例包括`v` 范围。
`help` | 列出给定操作的默认标头和其他可用标头。
`h`  |  将输出限制为特定的标题。
`format` |  返回JSON，YAML或CBOR格式的结果。
`sort` | 按指定的列对输出进行分类。

### 查询参数用法示例

您可以为任何CAT操作指定查询参数，以获得更具体的结果。

### 获取详细的输出

要查询别名并获取包含响应中所有列标题的详细输出，请使用`v` 查询参数。

```json
GET _cat/aliases?v
```
{% include copy-curl.html %}

响应提供了更多详细信息，例如响应中每列的名称。

```
alias index filter routing.index routing.search is_write_index
.kibana .kibana_1 - - - -
sample-alias1 sample-index-1 - - - -
```
没有详细参数，`v`，响应只会返回别名名称：

```

.kibana .kibana_1 - - - -
sample-alias1 sample-index-1 - - - -
```

### 获取所有可用标题

要查看所有可用的标题，请使用`help` 范围：

```
GET _cat/<operation_name>?help
```

### 获取标题子

要将输出限制为标头的子集，请使用`h` 范围：

```
GET _cat/<operation_name>?h=<header_name_1>,<header_name_2>&v
```

通常，对于任何操作，您都可以使用`help` 参数，然后使用`h` 参数将输出仅限于您关心的标头。

如果使用安全插件，请确保拥有适当的权限。
{:.note}

