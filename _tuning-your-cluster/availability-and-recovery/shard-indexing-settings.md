---
layout: default
title: 设置
parent: 碎片索引背压
nav_order: 50
grand_parent: 可用性和恢复
redirect_from: 
  - /opensearch/shard-indexing-settings/
---

# 设置

碎片索引背压将标准OpenSearch集群设置添加了几个设置。它们是动态的，因此您可以在不重新启动群集的情况下更改此功能的默认行为。

## 高的-级别控件

高-级别控件允许您打开或关闭碎片索引索引索引功能。

环境| 默认| 描述
:--- | :--- | :---
`shard_indexing_pressure.enabled` | 错误的| 改成`true` 启用碎片索引背压。
`shard_indexing_pressure.enforced` | 错误的| 在阴影模式或强制模式下运行碎片索引背压。在影子模式中（值集为`false`），碎片索引背压轨迹所有颗粒状-水平指标，但实际上并未拒绝任何索引请求。在强制模式下（值集为`true`），碎片索引背压拒绝对群集的任何请求，这可能会导致其性能下降。

## 节点-级别限制

节点-级别限制允许您控制节点上的内存使用率。

环境| 默认| 描述
:--- | :--- | :---
`shard_indexing_pressure.primary_parameter.node.soft_limit` | 70％| 定义节点的百分比-水平的存储阈值充当节点上应变的软指标。

## 碎片-级别限制

碎片-级别限制使您可以控制碎片上的内存使用量。

环境| 默认| 描述
:--- | :--- | :---
`shard_indexing_pressure.primary_parameter.shard.min_limit` | 0.001d| 在任何角色（协调员，主或复制品）中指定新碎片的最小配额。碎片索引背压会根据碎片流量的流入而增加或减少该分配的配额。
`shard_indexing_pressure.operating_factor.lower` | 75％| 指定碎片的内存分配配额的较低占用限制。如果碎片的总内存使用量低于此限制，则碎片索引背压会降低该碎片的当前分配内存。
`shard_indexing_pressure.operating_factor.optimal` | 85％| 指定碎片的内存分配配额的最佳占用。如果碎片的总内存使用量在此级别，则碎片索引背压不会改变该碎片的当前分配内存。
`shard_indexing_pressure.operating_factor.upper` | 95％| 指定碎片的内存分配配额的上限限制。如果碎片的总内存使用率高于此极限，则碎片索引背压会增加该碎片的当前分配内存。

## 性能降解因素

性能降解因子使您可以控制碎片的动态性能阈值。

环境| 默认| 描述
:--- | :--- | :---
`shard_indexing_pressure.secondary_parameter.throughput.request_size_window` | 2,000| 碎片上的采样窗口大小中的请求数。碎片索引背压将请求的整体性能与示例窗口中的请求进行比较，以检测任何性能降解。
`shard_indexing_pressure.secondary_parameter.throughput.degradation_factor` | 5倍| 请求的每单位字节降解因子。此参数确定任何延迟峰值的阈值。默认值为5倍，这意味着，如果延迟在历史悠久的视图中射出了5次，则shard索引背压将其标记为性能退化。
`shard_indexing_pressure.secondary_parameter.successful_request.elapsed_timeout` | 300000毫秒| 请求在集群中待定的时间。此参数有助于识别任何卡住-请求方案。
`shard_indexing_pressure.secondary_parameter.successful_request.max_outstanding_requests` | 100| 集群中最大的待处理请求数。

