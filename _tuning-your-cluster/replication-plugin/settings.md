---
layout: default
title: 复制设置
nav_order: 40
parent: 跨集群复制
redirect_from:
  - /replication-plugin/settings/
---

# 复制设置

复制插件将几个设置添加到标准OpenSearch集群设置中。
设置是动态的，因此您可以更改插件的默认行为，而无需重新启动群集。要了解有关静态和动态设置的更多信息，请参阅[配置OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)。

您可以将设置标记为`persistent` 或者`transient`。

例如，更新追随者群集的频率对领导者群集进行更新的频率：

```json
PUT _cluster/settings
{
  "persistent": {
    "plugins.replication.follower.metadata_sync_interval": "30s"
  }
}
```

这些设置管理远程回收消耗的资源。我们不建议更改这些设置；在大多数用例中，默认值应该很好。

环境| 默认| 描述
:--- | :--- | :---
`plugins.replication.follower.index.recovery.chunk_size` | 10 MB| 在文件传输期间，追随者群集要求的块大小。将块大小指定为值和单位，例如10 MB，5 kb。看[支持单位]({{site.url}}{{site.baseurl}}/opensearch/units/)。
`plugins.replication.follower.index.recovery.max_concurrent_file_chunks` | 4| 每个恢复可以并行发送的文件块请求数。
`plugins.replication.follower.index.ops_batch_size` | 50000| 可以在复制同步阶段一次获取操作数量。
`plugins.replication.follower.concurrent_readers_per_shard` | 2| 在复制的同步阶段，来自每个碎片的自行车群集群的并发请求数。
`plugins.replication.autofollow.fetch_poll_interval` | 30年代| 多久自动一次-遵循任务调查领导者集群的新匹配索引。
`plugins.replication.follower.metadata_sync_interval` | 60年代| 追随者群集对更新的索引元数据进行调查的频率进行调查。
`plugins.replication.translog.retention_lease.pruning.enabled` | 真的| 如果启用了，请根据领导指数的保留租赁来修剪转换。
`plugins.replication.translog.retention_size` | 512 MB| 控制领导索引上翻译的大小。


