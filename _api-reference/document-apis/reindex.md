---
layout: default
title: Reindex文档
parent: 文档API
nav_order: 60
redirect_from: 
  - /opensearch/reindex-data/
  - /opensearch/rest-api/document-apis/reindex/
---

# Reindex文档
**引入1.0**
{: .label .label-purple}

Reindex文档API操作使您可以将数据的全部或一个子集从源索引复制到目标索引中。

## 例子

```json
POST /_reindex
{
   "source":{
      "index":"my-source-index"
   },
   "dest":{
      "index":"my-destination-index"
   }
}
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
POST /_reindex
```

## URL参数

所有URL参数都是可选的。

范围| 类型| 描述
:--- | :--- | :---
刷新| 布尔| 如果为true，则OpenSearch刷新碎片以使ReIndex操作可用于搜索结果。有效的选项是`true`，`false`， 和`wait_for`，它告诉Opensearch在执行操作之前等待刷新。默认为`false`。
暂停| 时间| 等待群集的响应多长时间。默认为`30s`。
wait_for_active_shards| 细绳| 在OpenSearch处理ReIndex请求之前，必须可用的活动碎片数。默认值为1（仅是主要碎片）。设置`all` 或一个积极的整数。大于1的值需要复制品。例如，如果指定一个值为3的值，则索引必须在两个其他节点上分布两个副本才能成功。
wait_for_completion| 布尔| 等待匹配任务完成。默认为`false`。
requests_per_second| 整数| 指定请求在sub中的节流-每秒请求。默认为-1，这意味着没有节流。
require_alias| 布尔| 目的地索引是否必须是索引别名。默认值为false。
滚动| 时间| 保持搜索上下文开放多长时间。默认为`5m`。
切片| 整数| 子数量-任务OpenSearch应将此任务分为。默认值为1，这意味着OpenSearch不应划分此任务。将此参数设置为`auto` 向OpenSearch表示，它应该自动决定将任务分为多个切片。
max_docs| 整数| 查询操作更新的更新最多应处理多少文档。默认是所有文档。

## 请求身体

您的请求主体必须包含源索引和目标索引的名称。所有其他字段都是可选的。

场地| 描述
:--- | :---
冲突| 指示开放搜索，如果通过查询操作删除将发生什么会发生什么，则会发生在版本冲突中。有效的选项是`abort` 和`proceed`。默认值为中止。
来源| 有关要包括的源索引的信息。有效字段是`index`，`max_docs`，`query`，`remote`，`size`，`slice`， 和`_source`。
指数| 源索引复制数据的名称。
max_docs| 最多要重新索引的文档数量。
询问| 用于Reindex操作的搜索查询。
偏僻的| 有关远程OpenSearch集群的信息，以复制数据。有效字段是`host`，`username`，`password`，`socket_timeout`， 和`connect_timeout`。
主持人| OpenSearch集群的主机URL从中复制数据。
用户名| 用户名可使用远程群集进行身份验证。
密码| 密码可以使用远程群集进行身份验证。
socket_timeout| 插座的等待时间读取。默认值为30。
connect_timeout| 远程连接超时的等待时间。默认值为30。
尺寸| reindex的文档数量。
片| 是手动还是自动切成reindex操作，以便并并行执行。将此字段设置为`auto` 允许OpenSearch控制使用的切片数，即每片一个切片，最多20片。如果有多个来源，所使用的切片数是基于索引或备份索引，最小数量的碎片数。
_来源| 是否要重新索引源字段。指定要重新索引的字段列表或true以重新索引所有字段。默认是正确的。
ID| 与手动切片相关联的ID。
最大限度| 最大切片数。
命运| 有关目标索引的信息。有效值是`index`，`version_type`， 和`op_type`。
指数| 目标索引的名称。
version_type| 索引操作的版本类型。有效值是`internal`，`external`，`external_gt` （如果指定的版本号大于文档的当前版本，则检索文档），并且`external_gte` （如果指定的版本编号大于或等于文档的当前版本，则检索文档）。
op_type| 是否要复制目标索引中缺少的文档。有效值是`create` （忽略来自源索引中具有相同ID的文档）和`index` （复制源索引中的所有内容）。
脚本| OpenSearch使用的脚本将转换应用于ReIndex操作期间的数据。
来源| OpenSearch运行的实际脚本。
朗| 脚本语言。有效的选项是`painless`，`expression`，`mustache`， 和`java`。

## 回复
```json
{
    "took": 28829,
    "timed_out": false,
    "total": 111396,
    "updated": 0,
    "created": 111396,
    "deleted": 0,
    "batches": 112,
    "version_conflicts": 0,
    "noops": 0,
    "retries": {
        "bulk": 0,
        "search": 0
    },
    "throttled_millis": 0,
    "requests_per_second": -1.0,
    "throttled_until_millis": 0,
    "failures": []
}
```

## 响应身体场

场地| 描述
:--- | :---
拿| 操作花了多长时间。
时间到| 该操作是否定时。
全部的| 处理的文件总数。
更新| 目的地索引中更新的文档数量。
创建| 在目标索引中创建的文档数量。
删除| 删除的文档数量。
批次| 滚动响应的数量。
version_conflicts| 版本冲突的数量。
零| 在操作过程中忽略了多少个文档OpenSearch。
重试| 批量和搜索重试请求的数量。
throttled_millis| 在请求期间，毫秒的毫秒数。
requests_per_second| 操作过程中每秒执行的请求数。
throttled_until_millis| OpenSearch执行下一个节流请求之前的时间。
失败| 操作过程中发生的任何故障。

