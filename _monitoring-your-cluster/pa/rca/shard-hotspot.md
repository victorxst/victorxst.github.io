---
layout: default
title: 热碎片识别
parent: 根本原因分析
grand_parent: 性能分析仪
nav_order: 30
---

# 热碎片识别

热碎片识别根本原因分析（RCA）可让您在索引中识别热碎片。热碎片是一个比其他碎片更多的不同资源的异常值，并且可能导致索引和搜索性能差。Hot Shard识别RCA监视以下指标：

- CPU利用率
- 堆分配率

由于工作量的性质，碎片可能会变得热。当您使用`_routing` 参数或自定义文档ID，特定的碎片或群集中的几片接收到频繁的更新，比其他碎片要消耗更多的CPU和堆资源。

热碎片识别RCA将CPU利用率和堆分配速率与其阈值值进行比较。如果任一个指标的用法大于阈值，则将碎片视为_hot_。

有关Hot Shard标识RCA实施的更多信息，请参见[热shard rca](https://github.com/opensearch-project/performance-analyzer-rca/blob/main/src/main/java/org/opensearch/performanceanalyzer/rca/store/rca/hotshard/docs/README.md)。

#### 示例请求

以下查询请求热碎片标识：

```bash
GET _plugins/_performanceanalyzer/rca?name=HotShardClusterRca
```
{% include copy-curl.html %}

#### 示例响应

响应包含不健康的碎片清单：

```json
"HotShardClusterRca": [{
  "rca_name": "HotShardClusterRca",
  "timestamp": 1680721367563,
  "state": "unhealthy",
  "HotClusterSummary": [
    {
      "number_of_nodes": 3,
      "number_of_unhealthy_nodes": 1,
      "HotNodeSummary": [
        {
          "node_id": "7kosAbpASsqBoHmHkVXxmw",
          "host_address": "192.168.80.4",
          "HotResourceSummary": [
            {
              "resource_type": "cpu usage",
              "resource_metric": "cpu usage(num of cores)",
              "threshold": 0.027397981341796683,
              "value": 0.034449630200405396,
              "time_period_seconds": 60,
              "meta_data": "ssZw1WRUSHS5DZCW73BOJQ index9 4"
            },
            {
              "resource_type": "heap",
              "resource_metric": "heap alloc rate(heap alloc rate in bytes per second)",
              "threshold": 7605441.367010161,
              "value": 10872119.748328414,
              "time_period_seconds": 60,
              "meta_data": "ssZw1WRUSHS5DZCW73BOJQ index9 4"
            },
            {
              "resource_type": "heap",
              "resource_metric": "heap alloc rate(heap alloc rate in bytes per second)",
              "threshold": 7605441.367010161,
              "value": 8019622.354388569,
              "time_period_seconds": 60,
              "meta_data": "QRF4rBM7SNCDr1g3KU6HyA index9 0"
            }
          ]
        }
      ]
    }
  ]
}]
```

## 响应字段

下表列出了响应字段。

场地| 类型| 描述
:--- | :--- | :---
rca_name| 细绳| RCA的名称。在这种情况下，"HotShardClusterRca"。
时间戳| 整数| RCA的时间戳。
状态| 目的| 由RCA确定的群集的状态。这`state` 可`healthy`，`unhealthy`， 或者`unknown`。
hotclustersummary.hotnodesummary.number_of_nodes| 整数| 集群中的节点数量。
hotclustersummary.hotnodesummary.number_of_unhealthy_nodes| 整数| 发现节点的数量在`unhealthy` 状态。
hotclustersummary.hotnodesummary.hotresourcesummary.resource_type| 目的| 引起不健康状态的资源类型"cpu usage" 或者"heap"。
hotclustersummary.hotnodesummary.hotresourcesummary.resource_metric| 细绳| resource_type的定义。任何一个"cpu usage(num of cores)" 或者"heap alloc rate(heap alloc rate in bytes per second)"。
hotclustersummary.hotnodesummary.hotresourcesummary.threshold| 漂浮| 确定是否争夺资源的值。
hotclustersummary.hotnodesummary.hotresourcesummary.value| 漂浮| 资源的当前值。
hotclustersummary.hotnodesummary.hotresourcesummary.time_period_seconds| 时间| 在宣布其状态之前对碎片进行监测的时间。
hotclustersummary.hotnodesummary.hotresourcesummary.meta_data| 细绳| 与resource_type关联的元数据。

在上一个示例响应中，`meta_data` 是`QRF4rBM7SNCDr1g3KU6HyA index9 0`。这`meta_data` 字符串由三个字段组成：

- 节点名称：`QRF4rBM7SNCDr1g3KU6HyA`
- 索引名称：`index9`
- 碎片ID：`0`

这意味着碎片`0` 索引`index9` 在节点上`QRF4rBM7SNCDr1g3KU6HyA` 火爆

