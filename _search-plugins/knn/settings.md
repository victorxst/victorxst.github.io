---
layout: default
title: 设置
parent: k-NN
nav_order: 40
---

# k-NN设置

k-NN插件添加了几个新的群集设置。要了解有关静态和动态设置的更多信息，请参阅[配置OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)。

## 集群设置

环境| 默认| 描述
:--- | :--- | :---
`knn.algo_param.index_thread_qty` | 1| 用于本机库索引创建的线程数。保持该值低减少K的CPU影响-NN插件，但也降低了索引性能。
`knn.cache.item.expiry.enabled` | 错误的| 是否要删除内存一定持续时间尚未访问的本地库索引。
`knn.cache.item.expiry.minutes` | 3H| 如果启用，请从内存中删除本地库索引之前的空闲时间。
`knn.circuit_breaker.unset.percentage` | 75％| 断路器的本地内存使用阈值。内存使用必须低于此百分比`knn.memory.circuit_breaker.limit` 为了`knn.circuit_breaker.triggered` 保持虚假。
`knn.circuit_breaker.triggered` | 错误的| 当内存使用超过`knn.circuit_breaker.unset.percentage` 价值。
`knn.memory.circuit_breaker.limit` | 50％| 本机库索引的本地内存限制。在默认值中，如果机器具有100 GB的内存，并且JVM使用32 GB，则K-NN插件使用其余68 GB（34 GB）的50％。如果内存使用率超过此值，则-NN删除了最近使用的本机库索引。
`knn.memory.circuit_breaker.enabled` | 真的| 是否启用K-NN内存断路器。
`knn.plugin.enabled`| 真的| 启用或禁用K-NN插件。
`knn.model.index.number_of_shards`| 1| 用于模型系统索引的碎片数量，即存储用于大约最近邻居（ANN）搜索的模型的OpenSearch索引。
`knn.model.index.number_of_replicas`| 1| 用于模型系统索引的复制碎片数量。通常，在多个-节点群集，这应该至少1以提高稳定性。
`knn.advanced.filtered_exact_search_threshold`| 无效的| 在过滤ANN搜索过程中，用于切换到精确搜索的过滤ID的阈值值。如果段中过滤ID的数量小于此设置的值，则将在过滤后的ID上执行精确的搜索。

