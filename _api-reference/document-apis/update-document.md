---
layout: default
title: 更新文档
parent: 文档API
nav_order: 10
redirect_from: 
 - /opensearch/rest-api/document-apis/update-document/
---

# 更新文档
**引入1.0**
{: .label .label-purple }

如果您需要在索引中更新文档字段，则可以使用“更新文档API操作”。您可以通过在索引中指定所需的新数据或通过在请求主体中包含脚本来做到这一点，而OpenSearch运行以更新文档。

## 例子

```json
POST /sample-index1/_update/1
{
  "doc": {
    "first_name" : "Bruce",
    "last_name" : "Wayne"
  }
}
```
{% include copy-curl.html %}

## 脚本示例

```json
POST /test-index1/_update/1
{
  "script" : {
    "source": "ctx._source.secret_identity = \"Batman\""
  }
}
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
POST /<index>/_update/<_id>
```

## URL参数

范围| 类型| 描述| 必需的
:--- | :--- | :--- | :---
＆lt; index＆gt;| 细绳| 索引的名称。| 是的
＆lt; _id＆gt;| 细绳| 文档的ID更新。| 是的
if_seq_no| 整数| 仅在文档具有指定的序列编号时执行更新操作。| 不
if_primary_term| 整数| 如果文档具有指定的主要术语，则执行更新操作。| 不
朗| 细绳| 脚本的语言。默认为`painless`。| 不
require_alias| 布尔| 指定目标是否必须是索引别名。默认值为false。| 不
刷新| 枚举| 如果是真的，OpenSearch刷新碎片使操作可见。有效的选项是`true`，`false`， 和`wait_for`，它告诉Opensearch在执行操作之前等待刷新。默认为`false`。| 不
retry_on_conflict| 整数| 如果有文档冲突，则OpenSearch的次数应重试操作。默认值为0。| 不
路由| 细绳| 将更新操作路由到特定碎片的值。| 不
_来源| 布尔或列表| 是否包括`_source` 响应体中的田地。默认为`false`。此参数还支持逗号-分开的源字段列表，用于在查询响应中包含多个源字段。| 不
_source_excludes| 列表| 逗号-分开的源字段列表要在查询响应中排除。| 不
_source_includes| 列表| 逗号-分开的源字段列表要包括在查询响应中。| 不
暂停| 时间| 等待群集的响应多长时间。| 不
wait_for_active_shards| 细绳| 在OpenSearch处理更新请求之前，必须可用的活动碎片数量。默认值为1（仅是主要碎片）。设置`all` 或一个积极的整数。大于1的值需要复制品。例如，如果指定一个值为3的值，则索引必须在两个其他节点上分布两个副本才能成功。| 不

## 请求身体

您的请求主体必须包含您要更新文档的信息。如果您只想替换文档中的某些字段，您的请求正文必须包括一个`doc` 对象，其中您要更新的字段。

```json
{
  "doc": {
    "first_name": "Thomas",
    "last_name": "Wayne"
    }
}
```

您还可以使用脚本告诉Opensearch如何更新文档。

```json
{
  "script" : {
    "source": "ctx._source.oldValue += params.newValue",
    "lang": "painless",
    "params" : {
      "newValue" : 10
    }
  }
}
```

### UPSERT

UPSERT是一种有条件地更新现有文档或根据对象中信息插入新文档的操作。在下面的示例中，`upsert` 操作更新`last name` 并添加`first_name` 字段如果文档已经存在。如果不存在文档，则使用新的文档使用内容索引`upsert` 目的。

```json
{
  "doc": {
    "first_name": "Martha",
    "last_name": "Rivera"
  },
  "upsert": {
    "last_name": "Oliveira",
    "age": "31"
  }
}
```
您也可以添加`doc_as_upsert` 根据请求并将其设置为`true` 使用信息`doc` 用于执行UpSert操作。

```json
{
  "doc": {
    "first_name": "Martha",
    "last_name": "Oliveira",
    "age": "31"
  },
  "doc_as_upsert": true
}
```

## 回复
```json
{
  "_index": "sample-index1",
  "_id": "1",
  "_version": 3,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "_seq_no": 4,
  "_primary_term": 17
}
```

## 响应身体场

场地| 描述
:--- | :---
_指数| 索引的名称。
_ID| 文档的ID。
_版本| 文档的版本。
_结果| 更新操作的结果。
_沙尔德| 有关集群碎片的详细信息。
全部的| 碎片总数。
成功的| shards opensearch的数量成功地更新了文档。
失败的| OpenSearch的碎片数未能更新文档。
_seq_no| 索引文档时分配的序列号。
_primary_term| 索引文档时分配的主要术语。

