---
layout: default
title: 设置
parent: 索引汇总
nav_order: 30
---

# 索引汇总设置

我们不建议更改这些设置;默认值应该适用于大多数用例。

所有设置均可通过 OpenSearch `_cluster/settings` 操作使用。没有一个需要重新启动，所有都可以标记为 `persistent` 或 `transient`。要了解有关静态和动态设置的详细信息，请参阅[配置 OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)。

设置 | 默认 | 描述
:--- | :--- | :---
 `plugins.rollup.search.backoff_millis` |1000 毫秒 | 失败的汇总作业重试之间的回退时间。
 `plugins.rollup.search.backoff_count` |5 | 对于失败的汇总作业，插件应尝试的重试次数。
 `plugins.rollup.search.search_all_jobs` | 假 |OpenSearch 是否应返回与所有指定搜索词匹配的所有作业。如果禁用，OpenSearch 将仅返回一个（而不是所有）与搜索词匹配的作业。
 `plugins.rollup.dashboards.enabled` | 真 | 是否在 OpenSearch 控制面板中启用汇总。
 `plugins.rollup.enabled` | 真 | 是否启用了 rollup 插件。
 `plugins.ingest.backoff_millis` |1000 毫秒 | 汇总作业的数据引入之间的回退时间。
 `plugins.ingest.backoff_count` |5 | 插件应尝试对失败的摄取进行多少次重试。
