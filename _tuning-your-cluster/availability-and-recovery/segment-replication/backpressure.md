---
layout: default
title: 段复制背压
nav_order: 75
parent: 段复制
has_children: false
grand_parent: 可用性和恢复
---

# 段复制背压

段复制背压是一个碎片-级别拒绝机制动态拒绝索引请求作为群集中的复制碎片，落在主要碎片后面。通过细分复制背压，当复制组中的陈旧碎片百分比超过`segrep.pressure.replica.stale.limit` （默认情况下为50％）。如果复制品在主要碎片后面，则将其视为过时的检查点的数量`segrep.pressure.checkpoint.limit` 设置及其当前复制滞后大于定义的`segrep.pressure.time.limit` 场地。

还监视了副本碎片，以确定碎片是长时间内被卡住还是滞后。当副本碎片被卡住或滞后的时间超过两倍以上`segrep.pressure.time.limit` 田地，将碎片拆除并用新的副本碎片代替。

## 请求字段

段重复背压默认情况下是禁用的。为了启用，设置`segrep.pressure.enabled` 到`true`。您可以使用以下动态群集设置来使用[集群设置]({{site.url}}{{site.baseurl}}/api-reference/cluster-api/cluster-settings/) API端点。

场地| 数据类型| 描述
：--- | ：--- | ：---
`segrep.pressure.enabled `| 布尔| 启用细分复制背压机制。默认为`false`。
`segrep.pressure.time.limit` | 时间单元| 复制碎片可以从主碎片中复制的最长时间。一次`segrep.pressure.time.limit` 被违反`segrep.pressure.checkpoint.limit`，启动片段复制背压机制。默认为`5 minutes`。
`segrep.pressure.checkpoint.limit` | 整数| 从主复制时，复制碎片的最大索引检查点可能会落后。一次`segrep.pressure.checkpoint.limit` 被违反`segrep.pressure.time.limit`，启动片段复制背压机制。默认为`4` 检查点。
`segrep.pressure.replica.stale.limit `| 浮点| 复制组中可能存在的最大陈旧复制碎片数。一次`segrep.pressure.replica.stale.limit` 违反，启动了片段复制背压机制。默认为`.5`，是复制组的50％。

## 路径和HTTP方法

您可以使用段复制API端点来检索段复制背压指标，如下所示：

```bash
GET _cat/segment_replication
```
{％包含副本-curl.html％}

#### 示例响应

```json
shardId       target_node    target_host   checkpoints_behind bytes_behind   current_lag   last_completed_lag   rejected_requests
[index-1][0]     runTask-1    127.0.0.1              0              0b           0s              7ms                    0
```

这`checkpoints_behind` 和`current_lag` 在启动片段复制背压时，请考虑度量。他们被检查`segrep.pressure.checkpoint.limit` 和`segrep.pressure.time.limit`， 分别。

