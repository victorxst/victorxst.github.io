---
layout: default
title: CAT 节复制
parent: CAT API
nav_order: 53
has_children: false
---

# 猫节复制
**引入2.7**
{: .label .label-purple }

猫节复制操作返回有关活动的信息，最后完成[段复制]({{site.url}}{{site.baseurl}}/opensearch/segment-replication/index) 每个复制碎片上的活动，包括相关碎片-水平指标。这些指标提供了有关复制品落后的主要碎片的信息。

仅在启用段复制的索引上调用CAT段复制API。
{: .note}

## 路径和HTTP方法

```json
GET /_cat/segment_replication
GET /_cat/segment_replication/<index>
```

## 路径参数

下表列出了可用的可选路径参数。

范围| 类型| 描述
:--- | :--- | :---
`index` | 细绳| 索引的名称或逗号-用于过滤结果的索引名称的分离列表或通配符表达式。如果未提供此参数，则响应包含有关集群中所有索引的信息。

## 查询参数

CAT段复制API操作支持以下可选查询参数。

范围| 数据类型| 描述
:--- |:-----------| :---
`active_only` | 布尔| 如果`true`，响应仅包括主动段重复。默认为`false`。
[`detailed`](#additional-detailed-response-metrics) | 细绳| 如果`true`，响应包括段复制事件的每个阶段的其他指标。默认为`false`。
`shards` | 细绳| 逗号-分开显示的碎片列表。
`bytes` | 字节单元| [单位]({{site.url}}{{site.baseurl}}/opensearch/units/) 用于显示字节大小值。
`format` | 细绳| HTTP接受标头的简短版本。有效值包括`JSON` 和`YAML`。
`h` | 细绳| 逗号-要显示的列名称的分开列表。
`help` | 布尔| 如果`true`，响应包括帮助信息。默认为`false`。
`time` | 时间单元| [单位]({{site.url}}{{site.baseurl}}/opensearch/units/) 用于显示时间值。
`v` | 布尔| 如果`true`，响应包括列标题。默认为`false`。
`s` | 细绳| 指定分类结果。例如，`s=shardId:desc` 通过降序进行shardid。

## 例子

以下示例说明了各种段复制响应。

#### 示例1：没有主动段复制事件

以下查询请求将所有索引的列标题段复制指标：

```json
GET /_cat/segment_replication?v=true
```
{% include copy-curl.html %}

响应包含上述请求的指标：

```bash
shardId target_node target_host checkpoints_behind bytes_behind current_lag last_completed_lag rejected_requests
[index-1][0] runTask-1 127.0.0.1 0 0b 0s 7ms 0
```

#### 示例2：指定的碎片ID

以下查询请求段复制指标，带有ID的碎片列标题`0` 来自索引`index1` 和`index2`：

```json
GET /_cat/segment_replication/index1,index2?v=true&shards=0
```
{% include copy-curl.html %}

响应包含前面请求的指标。列标题对应于公制名称：

```bash
shardId target_node target_host checkpoints_behind bytes_behind current_lag last_completed_lag rejected_requests
[index-1][0] runTask-1 127.0.0.1 0 0b 0s 3ms 0
[index-2][0] runTask-1 127.0.0.1 0 0b 0s 5ms 0
```

#### 示例3：详细响应

以下查询请求详细的段复制指标，所有索引的列标题：

```json
GET /_cat/segment_replication?v=true&detailed=true
```
{% include copy-curl.html %}

响应包含有关段复制事件的文件和阶段的其他指标：

```bash
shardId target_node target_host checkpoints_behind bytes_behind current_lag last_completed_lag rejected_requests stage time files_fetched files_percent bytes_fetched bytes_percent start_time stop_time files files_total bytes bytes_total replicating_stage_time_taken get_checkpoint_info_stage_time_taken file_diff_stage_time_taken get_files_stage_time_taken finalize_replication_stage_time_taken
[index-1][0] runTask-1 127.0.0.1 0 0b 0s 3ms 0 done 10ms 6 100.0% 4753 100.0% 2023-03-16T13:46:16.802Z 2023-03-16T13:46:16.812Z 6 6 4.6kb 4.6kb 0s 2ms 0s 3ms 3ms
[index-2][0] runTask-1 127.0.0.1 0 0b 0s 5ms 0 done 7ms 3 100.0% 3664 100.0% 2023-03-16T13:53:33.466Z 2023-03-16T13:53:33.474Z 3 3 3.5kb 3.5kb 0s 1ms 0s 2ms 2ms
```

#### 示例4：对结果进行排序

以下查询请求段复制指标，所有索引的列标题，按shard ID按降序排序：

```json
GET /_cat/segment_replication?v&s=shardId:desc
```
{% include copy-curl.html %}

响应包含分类结果：

```bash
shardId    target_node  target_host checkpoints_behind bytes_behind current_lag last_completed_lag rejected_requests
[test6][1] runTask-2   127.0.0.1   0                  0b           0s          5ms                0
[test6][0] runTask-2   127.0.0.1   0                  0b           0s          4ms                0
```

#### 示例5：使用度量别名

在请求中，您可以使用度量标准的全名或其别名之一。以下查询与前面的查询相同，但使用别名`s` 代替`shardID` 用于排序：

```json
GET /_cat/segment_replication?v&s=s:desc
```
{% include copy-curl.html %}

## 响应指标

下表列出了所有请求返回的响应指标。在查询参数中指的指标时，您可以提供指标的全名或其任何别名，如上所述[例子](#example-5-using-a-metric-alias)。

公制| 别名| 描述
:--- | :--- | :---
`shardId` | `s` | 特定碎片的ID。
`target_host` | `thost` | 目标主机IP地址。
`target_node` | `tnode` | 目标节点名称。
`checkpoints_behind` | `cpb` | 复制碎片在主要碎片后面的检查点的数量。
`bytes_behind` | `bb` | 复制碎片在主要碎片后面的字节数。
`current_lag` | `clag` | 等待复制碎片赶上主要碎片时的时间。
`last_completed_lag` | `lcl` | 复制碎片所花费的时间赶上了最新的主要碎片刷新。
`rejected_requests` | `rr` | 复制组的拒绝请求数量。

### 其他详细响应指标

下表列出了返回的其他响应字段，如果`detailed` 被设定为`true`。

公制| 别名| 描述
:--- |:--- |:---
`stage` | `st` | 段复制事件的当前阶段。
`time` | `t`，`ti` | 分段复制事件以毫秒为单位完成的时间。
`files_fetched` | `ff` | 到目前为止，用于段复制事件的文件数量。
`files_percent` | `fp` | 到目前为止，用于段复制事件的文件百分比。
`bytes_fetched` | `bf` | 到目前为止，用于段复制事件的字节数。
`bytes_percent` | `bp` | 到目前为止，以段重复事件的百分比获取字节数。
`start_time` | `start` | 段复制开始时间。
`stop_time` | `stop` | 段复制停止时间。
`files` | `f` | 需要获取段复制事件的文件数量。
`files_total` | `tf` | 是此恢复的一部分的文件总数，包括重复使用和恢复的文件。
`bytes` | `b` | 需要获取细分复制事件的字节数。
`bytes_total` | `tb` | 碎片中的字节总数。
`replicating_stage_time_taken` | `rstt` | 时间的时间`replicating` 段重复事件的阶段完成。
`get_checkpoint_info_stage_time_taken` | `gcistt` | 时间的时间`get checkpoint info` 段重复事件的阶段完成。
`file_diff_stage_time_taken` | `fdstt` | 时间的时间`file diff` 段重复事件的阶段完成。
`get_files_stage_time_taken` | `gfstt` | 时间的时间`get files` 段重复事件的阶段完成。
`finalize_replication_stage_time_taken` | `frstt` | 时间的时间`finalize replication` 段重复事件的阶段完成。

