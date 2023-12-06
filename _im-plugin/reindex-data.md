---
layout: default
title: 重新索引数据
nav_order: 15
redirect_from:
  - /opensearch/reindex-data/
---

# 重新索引数据

创建索引后，你可能需要进行大量更改，例如向每个文档添加一个新字段或合并多个索引以形成一个新索引。你可以使用该 `reindex` 操作，而不是删除索引，脱机进行更改，然后再次为数据编制索引。

通过该 `reindex` 操作，可以将通过查询选择的所有文档或部分文档复制到另一个索引。重新索引是一项 `POST` 操作。在最基本的形式中，你可以指定源索引和目标索引。

重新编制索引可能是一项代价高昂的操作，具体取决于源索引的大小。我们建议你通过设置 `number_of_replicas` 目标 `0` 索引中的副本来禁用它们，并在重新索引过程完成后重新启用它们。
{:.note}

---

#### 目录
1. 目录
{:toc}


---

## 重新索引所有文档

你可以将所有文档从一个索引复制到另一个索引。

你首先需要使用所需的字段映射和设置创建目标索引，也可以从源索引中复制这些映射和设置：

```json
PUT destination
{
   "mappings":{
      "Add in your desired mappings"
   },
   "settings":{
      "Add in your desired settings"
   }
}
```

此 `reindex` 命令将所有文档从源索引复制到目标索引：

```json
POST _reindex
{
   "source":{
      "index":"source"
   },
   "dest":{
      "index":"destination"
   }
}
```

如果尚未创建目标索引，则 `reindex` 该操作将使用默认配置创建新的目标索引。

## 从远程群集重新编制索引

你可以从远程群集中的索引复制文档。使用该 `remote` 选项指定远程主机名和所需的登录凭据。

此命令访问远程集群，使用用户名和密码登录，并将该远程集群中的所有文档复制到本地集群中的目标索引：

```json
POST _reindex
{
   "source":{
      "remote":{
         "host":"https://<REST_endpoint_of_remote_cluster>:9200",
         "username":"YOUR_USERNAME",
         "password":"YOUR_PASSWORD"
      },
      "index": "source"
   },
   "dest":{
      "index":"destination"
   }
}
```

你可以指定以下选项：

选项 | 有效值 | 描述 | 必填
:--- | :--- | :---
 `host` | 字符串 | 远程群集的 REST 终结点。| 是的
 `username` | 字符串 | 用于登录远程群集的用户名。| 不
 `password` | 字符串 | 登录到远程群集的密码。| 不
 `socket_timeout` | 时间单位 | 套接字读取的等待时间（默认为 30 秒）。| 不
 `connect_timeout` | 时间单位 | 远程连接超时的等待时间（默认为 30 秒）。| 不


## 重新索引文档的子集

你可以复制与搜索查询匹配的特定文档集。

此命令仅将查询操作匹配的文档子集复制到目标索引：

```json
POST _reindex
{
   "source":{
      "index":"source",
      "query": {
        "match": {
           "field_name": "text"
         }
      }
   },
   "dest":{
      "index":"destination"
   }
}
```

有关所有查询操作的列表，请参见[全文查询]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/full-text/index)。

## 合并一个或多个索引

通过将源索引添加为列表，可以合并一个或多个索引中的文档。

此命令将所有文档从两个源索引复制到一个目标索引：

```json
POST _reindex
{
   "source":{
      "index":[
         "source_1",
         "source_2"
      ]
   },
   "dest":{
      "index":"destination"
   }
}
```
确保源索引和目标索引的分片数相同。

## 仅重新索引唯一文档

通过将选项设置为 `create`，可以仅复制目标索引中缺少的 `op_type` 文档。在这种情况下，如果已存在具有相同 ID 的文档，则操作将忽略源索引中的文档。要忽略文档的所有版本冲突，请将选项 `conflicts` 设置为 `proceed`。

```json
POST _reindex
{
   "conflicts":"proceed",
   "source":{
      "index":"source"
   },
   "dest":{
      "index":"destination",
      "op_type":"create"
   }
}
```

## 在重新编制索引期间转换文档

你可以使用该 `script` 选项在重新编制索引的过程中转换数据。我们建议使用 Painless 在 OpenSearch 中编写脚本。

此命令通过 Painless 脚本运行源索引，该脚本在将对象复制到目标索引之前递增 `number` 对象中的 `account` 字段：

```json
POST _reindex
{
   "source":{
      "index":"source"
   },
   "dest":{
      "index":"destination"
   },
   "script":{
      "lang":"painless",
      "source":"ctx._account.number++"
   }
}
```

还可以指定引入管道，以便在重新编制索引过程中转换数据。

首先必须创建一个具有 defined 的 `processors` 管道。在引入管道中可以使用许多不同的 `processors` 方法。

下面是一个示例引入管道，它定义了一个 `split` 处理器，该处理器根据 `space` 分隔符拆分 `text` 字段并将其存储在新 `word` 字段中。 `script` 处理器是一个 Painless 脚本，用于查找字段的 `word` 长度并将其存储在新 `word_count` 字段中。 `remove` 处理器删除该 `test` 字段。

```json
PUT _ingest/pipeline/pipeline-test
{
"description": "Splits the text field into a list. Computes the length of the 'word' field and stores it in a new 'word_count' field. Removes the 'test' field.",
"processors": [
 {
   "split": {
     "field": "text",
     "separator": "\\s+",
     "target_field": "word"
   },
 }
 {
   "script": {
     "lang": "painless",
     "source": "ctx.word_count = ctx.word.length"
   }
 },
 {
   "remove": {
     "field": "test"
   }
 }
]
}
```

创建流水线后，可以使用以下 `reindex` 操作：

```json
POST _reindex
{
  "source": {
    "index": "source",
  },
  "dest": {
    "index": "destination",
    "pipeline": "pipeline-test"
  }
}
```

## 更新当前索引中的文档

若要更新当前索引本身中的数据而不将其复制到其他索引，请使用该 `update_by_query` 操作。

该 `update_by_query` 操作是 `POST` 一次可以对单个索引执行的操作。

```json
POST <index_name>/_update_by_query
```

如果在不带参数的情况下运行此命令，它将递增索引中所有文档的版本号。

## 源索引选项

你可以为源索引指定以下选项：

选项 | 有效值 | 描述 | 必填
:--- | :--- | :---
 `index` | 字符串 | 源索引的名称。你可以将多个源索引作为列表提供。| 是的
 `max_docs` | 整数 | 要重新编制索引的最大文档数。| 不
 `query` | 对象 | 用于重新索引操作的搜索查询。| 不
 `size` | 整数 | 要重新编制索引的文档数。| 不
 `slice` | 字符串 | 指定手动或自动切片以并行化重新索引。| 不

## 目标索引选项

你可以为目标索引指定以下选项：

选项 | 有效值 | 描述 | 必填
:--- | :--- | :---
 `index` | 字符串 | 目标索引的名称。| 是的
 `version_type` | 枚举 | 索引操作的版本类型。取值范围：internal、external、external_gt、external_gte。| 不

## 索引编解码器注意事项

有关索引编解码器注意事项，请参见[索引编解码器]({{site.url}}{{site.baseurl}}/im-plugin/index-codecs/#reindexing)。
