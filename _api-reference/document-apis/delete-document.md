---
layout: default
title: 删除文档
parent: 文档API
nav_order: 15
redirect_from: 
 - /opensearch/rest-api/document-apis/delete-document/
---

# 删除文档
**引入1.0**
{: .label .label-purple }

如果您不再需要索引中的文档，则可以使用删除文档API操作将其删除。

## 例子

```
DELETE /sample-index1/_doc/1
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
DELETE /<index>/_doc/<_id>
```

## URL参数

范围| 类型| 描述| 必需的
:--- | :--- | :--- | :---
＆lt; index＆gt;| 细绳| 从中删除的索引。| 是的
＆lt; _id＆gt;| 细绳| 文档的ID要删除。| 是的
if_seq_no| 整数| 仅在文档的版本编号匹配指定的数字时执行删除操作。| 不
if_primary_term| 整数| 仅在文档具有指定的主要项时执行删除操作。| 不
刷新| 枚举| 如果为true，OpenSearch刷新碎片以使删除操作可用于搜索结果。有效的选项是`true`，`false`， 和`wait_for`，它告诉Opensearch在执行操作之前等待刷新。默认为`false`。| 不
路由| 细绳| 用于将操作路由到特定碎片的值。| 不
暂停| 时间| 等待群集的响应多长时间。默认为`1m`。| 不
版本| 整数| 要删除文档的版本，必须匹配文档的最后更新版本。| 不
version_type| 枚举| 检索专门键入的文档。可用选项是`external` （如果指定的版本编号大于文档的当前版本，则检索文档）和`external_gte` （如果指定的版本号大于或等于文档的当前版本，则检索文档）。例如，要删除文档的版本3，请使用`/_doc/1?version=3&version_type=external`。| 不
wait_for_active_shards| 细绳| 在OpenSearch处理删除请求之前，必须可用的活动碎片数。默认值为1（仅是主要碎片）。设置`all` 或一个积极的整数。大于1的值需要复制品。例如，如果指定一个值为3的值，则索引必须在两个其他节点上分布两个副本才能成功。| 不


## 回复
```json
{
  "_index": "sample-index1",
  "_id": "1",
  "_version": 2,
  "result": "deleted",
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "_seq_no": 1,
  "_primary_term": 15
}
```

## 响应身体场

场地| 描述
:--- | :---
_指数| 索引的名称。
_ID| 文档的ID。
_版本| 文档的版本。
_结果| 删除操作的结果。
_沙尔德| 有关集群碎片的详细信息。
全部的| 碎片总数。
成功的| OpenSearch的碎片数量成功地删除了该文档。
失败的| 碎片数量未能从文档中删除。
_seq_no| 索引文档时分配的序列号。
_primary_term| 索引文档时分配的主要术语。

