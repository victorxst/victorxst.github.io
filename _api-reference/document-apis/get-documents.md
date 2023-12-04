---
layout: default
title: 获取文件
parent: 文档API
nav_order: 5
redirect_from: 
 - /opensearch/rest-api/document-apis/get-documents/
---

# 获取文件
**引入1.0**
{: .label .label-purple }

将JSON文档添加到索引后，您可以使用GET文档API操作检索文档的信息和数据。

## 例子

```json
GET sample-index1/_doc/1
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET <index>/_doc/<_id>
HEAD <index>/_doc/<_id>
```
```
GET <index>/_source/<_id>
HEAD <index>/_source/<_id>
```

## URL参数

所有获取文档URL参数都是可选的。

范围| 类型| 描述
:--- | :--- | :---
偏爱| 细绳| 指定偏爱哪些碎片以检索结果。可用选项是`_local`，该操作告诉操作以从本地分配的碎片副本和分配给特定碎片副本的自定义字符串值来检索结果。默认情况下，OpenSearch在随机碎片上执行获取文档操作。
即时的| 布尔| 指定操作是否应实时运行。如果错误，则该操作等待索引刷新以分析源以检索数据，这使得操作使得该操作接近-即时的。默认是正确的。
刷新| 布尔| 如果为true，则OpenSearch刷新碎片以使GET操作可用于搜索结果。有效的选项是`true`，`false`， 和`wait_for`，它告诉Opensearch在执行操作之前等待刷新。默认为`false`。
路由| 细绳| 用于将操作路由到特定碎片的值。
stored_fields| 布尔| Get操作是否应检索存储在索引中的字段。默认值为false。
_来源| 细绳| 是否包括`_source` 响应体中的田地。默认是正确的。
_source_excludes| 细绳| 逗号-分开的源字段列表要在查询响应中排除。
_source_includes| 细绳| 逗号-分开的源字段列表要包括在查询响应中。
版本| 整数| 要返回的文档的版本，必须与文档的当前版本匹配。
version_type| 枚举| 检索专门键入的文档。可用选项是`external` （如果指定的版本编号大于文档的当前版本，则检索文档）和`external_gte` （如果指定的版本号大于或等于文档的当前版本，则检索文档）。例如，要检索文档的版本3，请使用`/_doc/1?version=3&version_type=external`。


## 回复
```json
{
  "_index": "sample-index1",
  "_id": "1",
  "_version": 1,
  "_seq_no": 0,
  "_primary_term": 9,
  "found": true,
  "_source": {
    "text": "This is just some sample text."
  }
}
```

## 响应身体场

场地| 描述
:--- | :---
_指数| 索引的名称。
_ID| 文档的ID。
_版本| 文档的版本号。每当文档更改时都会更新。
_seq_no| 索引文档时分配的序列号。
primal_term| 索引文档时分配的主要术语。
成立| 文档是否存在。
_路由| 该文档的碎片被路由。如果未将文档路由到特定的碎片，则将省略此字段。
_来源| 包含文档的数据`found` 是真的。如果`_source` 设置为false或`stored_fields` 在URL参数中设置为true，该字段省略了。
_Fields| 包含存储在索引中的文档的数据。仅当两者都返回`stored_fields` 和`found` 是真的。

