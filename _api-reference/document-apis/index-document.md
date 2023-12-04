---
layout: default
title: 索引文档
parent: 文档API
nav_order: 1
redirect_from: 
 - /opensearch/rest-api/document-apis/index-document/
---

# 索引文档
**引入1.0**
{: .label .label-purple}

在搜索数据之前，必须首先添加文档。此操作为您的索引添加了一个文档。

## 例子

```json
PUT sample-index/_doc/1
{
  "Description": "To be or not to be, that is the question."
}
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
PUT <index>/_doc/<_id>
POST <index>/_doc

PUT <index>/_create/<_id>
POST <index>/_create/<_id>
```

## URL参数

根据您的要求，您必须指定要添加文档的索引。如果索引尚未存在，OpenSearch会自动创建索引并在您的文档中添加。所有其他URL参数都是可选的。

范围| 类型| 描述| 必需的
:--- | :--- | :--- | :---
＆lt; index＆gt;| 细绳| 索引的名称。| 是的
＆lt; _id＆gt;| 细绳| 一个唯一的标识符，可以连接到文档上。要自动生成ID，请使用`POST <target>/doc` 根据您的要求而不是提取。| 不
if_seq_no| 整数| 仅在文档具有指定的序列编号时执行索引操作。| 不
if_primary_term| 整数| 仅在文档具有指定的主要术语时执行索引操作。| 不
op_type| 枚举| 指定与文档一起完成的操作类型。有效值是`create` （如果不存在，则创建索引）和`index`。如果请求中包含文档ID，则默认值为`index`。否则，默认值为`create`。| 不
管道| 细绳| 将索引操作路由到某个管道。| 不
路由| 细绳| 用于将索引操作分配给特定碎片的值。| 不
刷新| 枚举| 如果是真的，OpenSearch刷新碎片使操作可见。有效的选项是`true`，`false`， 和`wait_for`，它告诉Opensearch在执行操作之前等待刷新。默认值为false。| 不
暂停| 时间| 等待群集的响应多长时间。默认为`1m`。| 不
版本| 整数| 文档的版本号。| 不
version_type| 枚举| 为文档分配特定类型。有效的选项是`external` （如果指定的版本编号大于文档的当前版本，则检索文档）和`external_gte` （如果指定的版本号大于或等于文档的当前版本，则检索文档）。例如，要索引文档的版本3，请使用`/_doc/1?version=3&version_type=external`。| 不
wait_for_active_shards| 细绳| 在OpenSearch处理请求之前，必须可用的活动碎片数。默认值为1（仅是主要碎片）。设置`all` 或一个积极的整数。大于1的值需要复制品。例如，如果指定一个值为3的值，则索引必须在两个其他节点上分布两个副本才能成功。| 不
require_alias| 布尔| 指定目标索引是否必须是索引别名。默认值为false。| 不

## 请求身体

您的请求主体必须包含您要索引的信息。

```json
{
  "Description": "This is just a sample document"
}
```

## 回复
```json
{
  "_index": "sample-index",
  "_id": "1",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```

## 响应身体场

场地| 描述
:--- | :---
_指数| 索引的名称。
_ID| 文档的ID。
_版本| 文档的版本。
结果| 索引操作的结果。
_沙尔德| 有关集群碎片的详细信息。
全部的| 碎片总数。
成功的| shards opensearch的数量成功地添加了文档。
失败的| 碎片数量未能将文档添加到。
_seq_no| 索引文档时分配的序列号。
_primary_term| 索引文档时分配的主要术语。

