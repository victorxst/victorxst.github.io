---
layout: default
title: 还原快照
parent: 快照API

nav_order: 9
---

# 还原快照
**引入1.0**
{: .label .label-purple }

还原集群或指定数据流和索引的快照。

*有关索引和集群的信息，请参阅[OpenSearch简介]({{site.url}}{{site.baseurl}}/opensearch/index)。

*有关数据流的信息，请参阅[数据流]({{site.url}}{{site.baseurl}}/opensearch/data-streams)。

如果打开的索引与要还原的同名索引已经存在于集群中，则必须关闭，删除或重命名索引。看[示例请求](#example-request) 有关重命名索引的信息。看[关闭索引]({{site.url}}{{site.baseurl}}/api-reference/index-apis/close-index) 有关关闭索引的信息。
{:.note}

## 路径参数

| 范围| 数据类型| 描述|
:--- | :--- | :---
存储库| 细绳| 存储库包含快照以还原。|
| 快照| 细绳| 快照要恢复。|

## 查询参数

范围| 数据类型| 描述
:--- | :--- | :---
wait_for_completion| 布尔|  是否要等待快照恢复才能完成。|

### 请求字段

所有请求的身体参数都是可选的。

| 范围| 数据类型| 描述|
:--- | :--- | :--- 
| ignore_unavailable| 布尔| 如何处理丢失或关闭的数据流或索引。如果`false`，该请求返回丢失或关闭的任何数据流或索引的错误。如果`true`，该请求忽略了缺少或关闭的索引中的数据流和索引。默认为`false`。|
| ignore_index_settings| 布尔| 逗号-您不想从快照还原的索引设置列表。|
| 包括_ALIASES| 布尔| 如何处理原始快照中的索引别名。如果`true`，恢复了原始快照中的索引别名。如果`false`，尚未恢复别名以及相关指数。默认为`true`。|
| 包括_global_state| 布尔| 是否还原当前的群集状态<sup> 1 </sup>。如果`false`，群集状态未恢复。如果为true，则将恢复当前的群集状态。默认为`false`。|
| index_settings| 细绳| 逗号-划定的设置列表，以添加或更改所有还原索引。在快照修复期间，使用此参数覆盖索引设置。对于数据流，将这些索引设置应用于还原的备份索引。|
| 指数| 细绳| 逗号-划界数据流的列表和索引以从快照恢复。并发-支持索引语法。默认情况下，还原操作包括快照中的所有数据流和索引。如果提供了此参数，则还原操作仅包括您指定的数据流和索引。|
| 部分的| 布尔| 如果快照中的索引没有所有主要碎片，那么还原操作将如何行为。如果`false`，如果快照中的任何索引没有所有主要碎片，则整个还原操作会失败。<br /> <br /> if`true`，允许恢复不可用的碎片的部分索引快照。仅恢复了快照中成功包含的碎片。所有缺少的碎片都被重新创建为空。默认情况下，如果快照中包含一个或多个索引，则整个还原操作将失败。要改变这种行为，请设置`partial` 到`true`。默认为`false`。|
| Rename_pattern| 细绳| 适用于还原的数据流和索引的模式。与重命名模式相匹配的数据流和索引将根据`rename_replacement`。<br /> <br />将重命名模式应用于支持引用原始文本的正则表达式所定义。<br /> <br />如果两个或多个数据流或索引被重命名为同一名称，请求将失败。如果您重命名已恢复的数据流，则其备份索引也会重命名。例如，如果将日志数据流命名为`recovered-logs`，背景索引`.ds-logs-1` 被更名为`.ds-recovered-logs-1`。<br /> <br />如果您重命名已修复的流，请确保索引模板匹配新的流名称。如果没有匹配的索引模板名称，则该流将无法翻滚，并且未创建新的备份索引。|
| Rename_redlacement| 细绳| 重命名替换字符串。看`rename_pattern` 了解更多信息。|
| source_remote_store_repository| 细绳| 恢复源索引的远程存储存储库的名称。如果未提供，快照还原API将使用创建快照时注册的存储库。
| wait_for_completion| 布尔| 是否要在还原操作完成后返回响应。如果`false`，当还原操作初始化时，请求将返回响应。如果`true`，当恢复操作完成后，请求返回响应。默认为`false`。|

<sup> 1 </sup>群集状态包括：
*持续的集群设置
*索引模板
*传统索引模板
*摄取管道
*索引生命周期政策

#### 示例请求

以下请求还原`opendistro-reports-definitions` 索引来自`my-first-snapshot`。这`rename_pattern` 和`rename_replacement` 组合导致索引重命名为`opendistro-reports-definitions_restored` 因为不允许在集群中重复开放索引名称。

````json
POST /_snapshot/my-opensearch-repo/my-first-snapshot/_restore
{
  "indices": "opendistro-reports-definitions",
  "ignore_unavailable": true,
  "include_global_state": false,              
  "rename_pattern": "(.+)",
  "rename_replacement": "$1_restored",
  "include_aliases": false
}
````

#### 示例响应

成功后，响应返回以下JSON对象：

````json
{
  "snapshot" : {
    "snapshot" : "my-first-snapshot",
    "indices" : [ ],
    "shards" : {
      "total" : 0,
      "failed" : 0,
      "successful" : 0
    }
  }
}
````
除快照名称外，所有属性均为空或`0`。这是因为丢失了快照后对音量进行的任何更改。但是，如果您调用[获取快照]({{site.url}}{{site.baseurl}}/api-reference/snapshots/get-snapshot) API检查快照，返回一个填充的快照对象。

## 响应字段

| 场地| 数据类型| 描述|
| :--- | :--- | :--- | 
| 快照| 细绳| 快照名称。|
| 指数| 大批| 快照中的索引。|
| 碎片| 目的| 创建的碎片总数以及成功和失败的碎片数量。|

如果快照中的打开索引已经存在于集群中，并且您不会删除，关闭或重命名它们，则API返回以下错误：
{:.note}

````json
{
  "error" : {
    "root_cause" : [
      {
        "type" : "snapshot_restore_exception",
        "reason" : "[my-opensearch-repo:my-first-snapshot/dCK4Qth-TymRQ7Tu7Iga0g] cannot restore index [.opendistro-reports-definitions] because an open index with same name already exists in the cluster. Either close or delete the existing index or restore the index under a different name by providing a rename pattern and replacement name"
      }
    ],
    "type" : "snapshot_restore_exception",
    "reason" : "[my-opensearch-repo:my-first-snapshot/dCK4Qth-TymRQ7Tu7Iga0g] cannot restore index [.opendistro-reports-definitions] because an open index with same name already exists in the cluster. Either close or delete the existing index or restore the index under a different name by providing a rename pattern and replacement name"
  },
  "status" : 500
}
````
