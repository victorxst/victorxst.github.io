---
layout: default
title: 监视器
nav_order: 1
parent: 警报
has_children: true
redirect_from:
  - /monitoring-plugins/alerting/monitors/
---

# 监视器

在OpenSearch中主动监视您的数据，并具有警报和异常检测中可用的功能。例如，您可以将异常检测与警报配对，以确保在检测到异常后立即通知您。您可以通过设置检测器来自动检测流数据中的异常值并进行监控来执行此操作，以在数据超过某些阈值时提醒您通知。

## 监视类型

警报插件提供以下监视器类型：

1. **每个查询**：运行查询并根据匹配条件生成警报通知。看[每个查询监视器]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/per-query-bucket-monitors/) 有关创建和使用此监视器类型的信息。
1. **每个水桶**：运行一个查询，该查询根据数据集中的汇总值评估触发条件。看[每个水桶监视器]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/per-query-bucket-monitors/) 有关创建和使用此监视器类型的信息。
1. **每个集群指标**：在集群上运行API请求以监控其健康状况。看[每个集群指标监视器]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/per-cluster-metrics-monitors/) 有关创建和使用此监视器类型的信息。
1. **每个文件**：运行一个查询（或标签组合的多个查询），该查询返回与警报通知触发条件匹配的单个文档。看[每个文档监视器]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/per-document-monitors/) 有关创建和使用此监视器类型的信息。
1. **复合监视器**：在单个工作流程中运行多个显示器，并根据多个触发条件生成单个警报。看[复合监视器]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/composite-monitors/) 有关创建和使用此监视器类型的信息。

您可以创建的最大监视器数为1,000。您可以通过更新来更改集群的默认警报数量`plugins.alerting.monitor.max_monitors` 使用[集群设置API]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/settings/)。
{: .tip}

## 监视变量

下表列出了可自定义监视器的变量。

多变的| 数据类型| 描述
:--- | :--- | :---
`ctx.monitor` | 目的| 包括`ctx.monitor.name`，`ctx.monitor.type`，`ctx.monitor.enabled`，`ctx.monitor.enabled_time`，`ctx.monitor.schedule`，`ctx.monitor.inputs`，`triggers` 和`ctx.monitor.last_update_time`。
`ctx.monitor.user` | 目的| 包括有关创建监视器的用户的信息。包括`ctx.monitor.user.backend_roles` 和`ctx.monitor.user.roles`，其中包含分配给用户的后端角色和角色的数组。看[警告安全性]({{site.url}}{{site.baseurl}}/monitoring-plugins/alerting/security/) 了解更多信息。
`ctx.monitor.enabled` | 布尔| 是否启用了监视器。
`ctx.monitor.enabled_time` | 毫秒| Unix Epoch上一次启用了显示器的时间。
`ctx.monitor.schedule` | 目的| 包含一个时间表的时间表或何时应运行。
`ctx.monitor.schedule.period.interval` | 整数| 监视器运行的间隔。
`ctx.monitor.schedule.period.unit` | 细绳| 间隔的时间单位。
`ctx.monitor.inputs` | 大批| 包含用于创建监视器的索引和定义的数组。
`ctx.monitor.inputs.search.indices` | 大批| 包含监视器观察的索引的数组。
`ctx.monitor.inputs.search.query` | N/A。| 用于定义监视器的定义。

下表列出了可以与监视器一起使用的其他变量。

多变的| 数据类型| 描述
:--- | :--- : :---
`ctx.results` | 大批| 一个带有一个元素的数组，例如`ctx.results[0]`。包含查询结果。如果触发器无法检索结果，则此变量为空。看`ctx.error`。
`ctx.last_update_time` | 毫秒| unix epoch最后更新的显示器的时间。
`ctx.periodStart` | 细绳| UNIX时间戳在警报触发的时期开始。例如，如果监视器每10分钟运行一次，则可能从10:40开始，并在10:50结束。
`ctx.periodEnd` | 细绳| 警报触发的时期结束。
`ctx.error` | 细绳| 错误消息如果触发器无法检索结果或无法评估触发器，通常是由于编译错误或无效指针异常。否则为无效。
`ctx.alert` | 目的| 当前的主动警报（如果存在）。包括`ctx.alert.id`，`ctx.alert.version`， 和`ctx.alert.isAcknowledged`。null如果没有警报处于活动状态。仅与查询一起使用-电平监视器。
`ctx.dedupedAlerts` | 目的| 已经触发的警报。OpenSearch保留现有警报，以防止插件创建无数量相同的警报。仅带有水桶-电平监视器。
`ctx.newAlerts` | 目的| 新创建的警报。仅带有水桶-电平监视器。
`ctx.completedAlerts` | 目的| 不再持续的警报。仅带有水桶-电平监视器。
`bucket_keys` | 细绳| 逗号-监视器的存储键钥匙值的分开列表。仅适用于`ctx.dedupedAlerts`，`ctx.newAlerts`， 和`ctx.completedAlerts`。访问`ctx.dedupedAlerts[0].bucket_keys`。
`parent_bucket_path` | 细绳| 触发警报的桶的父级路径。访问`ctx.dedupedAlerts[0].parent_bucket_path`。


