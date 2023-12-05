---
layout: default
title: 远程段背压
nav_order: 10
parent: 远程支持存储
grand_parent: 可用性和恢复
---

# 远程段背压

引入2.10
{：.label .label-紫色的 }

远程部分背压是一个碎片-当远程段存储落后于主碎片上的本地承诺段时，会动态拒绝索引请求的级别拒绝机制。使用远程段背压，您可以防止远程存储和本地主商店之间的滞后。滞后可能是由缓慢或失败的远程存储交互，远程存储节流，长垃圾收集停顿或高CPU利用率引起的。

## 阈值

如果违反以下任何一个阈值，则激活远程段背压：

- **连续失败**：如果有_n_或更多连续的失败，则会激活背压。_n_的值可在`remote_store.segment.pressure.consecutive_failures.limit`。
- **字节滞后**：通过添加本地承诺段中存在的所有文件的大小，而不是在远程存储中，可以计算字节滞后。如果字节滞后大于_K_，则激活背压，乘以每次刷新后上传的文件的大小，字节的移动平均值。差异因子_k_在`remote_store.segment.pressure.bytes_lag.variance_factor`。移动窗口大小可通过`remote_store.moving_average_window_size` 环境。
- **时间滞后**：通过比较最新本地刷新和最新远程商店段上传的时间戳来计算时间滞后。如果时间滞后大于_k_乘以每次刷新后上传新段和元数据文件所花费的时间的移动平均值，则将其激活。差异因子_k_可以通过`remote_store.segment.pressure.time_lag.variance_factor` 环境。移动窗口大小可通过`remote_store.moving_average_window_size` 环境。

## 处理细分市场合并

在每个细分市场合并中，都会启动相应的刷新。由于此刷新具有新的合并段，因此字节滞后会立即尖峰。为了补偿此尖峰，仅当远程存储在本地主要商店后面的偏移量后，才能评估字节滞后和时间滞后。但是，连续故障引起的背压都会激活，无论刷新滞后如何（远程存储落后于本地商店的刷新数量）。

## 远程段背压设置

远程段背压将几个设置添加到标准OpenSearch集群设置中。设置是动态的，因此您可以在不重新启动群集的情况下更改默认的背压行为。

下表列出了用于激活背压的设置。有关阈值计算，请参见[阈值](#thresholds)。

|环境|数据类型|描述|
|：---|：---|：---|
|`remote_store.segment.pressure.enabled`|布尔| 如果`true`，启用远程段背压。默认为`false`。|
|`remote_store.segment.pressure.consecutive_failures.limit`|整数|激活远程段背压的最小连续故障计数。默认为`5`。|
|`remote_store.segment.pressure.bytes_lag.variance_factor`|漂浮| 与移动平均线一起使用的方差因子来计算激活远程段背压的动态字节滞后阈值。默认为`10`。|
|`remote_store.segment.pressure.time_lag.variance_factor`|漂浮|与移动平均线一起使用的方差因子来计算激活远程段背压的动态时间滞后阈值。默认为`10`。|

下表列出了用于统计的设置。

|环境|数据类型|描述|
|：---|：---|：---|
| `remote_store.moving_average_window_size` | 整数| 移动平均窗口大小用于计算通过[远程商店统计API]({{site.url}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/remote-store/remote-store-stats-api/)。默认为`20`。最低执行是`5`。|


