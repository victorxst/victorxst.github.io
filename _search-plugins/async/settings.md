---
layout: default
title: 设置
parent: 异步搜索
nav_order: 4
---

# 异步搜索设置

异步搜索插件将几个设置添加到标准OpenSearch集群设置中。它们是动态的，因此您可以在不重新启动群集的情况下更改插件的默认行为。要了解有关静态和动态设置的更多信息，请参阅[配置OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)。

您可以将设置标记为`persistent` 或者`transient`。

例如，更新结果索引的保留期：

```json
PUT _cluster/settings
{
  "transient": {
    "plugins.asynchronous_search.max_wait_for_completion_timeout": "5m"
  }
}
```

环境| 默认| 描述
：--- | ：--- | ：---
`plugins.asynchronous_search.max_search_running_time` | 12小时| 搜索终止搜索的最大运行时间。
`plugins.asynchronous_search.node_concurrent_running_searches` | 20| 每个协调器节点运行的并发搜索。
`plugins.asynchronous_search.max_keep_alive` | 5天| 搜索结果的最长时间可以存储在集群中。
`plugins.asynchronous_search.max_wait_for_completion_timeout` | 1分钟| 最大值`wait_for_completion_timeout` 范围。
`plugins.asynchronous_search.persist_search_failures` | 错误的| 持续的异步搜索结果以系统索引中的搜索失败结束。

