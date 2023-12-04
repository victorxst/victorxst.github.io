---
layout: default
title: 并发-获取文件
parent: 文档API
nav_order: 30
redirect_from: 
 - /opensearch/rest-api/document-apis/multi-get/
---

# 并发-获取文件
**引入1.0**
{: .label .label-purple }

并发-获取操作允许您在一个请求中运行多个获取操作，因此您可以获取符合您条件的所有文档。

## 路径和HTTP方法

```
GET _mget
GET <index>/_mget
POST _mget
POST <index>/_mget
```

## URL参数

全部并发-获取URL参数是可选的。

范围| 类型| 描述
:--- | :--- | :--- | :---
＆lt; index＆gt;| 细绳| 从中检索文档的索引的名称。
偏爱| 细绳| 指定节点或碎片OpenSearch应执行多个-继续操作。默认值是随机的。
即时的| 布尔| 指定操作是否应实时运行。如果错误，则该操作等待索引刷新以分析源以检索数据，这使得操作使得该操作接近-即时的。默认为`true`。
刷新| 布尔| 如果是真的，OpenSearch刷新碎片以使多个-获取可用的操作来搜索结果。有效的选项是`true`，`false`， 和`wait_for`，它告诉Opensearch在执行操作之前等待刷新。默认为`false`。
路由| 细绳| 用来路由多人的价值-将操作运行到特定的碎片。
stored_fields| 布尔| 指定OpenSearch是否应该从索引检索文档字段，而不是文档的`_source`。默认为`false`。
_来源| 细绳| 是否包括`_source` 查询响应中的字段。默认为`true`。
_source_excludes| 细绳| 逗号-分开的源字段列表要在查询响应中排除。
_source_includes| 细绳| 逗号-分开的源字段列表要包括在查询响应中。

## 请求身体

如果您没有在请求的URL中指定索引，则必须指定您的目标索引和请求主体中的相关文档ID。其他字段是可选的。

场地| 类型| 描述| 必需的
:--- | :--- | :--- | :---
文档| 大批| 您要从中检索数据的文档。可以包含属性：`_id`，`_index`，`_routing`，`_source`， 和`_stored_fields`。如果您在URL中指定索引，则可以省略此字段并添加文档的ID以检索。| 是的，如果URL中未指定索引
_ID| 细绳| 文档的ID。| 是的，如果`docs` 在请求正文中指定
_指数| 细绳| 索引的名称。| 是的，如果URL中未指定索引
_路由| 细绳| 具有文档的碎片的价值。| 是的，如果索引文档时使用了路由值
_来源| 目的| 指定是否返回`_source` 来自索引（布尔）的字段，是否返回特定字段（数组），还是包括或排除某些字段。| 不
_source.在内| 大批| 指定在查询响应中包含哪些字段。例如，`"_source": { "include": ["Title"] }` 检索`Title` 从索引。| 不
_source.Excludes| 大批| 指定在查询响应中排除哪些字段。例如，`"_source": { "exclude": ["Director"] }` 排除`Director` 从查询响应中。| 不
IDS| 大批| 文件的ID。仅在URL中指定索引时允许。| 不


#### 示例未在URL中指定索引

```json
GET _mget
{
  "docs": [
  {
    "_index": "sample-index1",
    "_id": "1"
  },
  {
    "_index": "sample-index2",
    "_id": "1",
    "_source": {
      "include": ["Length"]
    }
  }
  ]
}
```
{% include copy-curl.html %}

#### URL中指定索引的示例

```json
GET sample-index1/_mget
{
  "docs": [
    {
      "_id": "1",
      "_source": false
    },
    {
      "_id": "2",
      "_source": [ "Director", "Title" ]
    }
  ]
}
```
{% include copy-curl.html %}

#### 示例响应
```json
{
  "docs": [
    {
      "_index": "sample-index1",
      "_id": "1",
      "_version": 4,
      "_seq_no": 5,
      "_primary_term": 19,
      "found": true,
      "_source": {
        "Title": "Batman Begins",
        "Director": "Christopher Nolan"
      }
    },
    {
      "_index": "sample-index2",
      "_id": "1",
      "_version": 1,
      "_seq_no": 6,
      "_primary_term": 19,
      "found": true,
      "_source": {
        "Title": "The Dark Knight",
        "Director": "Christopher Nolan"
      }
    }
  ]
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

