---
layout: default
title: 主体
parent: 文档API
nav_order: 20
redirect_from:
 - /opensearch/rest-api/document-apis/bulk/
---

# 主体
**引入1.0**
{: .label .label-purple }

批量操作使您可以在一个请求中添加，更新或删除多个文档。与单个OpenSearch索引请求相比，大量操作具有显着的性能优势。每当实用的情况下，我们建议将索引操作批量操作到批量请求中。


从OpenSearch 2.9开始，当使用批量操作索引文档时`_id` 大小必须为512个字节或更小。
{: .note}

## 例子

```json
POST _bulk
{ "delete": { "_index": "movies", "_id": "tt2229499" } }
{ "index": { "_index": "movies", "_id": "tt1979320" } }
{ "title": "Rush", "year": 2013 }
{ "create": { "_index": "movies", "_id": "tt1392214" } }
{ "title": "Prisoners", "year": 2013 }
{ "update": { "_index": "movies", "_id": "tt0816711" } }
{ "doc" : { "title": "World War Z" } }

```
{% include copy-curl.html %}


## 路径和HTTP方法

```
POST _bulk
POST <index>/_bulk
```

在路径中指定索引意味着您不需要将其包括在[请求身体]({{site.url}}{{site.baseurl}}/api-reference/document-apis/bulk/#request-body)。

OpenSearch还接受将请求提交给`_bulk` 路径，但我们强烈建议使用帖子。公认的看法---在给定路径上添加或替换单个资源---对于批量请求没有意义。
{: .note }


## URL参数

所有批量URL参数都是可选的。

范围| 类型| 描述
:--- | :--- | :---
管道| 细绳| 用于预处理文档的管道ID。
刷新| 枚举| 是否在执行索引操作后刷新受影响的碎片。默认为`false`。`true` 使更改立即显示在搜索结果中，但会损害群集性能。`wait_for` 等待刷新。请求需要更长的时间才能返回，但是集群性能不会受苦。
require_alias| 布尔| 设置`true` 要求所有动作针对索引别名而不是索引。默认为`false`。
路由| 细绳| 将请求路由到指定的碎片。
暂停| 时间| 等待请求返回多长时间。默认`1m`。
类型| 细绳| （已弃用）未指定类型的文档的默认文档类型。默认为`_doc`。我们强烈建议您忽略此参数，并使用一种类型`_doc` 对于所有索引。
wait_for_active_shards| 细绳| 指定在OpenSearch处理批量请求之前必须可用的活动碎片数。默认值为1（仅是主要碎片）。设置`all` 或一个积极的整数。大于1的值需要复制品。例如，如果指定一个值为3的值，则该索引必须在两个其他节点上分布两个副本才能成功。
{％注释％} _来源| 列表| ASDF
_source_excludes| 列表| ASDF
_source_includes| 列表| ASDF {％端次}


## 请求身体

批量请求主体遵循以下模式：

```
Action and metadata\n
Optional document\n
Action and metadata\n
Optional document\n

```

可选的JSON文档不需要更新---空间很好---但是它确实需要在一条线上。OpenSearch使用Newline字符来解析批量请求，并要求请求主体以Newline字符结束。

所有动作都支持相同的元数据：`_index`，`_id`， 和`_require_alias`。如果您不提供ID，则OpenSearch会自动生成一个ID，这可能会使以后更新文档更具挑战性。

- 创造

  如果文档尚不存在，则创建文档，否则会返回错误。下一行必须包括一个JSON文档。

  ```json
  { "create": { "_index": "movies", "_id": "tt1392214" } }
  { "title": "Prisoners", "year": 2013 }
  ```

- 删除

  如果存在此操作，则会删除文档。如果文档不存在，OpenSearch不会返回错误，而是返回`not_found` 在下面`result`。删除操作不需要下一行上的文档。

  ```json
  { "delete": { "_index": "movies", "_id": "tt2229499" } }
  ```

- Index

  Index actions create a document if it doesn't yet exist and replace the document if it already exists. The next line must include a JSON document.

  ```json
  { "index": { "_index": "movies", "_id": "tt1979320" } }
  { "title": "Rush", "year": 2013}
  ```

- Update

  This action updates existing documents and returns an error if the document doesn't exist. The next line must include a full or partial JSON document, depending on how much of the document you want to update.

  ```json
  { "update": { "_index": "movies", "_id": "tt0816711" } }
  { "doc" : { "title": "World War Z" } }
  ```

  It can also include a script or upsert for more complex document updates.

  - Script
  ```json
  { "update": { "_index": "movies", "_id": "tt0816711" } }
  { "script" : { "source": "ctx._source.title = \"World War Z\"" } }
  ```

  - Upsert
  ```json
  { "update": { "_index": "movies", "_id": "tt0816711" } }
  { "doc" : { "title": "World War Z" }, "doc_as_upsert": true }
  ```

## 回复

在回应中，特别注意顶部-等级`errors` 布尔。如果是真的，您可以迭代个人操作以获取更多详细信息。

```json
{
  "took": 11,
  "errors": true,
  "items": [
    {
      "index": {
        "_index": "movies",
        "_id": "tt1979320",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 1,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "create": {
        "_index": "movies",
        "_id": "tt1392214",
        "status": 409,
        "error": {
          "type": "version_conflict_engine_exception",
          "reason": "[tt1392214]: version conflict, document already exists (current version [1])",
          "index": "movies",
          "shard": "0",
          "index_uuid": "yhizhusbSWmP0G7OJnmcLg"
        }
      }
    },
    {
      "update": {
        "_index": "movies",
        "_id": "tt0816711",
        "status": 404,
        "error": {
          "type": "document_missing_exception",
          "reason": "[_doc][tt0816711]: document missing",
          "index": "movies",
          "shard": "0",
          "index_uuid": "yhizhusbSWmP0G7OJnmcLg"
        }
      }
    }
  ]
}
```

