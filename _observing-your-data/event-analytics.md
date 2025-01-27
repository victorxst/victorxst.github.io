---
layout: default
title: 事件分析
nav_order: 20
redirect_from:
  - /observing-your-data/event-analytics/
---

# 事件分析

可观察性的事件分析是您可以使用的[管道处理语言]({{site.url}}{{site.baseurl}}/search-plugins/sql/ppl/index) （PPL）查询以构建和查看数据的不同可视化。

## 开始活动分析

要开始，请选择**可观察性** 在OpenSearch仪表板中，然后选择**事件分析**。如果您想开始探索而不添加任何数据，请选择**添加样品**，仪表板添加了您可以与之交互的示例可视化。

## 建立查询

要生成自定义可视化，您必须首先指定一个PPL查询。然后，OpenSearch仪表板自动根据查询结果创建可视化。

例如，以下ppl查询返回数据中当前有多少个主机地址的计数。

```
source = opensearch_dashboards_sample_data_logs | fields host | stats count()
```

默认情况下，仪表板显示了数据的最后15分钟的结果。要从不同的时间范围内查看数据，请使用日期和时间选择器。

有关构建PPL查询的更多信息，请参阅[管道处理语言]({{site.url}}{{site.baseurl}}/search-plugins/sql/ppl/index)。

## 保存可视化

仪表板生成可视化后，如果要在以后返回或要将其添加到一个，则必须保存它[操作面板]({{site.url}}{{site.baseurl}}/observing-your-data/operational-panels)。

要保存可视化，请扩展旁边的保存下拉菜单**刷新**，输入可视化的名称，然后选择**节省**。您可以在事件分析页面上重新打开任何保存的可视化。

## 创建事件分析可视化并将其添加到仪表板

此功能可在OpenSearch仪表板版本2.7和更高版本中获得。它可以使用2.7版或更高版本中创建的新可视化，该可视化使用PPL从Opensearch或Prometheus等联合数据源查询数据。
{: .note}

在仪表板上呈现您的可视化，而不是事件分析页面，使用户更容易浏览和解释数据。

要创建PPL可视化，请按照以下步骤：

1. 在主菜单上，选择**可视化** >**ppl**。
2. 在里面**可观察性** >**日志** >**资源管理器** 窗口，输入索引源**ppl查询** 例如，字段`source = opensearch_dashboards_sample_data_flights | stats count() by DestCountry`。您必须使用PPL语法输入查询。
3. 设置时间过滤器，例如**本星期**，然后选择**刷新**。
4. 选择可视化类型，例如**馅饼**，从右侧栏下拉菜单中。
5. 选择**节省** 并输入可视化的名称。

您已经创建了一个新的可视化，可以添加到新的或现有的仪表板中。要向仪表板添加PPL查询，请按照以下步骤：

1. 选择**仪表板** 从主菜单。
2. 在里面**仪表板** 窗口，选择**创建>仪表板**。
3. 在里面**编辑新仪表板** 窗口，选择**添加现有**。
4. 在里面**添加面板** 窗口，选择**ppl** 并选择可视化。现在它显示在仪表板上。
5. 选择**节省** 并输入仪表板的名称。
6. 要向仪表板添加更多可视化，请选择**选择现有的可视化** 并按照上述步骤进行操作。或者选择**创建新的** 然后选择**ppl** 在里面**新的可视化** 窗户。您将返回事件分析页面，然后按照步骤1--6在前面的说明中。

![如何创建事件分析可视化的演示并将其添加到仪表板中]({{site.url}}{{site.baseurl}}/images/dashboards/event-analytics-dashboard.gif)

### 事件分析可视化的局限性

事件分析可视化目前不支持[仪表板查询语言（DQL）]({{site.url}}{{site.baseurl}}/dashboards/discover/dql/) 或者[查询域-特定语言（DSL）]({{site.url}}{{site.baseurl}}/query-dsl/index/)，并且他们不使用索引模式。注意以下限制：

- 事件分析可视化仅使用使用下拉界面创建的过滤器。如果您的仪表板中有DQL查询或DSL过滤器，则可视化不使用它们。
- 这**仪表板** 滤波器下拉界面仅显示来自同一仪表板中其他可视化的默认索引模式或索引模式的字段。

## 查看日志

以下是您可以用来查看日志的方法。

### 关联日志和痕迹

如果您定期跟踪跨应用程序的事件，则可以关联日志和跟踪。要查看相关性，您必须根据打开的遥测标准（类似于Trace Analytics）为踪迹索引。一旦添加`TraceId` 在日志中，您可以在事件Explorer日志详细信息中查看相关的跟踪信息。此方法使您可以将与相同执行上下文相对应的日志和跟踪关联。

![跟踪日志相关]({{site.url}}{{site.baseurl}}/images/trace_log_correlation.gif)

### 查看周围事件

如果您想了解您正在查看的日志事件的更多信息，则可以选择**查看周围事件** 为了更大的了解，在兴趣时期正在发生的事情。

![周围事件]({{site.url}}{{site.baseurl}}/images/surrounding_events.gif)

### 直播日志

如果您希望观看事件发生现场直播，则可以配置一个间隔，因此事件分析会自动刷新内容。实时尾巴使您可以根据提供的PPL查询来播放日志到OpenSearch搜索可观察性事件分析，并提供丰富的功能（例如过滤器）。这样做可以改善您的调试经验，并让您实际监视原木-时间不必手动刷新。

您还可以选择间隔并在它们之间切换以决定活尾部应播放的频率。此功能类似于CLI的功能`tail -f` 命令仅通过消除大部分实时日志来检索最新的实时日志。Live Tail还为您提供了OpenSearch在实时流中收到的实时日志的总数，您可以使用该日志，以更好地了解传入的流量。

![活尾]({{site.url}}{{site.baseurl}}/images/live_tail.gif)

