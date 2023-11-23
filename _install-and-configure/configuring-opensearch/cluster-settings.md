---
layout: default
title: 集群设置
parent: 配置 OpenSearch
nav_order: 60
---

# 群集设置

以下设置与 OpenSearch 集群相关。

要了解有关静态和动态设置的详细信息，请参阅[配置 OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)。

## 集群级路由和分配设置

OpenSearch 支持以下集群级路由和分片分配设置。此列表中的所有设置都是动态的：

-  `cluster.routing.allocation.enable`（字符串）：启用或禁用特定类型分片的分配。
    
    有效值为：
     -  `all` –允许为所有类型的分片分配分片。
     -  `primaries` –仅允许为主分片分配分片。
     -  `new_primaries` –仅允许为新索引的主分片分配分片。
     -  `none` –不允许为任何索引分配分片。
     
     缺省值为 `all`。

-  `cluster.routing.allocation.node_concurrent_incoming_recoveries`（整数）：配置一个节点上允许发生的并发传入分片恢复数。缺省值为 `2`。

-  `cluster.routing.allocation.node_concurrent_outgoing_recoveries`（整数）：配置允许在节点上进行多少个并发传出分片恢复。缺省值为 `2`。

-  `cluster.routing.allocation.node_concurrent_recoveries`（String）：用于将和 `cluster.routing.allocation.node_concurrent_outgoing_recoveries` 设置为 `cluster.routing.allocation.node_concurrent_incoming_recoveries` 相同的值。

-  `cluster.routing.allocation.node_initial_primaries_recoveries`（整数）：设置节点重新启动后未分配主节点的恢复次数。缺省值为 `4`。

-  `cluster.routing.allocation.same_shard.host`（布尔值）：设置为 `true` 时，将阻止将分片的多个副本分配给同一主机上的不同节点。缺省值为 `false`。

-  `cluster.routing.rebalance.enable`（字符串）：启用或禁用特定类型分片的重新平衡。
    
    有效值为：
     -  `all` –允许所有类型的分片的分片平衡。
     -  `primaries` –仅允许主分片的分片平衡。
     -  `replicas` –仅允许副本分片的分片平衡。
     -  `none` –不允许对任何索引进行分片平衡。

     缺省值为 `all`。

-   `cluster.routing.allocation.allow_rebalance`（String）：指定何时允许分片重新平衡。
    
    有效值为：
    -   `always` –始终允许重新平衡。
    -  `indices_primaries_active` –仅当集群中的所有主节点都已分配时，才允许重新平衡。
    -  `indices_all_active` –仅当集群中的所有分片都已分配时，才允许重新平衡。

    缺省值为 `indices_all_active`。

-  `cluster.routing.allocation.cluster_concurrent_rebalance`（整数）：允许你控制集群中允许的并发分片重新平衡数。缺省值为 `2`。

-  `cluster.routing.allocation.balance.shard`（浮点）：定义每个节点分配的分片总数的权重因子。缺省值为 `0.45`。

-  `cluster.routing.allocation.balance.index`（浮点）：定义节点上分配的每个索引的分片数的权重因子。缺省值为 `0.55`。

-  `cluster.routing.allocation.balance.threshold`（浮点）：应执行的操作的最小优化值。缺省值为 `1.0`。

-  `cluster.routing.allocation.balance.prefer_primary`（Boolean）：设置为 `true` 时，OpenSearch 会尝试在集群节点之间均匀分配主分片。启用此设置并不总是保证每个节点上的主分片数量相等，尤其是在故障转移的情况下。将此设置 `false` 更改为之后 `true`，不会调用主分片的重新分发。缺省值为 `false`。

-  `cluster.routing.allocation.disk.threshold_enabled`（Boolean）：设置为 `false` 时，禁用磁盘分配决策程序。这也将在禁用时删除任何现有 `index.blocks.read_only_allow_delete index blocks` 内容。缺省值为 `true`。

-  `cluster.routing.allocation.disk.watermark.low`（String）：控制磁盘使用率的低水位线。设置为百分比时，OpenSearch 不会将分片分配给已使用磁盘百分比的节点。这也可以输入为比率值，如 `0.85`.最后，也可以将其设置为字节值，例如 `400mb`.此设置不会影响新创建的索引的主分片，但会阻止分配其副本。缺省值为 `85%`。

-  `cluster.routing.allocation.disk.watermark.high`（String）：控制高水位线。OpenSearch 将尝试将分片从磁盘使用率高于定义的百分比的节点上移开。这也可以输入为比率值，例如 `0.85`。最后，也可以将其设置为字节值，例如 `400mb`.此设置会影响所有分片的分配。缺省值为 `90%`。

-  `cluster.routing.allocation.disk.watermark.flood_stage`（String）：控制洪水阶段水印。这是防止节点磁盘空间不足的最后手段。OpenSearch 对节点上分配了一个或多个分区且至少有一个磁盘超过泛滥阶段的每个索引强制执行只读索引块（ `index.blocks.read_only_allow_delete`）。一旦磁盘利用率低于高水位线，索引块就会被释放。这也可以输入为比率值，例如 `0.85`。最后，也可以将其设置为字节值，例如 `400mb`.缺省值为 `95%`。

-  `cluster.info.update.interval`（时间单位）：设置 OpenSearch 检查集群中每个节点的磁盘使用情况的频率。缺省值为 `30s`。

-  `cluster.routing.allocation.include.<attribute>`（枚举）：将分片分配给至少具有一个包含逗号分隔值的节点 `attribute`。

-  `cluster.routing.allocation.require.<attribute>`（枚举）：仅将分片分配给具有所有包含逗号分隔值的节点 `attribute`。

-  `cluster.routing.allocation.exclude.<attribute>`（枚举）：不将分片分配给具有任何包含逗号分隔值的节点 `attribute`。群集分配设置支持以下内置属性。
    
    有效值为：
    -  `_name` –按节点名称匹配节点。
    -  `_host_ip` –按主机 IP 地址匹配节点。
    -  `_publish_ip` –按发布 IP 地址匹配节点。
    -  `_ip` –匹配 `_host_ip` 或 `_publish_ip`.
    -  `_host` –按主机名匹配节点。
    -  `_id` –按节点 ID 匹配节点。
    -  `_tier` –按数据层角色匹配节点。

-  `cluster.routing.allocation.shard_movement_strategy`（枚举）：确定分片从传出节点重新定位到传入节点的顺序。

    此设置支持以下策略：
    -  `PRIMARY_FIRST` –先重新定位主分片，然后再重新定位副本分片。此优先级可能有助于防止在重新定位节点在此过程中失败时群集的运行状况变为红色。
    -  `REPLICA_FIRST` –先重新定位副本分片，然后再重新定位主分片。此优先级可能有助于防止在启用了分段复制的混合版本的 OpenSearch 集群中执行分片重定位时集群的运行状况变为红色。在这种情况下，重新定位到较新版本的 OpenSearch 节点的主分片可能会尝试将分段文件复制到较旧版本的 OpenSearch 上的副本分片，这将导致分片失败。首先重新定位副本分片可能有助于避免在多版本集群中出现这种情况。
    -  `NO_PREFERENCE` –分片重定位顺序不重要的默认行为。

## 集群级分片、块和任务设置

OpenSearch 支持以下集群级分片、块和任务设置：

-  `cluster.blocks.read_only`（布尔值）：将整个群集设置为只读。缺省值为 `false`。

-  `cluster.blocks.read_only_allow_delete`（Boolean）：类似于 `cluster.blocks.read_only`，但允许你删除索引。

-  `cluster.max_shards_per_node`（整数）：限制集群的主分片和副本分片总数。限制的计算方式如下： `cluster.max_shards_per_node` 乘以非冻结数据节点的数量。已关闭索引的分片不计入此限制。缺省值为 `1000`。

-  `cluster.persistent_tasks.allocation.enable`（字符串）：启用或禁用持久性任务的分配。

    有效值为：
    -  `all` –允许将持久性任务分配给节点。
    -  `none` –不允许为持久性任务分配。这不会影响已在运行的持久性任务。

    缺省值为 `all`。

-  `cluster.persistent_tasks.allocation.recheck_interval`（时间单位）：当集群状态发生重大变化时，集群管理器会自动检查是否需要分配持久性任务。还有其他因素（如内存使用情况）会影响是否将持久性任务分配给节点，但不会导致群集状态更改。此设置定义为响应这些因素而执行分配检查的频率。默认值为 `30 seconds`，最小 `10 seconds` 值为必需。

## 集群级索引设置

有关索引级索引设置的信息，请参见[集群级索引设置]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index-settings/#cluster-level-index-settings)。