---
layout: default
title: 管理
parent: 警报
nav_order: 5
redirect_from:
  - /monitoring-plugins/alerting/settings/
---

# 管理


## 警报索引

警报功能创建了几个索引和一个别名。安全插件演示脚本将它们配置为[系统索引]({{site.url}}{{site.baseurl}}/security/configuration/system-indices/) 额外的保护层。不要在不使用警报API的情况下删除这些索引或修改其内容。

指数| 目的
：--- | ：---
`.opendistro-alerting-alerts` | 商店正在进行的警报。
`.opendistro-alerting-alert-history-<date>` | 存储完整警报的历史记录。
`.opendistro-alerting-config` | 存储监视，触发器和目的地。[拍摄快照]({{site.url}}{{site.baseurl}}/opensearch/snapshots/snapshot-restore) 此索引备份您的警报配置。
`.opendistro-alerting-alert-history-write` （别名）| 为`.opendistro-alerting-alert-history-<date>` 指数。

默认情况下，所有警报索引都隐藏。摘要，提出以下请求：

```
GET _cat/indices?expand_wildcards=open,hidden
```


## 警报设置

我们不建议更改这些设置；在大多数用例中，默认值应该很好。

所有设置均可使用OpenSearch提供`_cluster/settings` API。无需重新启动，所有人都可以标记`persistent` 或者`transient`。要了解有关静态和动态设置的更多信息，请参阅[配置OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)。

环境| 默认| 描述
：--- | ：--- | ：---
`plugins.scheduled_jobs.enabled` | 真的| 是否启用了警报插件。如果禁用，所有监视器会立即停止运行。
`plugins.alerting.index_timeout` | 60年代| 使用REST API创建监视器和目的地的超时。
`plugins.alerting.request_timeout` | 10s| 插件的其他请求超时。
`plugins.alerting.action_throttle_max_value` | 24H| 您可以设置为动作节流的最长时间。默认情况下，此值在OpenSearch仪表板中显示为1440分钟。
`plugins.alerting.input_timeout` | 30年代| 监视器可以发出搜索请求需要多长时间。
`plugins.alerting.bulk_timeout` | 120年代| 监视器可以将警报写入警报索引多长时间。
`plugins.alerting.alert_backoff_count` | 3| 在操作失败之前，要编写警报的重试次数。
`plugins.alerting.alert_backoff_millis` | 50ms| 在两次重新恢复之间等待的时间---每个重试失败后，都会呈指数增长。
`plugins.alerting.alert_history_rollover_period` | 12H| 一次检查一次的频率`.opendistro-alerting-alert-history-write` 别名应转换为新的历史记录索引，以及警报插件是否应删除任何历史记录索引。
`plugins.alerting.move_alerts_backoff_millis` | 250| 在两次重新恢复之间等待的时间---每个重试失败后，都会呈指数增长。
`plugins.alerting.move_alerts_backoff_count` | 3| 删除了监视器或触发器后，将警报移动到已删除状态的重试次数。
`plugins.alerting.monitor.max_monitors` | 1000| 用户可以创建的最大监控器数量。
`plugins.alerting.alert_history_max_age` | 30d| 最古老的文件存储在`.opendistro-alert-history-<date>` 创建新索引之前的索引。如果此时间段的警报数量不超过`alert_history_max_docs`，提醒每个时期创建一个历史指数（例如，每30天一个索引）。
`plugins.alerting.alert_history_max_docs` | 1000| 存储在`.opendistro-alert-history-<date>` 创建新索引之前的索引。
`plugins.alerting.alert_history_enabled` | 真的| 是否创建`.opendistro-alerting-alert-history-<date>` 索引。
`plugins.alerting.alert_history_retention_period` | 60d| 在自动删除历史记录之前保留历史记录索引的时间。
`plugins.alerting.destination.allow_list` | ["chime"，，，，"slack"，，，，"custom_webhook"，，，，"email"，，，，"test_action"这是给出的| 允许目的地的清单。如果您不想允许用户进入某种类型的目的地，则可以从此列表中删除它，但是我们建议将此设置作为-是。
`plugins.alerting.filter_by_backend_roles` | "false" | 通过后端角色限制了对监视器的访问。看[警告安全性]({{site.url}}{{site.baseurl}}/monitoring-plugins/alerting/security/)。
`plugins.scheduled_jobs.sweeper.period` | 5m| 警报功能使用其"job sweeper" 要定期检查新作业或更新的作业的组件。此设置是扫地机检查以查看是否有任何工作（监视器）发生变化并需要重新安排的速率。
`plugins.scheduled_jobs.sweeper.page_size` | 100| 扫地机的页面大小。您不需要更改此值。
`plugins.scheduled_jobs.sweeper.backoff_millis` | 50ms| 清除器之间等待之间的时间---每个重试失败后，都会呈指数增长。
`plugins.scheduled_jobs.sweeper.retry_count` | 3| 扫地机在扔错误之前应重试的总次数。
`plugins.scheduled_jobs.request_timeout` | 10s| 扫除工作碎片的要求的超时。

