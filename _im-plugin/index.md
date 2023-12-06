---
layout: default
title: 管理索引
nav_order: 1
has_children: false
nav_exclude: true
redirect_from:
  - /im-plugin/
  - /opensearch/index-data/
  - /opensearch/rest-api/index-apis/index/
---

# 管理索引

你可以使用 OpenSearch REST API 为数据编制索引。存在两个 API：索引 API 和 `_bulk` API。

对于新数据以增量方式到达的情况（例如，来自小型企业的客户订单），你可以使用索引 API 在文档到达时单独添加文档。对于数据流不太频繁的情况（例如，营销网站的每周更新），你可能更愿意生成文件并将其发送到 `_bulk` API。对于大量文档，将请求集中在一起并使用 `_bulk` API 可提供卓越的性能。但是，如果你的文档非常庞大，则可能需要单独为它们编制索引。

为文档编制索引时，文档 `_id` 的大小不得超过 512 字节。


## 索引简介

在搜索数据之前，必须*指数*搜索数据。索引是搜索引擎组织数据以进行快速检索的方法。生成的结构被恰当地称为索引。

在 OpenSearch 中，数据的基本单位是 JSON *公文*。在索引中，OpenSearch 使用唯一 ID 标识每个文档。

对索引 API 的请求如下所示：

```json
PUT <index>/_doc/<id>
{ "A JSON": "document" }
```

对 `_bulk` API 的请求看起来略有不同，因为你在批量数据中指定了索引和 ID：

```json
POST _bulk
{ "index": { "_index": "<index>", "_id": "<id>" } }
{ "A JSON": "document" }
```

批量数据必须符合特定格式，该格式要求在每行（包括最后一行）的末尾使用换行符（ `\n`）。这是基本格式：

```
Action and metadata\n
Optional document\n
Action and metadata\n
Optional document\n
```

文档是可选的，因为 `delete` 操作不需要文档。其他操作（ `index`、 `create` 和 `update`）都需要文档。如果明确希望在文档已存在时操作失败，请使用操作 `create` 而不是 `index` 操作。{: .note}

若要使用该 `curl` 命令为批量数据编制索引，请导航到保存文件的文件夹，然后运行以下命令：

```json
curl -H "Content-Type: application/x-ndjson" -POST https://localhost:9200/data/_bulk -u 'admin:admin' --insecure --data-binary "@data.json"
```

如果 API 中 `_bulk` 的任何一个操作失败，OpenSearch 将继续执行其他操作。检查 `items` 响应中的数组，找出问题所在。数组中的 `items` 条目与请求中指定的操作的顺序相同。

当你将文档添加到尚不存在的索引时，OpenSearch 会自动创建索引。如果你未在请求中指定 ID，它还会自动生成一个 ID。这个简单的示例会自动创建电影索引，为文档编制索引，并为其分配一个唯一的 ID：

```json
POST movies/_doc
{ "title": "Spirited Away" }
```

自动生成 ID 有一个明显的缺点：由于索引请求未指定文档 ID，因此以后无法轻松更新文档。此外，如果你运行此请求 10 次，OpenSearch 会将此文档索引为具有唯一 ID 的 10 个不同文档。若要指定 ID 1，请使用以下请求（请注意使用 PUT 而不是 POST）：

```json
PUT movies/_doc/1
{ "title": "Spirited Away" }
```

由于必须指定 ID，因此，如果运行此命令 10 次，则仍然只有一个文档为该字段递增至 10 的文档编制索引 `_version`。

索引默认为一个主分片和一个副本。如果要指定非默认设置，请在添加文档之前创建索引：

```json
PUT more-movies
{ "settings": { "number_of_shards": 6, "number_of_replicas": 2 } }
```

## 索引的命名限制

OpenSearch 索引具有以下命名限制：

- 所有字母必须为小写。
- 索引名称不能以下划线（ `_`）或连字符（ `-`）开头。
- 索引名称不能包含空格、逗号或以下字符：

   `:`、、 `"`、、 `/` `*` `+` `\` `|` `?` `#` `>` `<`



## 读取数据

为文档编制索引后，可以通过向用于编制索引的同一终结点发送 GET 请求来检索该文档：

```json
GET movies/_doc/1

{
  "_index" : "movies",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "title" : "Spirited Away"
  }
}
```

你可以在对象中看到 `_source` 文档。如果未找到文档，则键是 `false`， `found` `_source` 对象不是响应的一部分。

若要使用单个命令检索多个文档，请使用该 `_mget` 操作。检索多个文档的格式类似于 `_bulk` 操作，你必须在请求正文中指定索引和 ID：

```json
GET _mget
{
  "docs": [
    {
      "_index": "<index>",
      "_id": "<id>"
    },
    {
      "_index": "<index>",
      "_id": "<id>"
    }
  ]
}
```

要仅返回文档中的特定字段，请执行以下操作：

```json
GET _mget
{
  "docs": [
    {
      "_index": "<index>",
      "_id": "<id>",
      "_source": "field1"
    },
    {
      "_index": "<index>",
      "_id": "<id>",
      "_source": "field2"
    }
  ]
}
```

要检查文档是否存在：

```json
HEAD movies/_doc/<doc-id>
```

如果文档存在，则返回响应 `200 OK`，如果 `404 - Not Found` 不存在，则返回错误。

## 更新数据

若要更新现有字段或添加新字段，请向 `_update` 操作发送 POST 请求，其中包含对象中的 `doc` 更改：

```json
POST movies/_update/1
{
  "doc": {
    "title": "Castle in the Sky",
    "genre": ["Animation", "Fantasy"]
  }
}
```

请注意更新 `title` 的字段和新 `genre` 字段：

```json
GET movies/_doc/1

{
  "_index" : "movies",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "title" : "Castle in the Sky",
    "genre" : [
      "Animation",
      "Fantasy"
    ]
  }
}
```

该文档还有一个递 `_version` 增的字段。使用此字段可以跟踪文档的更新次数。

POST 请求对文档进行部分更新。要完全替换文档，请使用 PUT 请求：

```json
PUT movies/_doc/1
{
  "title": "Spirited Away"
}
```

ID 为 1 的文档将仅包含该 `title` 字段，因为整个文档将替换为此 PUT 请求中索引的文档。

使用该 `upsert` 对象可以根据文档是否已存在来有条件地更新文档。在这里，如果文档存在，则其 `title` 字段将更改为 `Castle in the Sky`。如果没有，OpenSearch 将对对象中的 `upsert` 文档编制索引。

```json
POST movies/_update/2
{
  "doc": {
    "title": "Castle in the Sky"
  },
  "upsert": {
    "title": "Only Yesterday",
    "genre": ["Animation", "Fantasy"],
    "date": 1993
  }
}
```

### 响应示例

```json
{
  "_index" : "movies",
  "_type" : "_doc",
  "_id" : "2",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 3,
  "_primary_term" : 1
}
```

文档的每个更新操作都具有唯一的和 `_primary_term` 值组合 `_seq_no`。

OpenSearch 首先将你的更新写入主分片，然后将此更改发送到所有副本分片。如果基于 OpenSearch 的应用程序的多个用户对同一索引中的现有文档进行更新，则可能会出现不常见的问题。在这种情况下，其他用户可以在从主分片接收更新之前从副本读取和更新文档。然后，你的更新操作最终会更新文档的旧版本。在最好的情况下，你和其他用户进行相同的更改，并且文档保持准确。在最坏的情况下，文档现在包含过时的信息。

若要防止出现这种情况，请在 `_seq_no` 请求标头中使用 and `_primary_term` 值：

```json
POST movies/_update/2?if_seq_no=3&if_primary_term=1
{
  "doc": {
    "title": "Castle in the Sky",
    "genre": ["Animation", "Fantasy"]
  }
}
```

如果在检索文档后更新了文档，则和 `_primary_term` 值不同， `_seq_no` 并且更新操作将失败并显示 `409 — Conflict` 错误。

使用 `_bulk` API 时，请在操作元数据中指定 `_seq_no` 和 `_primary_term` 值。

## 删除数据

若要从索引中删除文档，请使用 DELETE 请求：

```json
DELETE movies/_doc/1
```

DELETE 操作使 `_version` 该字段递增。如果将文档添加回同一 ID，则该 `_version` 字段将再次递增。出现此现象的原因是 OpenSearch 删除了文档 `_source`，但保留了其元数据。


## 后续步骤

- 索引管理（IM）插件可让你自动执行重复的索引管理活动并降低存储成本。有关详细信息，请参阅[索引状态管理]({{site.url}}{{site.baseurl}}/im-plugin/ism/index)。

- 有关如何重新索引数据的说明，请参见[重新索引数据]({{site.url}}{{site.baseurl}}/im-plugin/reindex-data/)。

