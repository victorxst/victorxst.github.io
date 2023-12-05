---
layout: default
title: 设置
parent: 异常检测
nav_order: 4
redirect_from: 
  - /monitoring-plugins/ad/settings/
---

# 异常检测设置

异常检测插件在标准OpenSearch集群设置中添加了多个设置。
设置是动态的，因此您可以更改插件的默认行为，而无需重新启动群集。要了解有关静态和动态设置的更多信息，请参阅[配置OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)。

您可以将设置标记为`persistent` 或者`transient`。

例如，更新结果索引的保留期：

```json
PUT _cluster/settings
{
  "transient": {
    "plugins.anomaly_detection.ad_result_history_retention_period": "5m"
  }
}
```

环境| 默认| 描述
:--- | :--- | :---
plugins.anomaly_detection.enabled| 真的| 是否启用了异常检测插件。如果禁用，所有检测器都会立即停止运行。
plugins.anomaly_detection.max_anomaly_detectors| 1,000| 最大非数量-高基数检测器（没有类别字段）用户可以创建。
plugins.anomaly_detection.max_multi_entity_anomaly_detectors| 10| 群集中最大的高基数检测器数（带有类别字段）。
plugins.anomaly_detection.max_anomaly_features| 5| 检测器的最大功能数量。
plugins.anomaly_detection.ad_result_history_rollover_period| 12H| 一次性转盘条件多久检查一次。如果`true`，异常检测插件将结果索引滚动到新索引。
plugins.anomaly_detection.ad_result_history_max_docs_per_shard| 1,350,000,000| 结果索引的单个碎片中的最大文档数量。异常检测插件仅计入主碎片中的刷新文档。
plugins.anomaly_detection.max_entities_per_query| 1,000,000| 高基数检测器的每个检测间隔的最大唯一值。默认情况下，如果类别字段比检测器间隔中配置的唯一值多，则通过自然订购分类值的异常检测插件（例如，实体）订购它们`ab` 来了`bc`），然后选择最高值。
plugins.anomaly_detection.max_entities_for_preview| 5| 高基数检测器的预览操作显示的最大唯一类别字段值。默认情况下，如果类别字段比检测器间隔中配置的唯一值多，则通过自然订购分类值的异常检测插件（例如，实体）订购它们`ab` 来了`bc`），然后选择最高值。
plugins.anomaly_detection.max_primary_shards| 10| 一个异常检测指数的最大主要碎片数量。
plugins.anomaly_detection.filter_by_backend_roles| 错误的| 当您启用安全插件并将其设置为`true`，根据用户的后端角色过滤异常检测插件的过滤结果。
plugins.anomaly_detection.max_batch_task_per_node| 10| 开始历史分析会触发批处理任务。此设置是您每个数据节点可以运行的批处理任务数量。您可以将此设置从1到1,000调整。如果数据节点无法支持所有批处理任务，并且您不确定数据节点是否能够运行更多的历史分析，请添加更多数据节点，而不是将此设置更改为更高的值。增加此值可能会为每个数据节点带来更多负载。
plugins.anomaly_detection.max_old_ad_task_docs_per_detector| 1| 您可以多次对同一检测器进行历史分析。对于每次运行，异常检测插件都会创建一个新任务。此设置是插件保留的先前任务的数量。将此值设置为至少1个以跟踪其上一次运行。您最多可以保留1,000个旧任务，以避免压倒群集。
plugins.anomaly_detection.batch_task_piece_size| 1,000| 历史任务的日期范围分为较小的零件，而异常检测插件则按件运行任务。默认情况下，每件都包含1,000个检测间隔。例如，如果检测器间隔为1分钟，一件是1,000分钟，则每1,000分钟查询功能数据。您可以将此设置从1更改为10,000。
plugins.anomaly_detection.batch_task_piece_interval_seconds| 5| 在两个相同的历史分析任务之间添加时间间隔。此间隔阻止任务消耗过多的可用资源，并饿死其他操作，例如搜索和批量索引。您可以将此设置从1到600秒更改。
plugins.anomaly_detection.max_top_entities_for_historical_analysis| 1,000| 您为高基数检测器历史分析而运行的最大最大实体数量。范围从1到10,000。
plugins.anomaly_detection.max_running_entities_per_detector_for_historical_analysis| 10| 您可以并行运行的实体任务数量，以进行高基数检测器分析。群集上可用的任务插槽还影响了多少实体并行运行。如果集群具有3个数据节点，则每个数据节点默认情况下具有10个任务插槽。假设您已经有两个高基数检测器，并且每个探测器都运行10个实体。如果您开始一个-采用1个任务插槽的实体检测器，可用的任务插槽数为10 * 3- 10 * 2- 1 = 9.如果您现在启动新的高基数检测器，则检测器只能并行运行9个实体，而不是10个实体。您可以根据群集的能力从1到1,000来调整此值。如果设置更高的值，则异常检测插件将更快地运行历史分析，但也消耗了更多的资源。
plugins.anomaly_detection.max_cached_deleted_tasks| 1,000| 您可以根据需要重新重新进行单个检测器的历史分析。默认情况下，Anomaly检测插件仅保持有限数量的旧任务。如果您对检测器进行三次历史分析，则删除了最古老的任务。由于历史分析会产生许多异常会导致时间较短，因此有必要清理已删除任务的异常。在此字段中，您最多可以配置最多可以缓存的删除任务。删除任务后，该插件会清理任务结果。如果插件无法进行此清理，则将任务的结果添加到缓存中，并且每小时的CRON作业执行清理工作。您可以使用此设置来限制在缓存中输入多少旧任务，以避免DDOS攻击。一个小时后，如果仍然在缓存中找到旧任务结果，请使用[删除检测器结果API]({{site.url}}{{site.baseurl}}/monitoring-plugins/ad/api/#delete-detector-results) 手动删除任务结果。您可以将此设置从1到10,000。
plugins.anomaly_detection.delete_anomaly_result_when_delete_detector| 错误的| 删除检测器时，异常检测插件是否会删除异常结果。如果您想节省一些磁盘空间，尤其是如果您的高基数检测器生成大量结果，请将此字段设置为true。或者，您可以使用[删除检测器结果API]({{site.url}}{{site.baseurl}}/monitoring-plugins/ad/api/#delete-detector-results) 手动删除结果。
plugins.anomaly_detection.dedicated_cache_size| 10| 如果是真实的-高基数检测器的时间分析成功地开始，异常检测插件可确保在每个节点内存中保持10（通过此设置动态调节）实体模型。如果实体数量超过此限制，则插件将额外的实体模型放在所有检测器共享的内存空间中。实体的实际数量根据您可用的内存和实体的频率而变化。如果您希望插件保证将更多实体的模型保留在内存中，并且群集具有足够的内存，则可以增加此设置值。
plugins.anomaly_detection.max_concurrent_preview| 2| 并发预览的最大数量。您可以使用此设置来限制资源使用情况。
plugins.anomaly_detection.model_max_size_percent| 0.1| 模型的内存百分比的上限。
plugins.anomaly_detection.door_keeper_in_cache.enabled| 错误的| 设置为`true`，OpenSearch将Bloom过滤器放在非活动实体缓存的前面，以滤除不太可能出现一次以上的项目。
plugins.anomaly_detection.hcad_cold_start_interpolation.enabled| 错误的| 设置为`true`，启用插值高-基数异常检测（HCAD）冷启动

