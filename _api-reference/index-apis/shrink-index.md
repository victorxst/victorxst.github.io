---
layout: default
title: 收缩索引
parent: 索引API
nav_order: 65
redirect_from:
  - /opensearch/rest-api/index-apis/shrink-index/
---

# 收缩索引
**引入1.0**
{: .label .label-purple }

收缩索引API操作将现有索引中的所有数据移动到一个新的索引中，主碎片较少。

## 例子

```json
POST /my-old-index/_shrink/my-new-index
{
  "settings": {
    "index.number_of_replicas": 4,
    "index.number_of_shards": 3
  },
  "aliases":{
    "new-index-alias": {}
  }
}
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
POST /<index-name>/_shrink/<target-index>
PUT /<index-name>/_shrink/<target-index>
```

使用此操作创建新索引时，请记住，OpenSearch索引具有以下命名限制：

- 所有字母必须是小写。
- 索引名称不能以下划线开头（`_`）或连字符（`-`）。
- 索引名称不能包含空格，逗号或以下字符：

  `:`，`"`，`*`，`+`，`/`，`\`，`|`，`?`，`#`，`>`， 或者`<`

## URL参数

收缩索引API操作要求您指定源索引和目标索引。所有其他参数都是可选的。

范围| 类型| 描述
:--- | :--- | :---
＆lt;索引-名称＆gt;| 细绳| 收缩索引。
＆lt;目标-索引＆gt;| 细绳| 缩小源指数的目标索引。
wait_for_active_shards| 细绳| 指定在OpenSearch处理请求之前必须可用的活动碎片数。默认值为1（仅是主要碎片）。设置为全部或正整数。大于1的值需要复制品。例如，如果指定一个值为3的值，则该索引必须在两个其他节点上分布两个副本才能成功。
cluster_manager_timeout| 时间| 等待连接到群集管理器节点多长时间。默认为`30s`。
暂停| 时间| 等待请求返回响应多长时间。默认为`30s`。
wait_for_completion| 布尔| 设置为`false`，请求立即返回操作完成后。要监视操作状态，请使用[任务API]({{site.url}}{{site.baseurl}}/api-reference/tasks/) 随着请求返回任务ID。默认为`true`。
task_execution_timeout| 时间| 明确的任务执行超时。仅当将WAIT_FOR_COMPLETION设置为`false`。默认为`1h`。

## 请求身体

您可以使用请求主体为目标索引配置一些索引设置。所有字段都是可选的。

场地| 类型| 描述
:--- | :--- | :---
别名| 目的| 设定目标索引的别名。可以有字段`filter`，`index_routing`，`is_hidden`，`is_write_index`，`routing`， 或者`search_routing`。看[索引别名]({{site.url}}{{site.baseurl}}/api-reference/alias/#request-body)。
设置| 目的| 索引设置您可以应用于目标索引。看[索引设置]({{site.url}}{{site.baseurl}}/im-plugin/index-settings/)。
[max_shard_size](#the-max_shard_size-parameter) | 字节| 指定目标索引中主碎片的最大尺寸。因为`max_shard_size` 与`index.number_of_shards` 设置，您不能同时设置两个设置。

### 这`max_shard_size` 范围

这`max_shard_size` 参数指定目标索引中主碎片的最大大小。OpenSearch使用`max_shard_size` 以及源指数中所有主要碎片的总存储，以计算主要碎片数量及其大小的目标索引。

目标索引的主要分片计数是源指数的主要碎片计数的最小因素，该碎片尺寸不超过`max_shard_size`。例如，如果源索引有8个主要碎片，它们总共占有400 GB的存储空间，并且`max_shard_size` 等于150 GB，OpenSearch使用以下算法计算目标索引中的主要碎片数：

1. 将最小碎片数量计算为400/150，舍入到最接近的整个整数。主要碎片的最小数量为3。
1. 计算主要碎片的数量为最小的8个大于3的因子。主碎片的数量为4。

目标索引的最大碎片数量等于源指数中的主要碎片数量，因为收缩操作用于减少主碎片计数。例如，考虑一个带有5个主要碎片的源索引，总计600 GB的存储空间。如果`max_shard_size` 为100 GB，主要碎片的最小数量为600/100，即6。但是，由于源索引中的主碎片数小于6，因此目标索引中的主碎片的数量设置为5。

目标指数的主要碎片的最小数量为1。
{:.note}

## 索引编解码器注意事项

有关索引编解码器的注意事项，请参阅[索引编解码器]({{site.url}}{{site.baseurl}}/im-plugin/index-codecs/#splits-and-shrinks)

