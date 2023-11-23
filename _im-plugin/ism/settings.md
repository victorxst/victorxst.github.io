---
layout: default
title: 设置
parent: 索引状态管理
nav_order: 4
---

# ISM 设置

我们不建议更改这些设置;默认值应该适用于大多数用例。

索引状态管理（ISM）将其配置存储在索引中 `.opendistro-ism-config`。请勿在不使用[ISM API 操作]({{site.url}}{{site.baseurl}}/im-plugin/ism/api/).

所有设置均可通过 OpenSearch `_cluster/settings` 操作使用。没有一个需要重新启动，所有都可以标记为 `persistent` 或 `transient`。要了解有关静态和动态设置的详细信息，请参阅[配置 OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)。

设置 | 默认 | 描述
:--- | :--- | :---
 `plugins.index_state_management.enabled` | 真 | 指定是否启用 ISM。
 `plugins.index_state_management.job_interval` |5 分钟 | 运行托管索引作业的时间间隔。
 `plugins.index_state_management.jitter` |0.6 | 添加到作业基本运行时间的随机延迟，以防止所有索引同时出现活动激增。值 0.6 表示将作业间隔的 0-60% 延迟添加到基本间隔中。例如，如果基本间隔时间为 30 分钟，则值 0.6 表示将 0 到 18 分钟之间的任何时间添加到作业间隔中。最大值为 1，这意味着额外的间隔时间为 100%。此最大值不能超过 `plugins.jobscheduler.jitter_limit`，默认值也为 0.6. 例如，如果 `plugins.index_state_management.jitter` 设置为 0.8，则 ISM 将改用 `plugins.jobscheduler.jitter_limit` 0.6。
 `plugins.index_state_management.coordinator.sweep_period` |10 分钟 | 例行背景扫描的运行频率。
 `plugins.index_state_management.coordinator.backoff_millis` |50 毫秒 | 重试失败 `ManagedIndexCoordinator` 之间的回退时间（例如，当我们更新托管索引时）。
 `plugins.index_state_management.coordinator.backoff_count` |2 | 中失败 `ManagedIndexCoordinator` 的重试次数。
 `plugins.index_state_management.history.enabled` | 真 | 指定是否启用审核历史记录。来自 ISM 的日志会自动索引到日志文档中。
 `plugins.index_state_management.history.max_docs` |2,500,000 | 滚动更新审核历史记录索引之前的最大文档数。
 `plugins.index_state_management.history.max_age` |24 小时 | 滚动更新审核历史记录索引之前的最长期限。
 `plugins.index_state_management.history.rollover_check_period` |8 小时 | 审核历史记录索引的滚动更新检查之间的时间间隔。
 `plugins.index_state_management.history.rollover_retention_period` |30 天 | 审核历史记录索引的保留时间。
 `plugins.index_state_management.allow_list` | 所有操作 | 可以使用的操作列表。
