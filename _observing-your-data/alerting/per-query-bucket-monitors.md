---
layout: default
title: 每个查询和每个水桶显示器
nav_order: 5
parent: 监视器
grand_parent: 警报
has_children: false
---

# 每个查询和每个水桶显示器

每个查询显示器是一种警报监视器，可用于识别和警报针对OpenSearch索引运行的特定查询；例如，检测和响应特定查询异常的查询。每个查询监视器一次一次触发一个警报。

每个存储桶监视器是一种警报监视器，可用于识别和警报针对OpenSearch索引创建的特定数据桶。

## 创建每个查询或每个存储桶监视器

要创建每个查询监视器，请按照以下步骤：

**步骤1。** 定义您的查询和[触发器]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/triggers/)。您可以使用以下任何方法：视觉编辑器，查询编辑器或异常检测器。

   - 视觉定义适用于可以定义为的显示器"some value is above or below some threshold for some amount of time." 它对大多数监视器也很好。

   - 查询定义提供了与查询有关的灵活性（使用[OpenSearch查询DSL]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/full-text/index)）以及如何评估该查询的结果（无痛脚本）。

以下示例平均`cpu_usage` 场地：

```json
     {
       "size": 0,
       "query": {
         "match_all": {}
       },
       "aggs": {
         "avg_cpu": {
           "avg": {
             "field": "cpu_usage"
           }
         }
       }
     }
```

您也可以使用`{% raw %}{{period_start}}{% endraw %}` 和`{% raw %}{{period_end}}{% endraw %}`：

```json
     {
       "size": 0,
       "query": {
         "bool": {
           "filter": [{
             "range": {
               "timestamp": {
                 "from": "{% raw %}{{period_end}}{% endraw %}||-1h",
                 "to": "{% raw %}{{period_end}}{% endraw %}",
                 "include_lower": true,
                 "include_upper": true,
                 "format": "epoch_millis",
                 "boost": 1
               }
             }
           }],
           "adjust_pure_negative": true,
           "boost": 1
         }
       },
       "aggregations": {}
     }
```

"Start" 和"end" 请参阅监视器运行的间隔。看[监视变量]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/monitors/#monitor-variables)。

要在视觉上定义监视器，请选择**视觉编辑器**。然后选择一个源索引，时间范围，一个聚合（例如，`count()` 或者`average()`），一个数据过滤器（如果要监视源索引的子集），并且一个组-如果要在查询中包含一个聚合字段，请按字段进行字段。至少一组-如果您要定义每个存储桶监视器，则需要按字段。

视觉定义适合大多数监视器。
{: .tip }

如果使用安全插件，则只能选择有权访问的索引。有关详细信息，请参阅[警告安全性]({{site.url}}{{site.baseurl}}/security/)。

要使用查询，请选择**提取查询编辑器**，添加您的查询（使用[OpenSearch查询DSL]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/full-text/index)），并使用**跑步** 按钮。

监视器使此查询按时间表所规定的经常进行搜索；检查**查询性能** 部分并确保您对性能的影响感到满意。

仅当您定义每个查询监视器时，就可以使用异常检测。
{: .warning}

要使用异常检测器，请选择**异常检测器** 并选择您的**探测器**。

异常检测选项是与异常检测插件配对。看[异常检测]({{site.url}}{{site.baseurl}}/monitoring-plugins/ad/)。

对于异常检测器，请根据检测器间隔为监视器选择适当的时间表。否则，警报显示器可能会错过阅读结果。例如，假设您将监视器间隔和检测器间隔设置为5分钟，然后从12:00开始检测器。如果在12:05检测到异常，则可能在12:06上可用，因为编写异常和可用于查询之间的延迟。监视器在12:00到12:05之间读取异常结果，因此它无法在12:06获得异常结果。

为避免此问题，请确保警报显示器至少是检测器间隔的两倍。当您使用OpenSearch仪表板创建监视器时，异常检测器插件会生成默认的监视时间表，该时间表是检测器间隔的两倍。

每当您更新检测器的间隔时，请确保更新关联的监视器间隔，因为异常检测插件不会自动执行此操作。

**第2步。** 选择以按时间间隔（分钟，小时，天）或时间表来运行监视器的频率。如果您按时间间隔或自定义运行[自定义cron表达式]({{site.url}}{{site.baseurl}}/monitoring-plugins/alerting/cron/)，那么您必须提供时区。

**步骤3。** 向监视器添加扳机。

