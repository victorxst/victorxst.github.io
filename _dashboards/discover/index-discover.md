---
layout: default
title: 发现
nav_order: 20
has_children: true
---

# 发现

**发现** 是在OpenSearch仪表板中探索数据的工具。您可以使用**发现** 在仪表板上以视觉表示您的数据并提供高度-关键指标的级别视图。

以下图像代表典型的**发现** 使用示例数据的页面。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/discover-app.png" alt="Discover start screen" width="700">

## 入门

在本教程中，您将了解使用**发现** 到：

- 添加数据。
- 解释和可视化数据。
- 分享数据发现。
- 设置警报。

## 先决条件

以下是使用的先决条件**发现**：

- 安装[OpenSearch仪表板2.10或更高版本](https://opensearch.org/downloads.html)。
- 添加OpenSearch[样本数据]({{site.url}}{{site.baseurl}}/dashboards/quickstart/) 或将您自己的数据导入OpenSearch。
- 对OpenSearch有基本的理解[文档和索引]({{site.url}}{{site.baseurl}}/im-plugin/index/)。

## 添加数据

必须将数据添加到OpenSearch进行分析之前。在本教程中，您将使用示例数据。要了解导入您自己的数据，请参阅[管理索引]({{site.url}}{{site.baseurl}}/im-plugin/index/)。

要添加示例数据，请遵循以下步骤：

1. 在OpenSearch仪表板主页上，选择**添加示例数据**。
2. 选择所需的示例数据，然后选择**添加数据** 按钮。屏幕截图**添加示例数据** 接口如下图所示。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/add-sample.png" alt="Add sample data interface"  width="700">

## 定义搜索

要定义搜索，请执行以下步骤：

1. 在OpenSearch仪表板导航菜单上，选择**发现**。
2. 选择要使用的数据。在这种情况下，选择`opensearch_dashboards_sample_data_flights` 从鞋面-左下拉菜单。
3. 选择日历图标（{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/icons/calendar-oui.png" class ="inline-icon" alt ="calendar icon"/> {:/}）更改搜索的时间范围，然后选择**刷新**。

您会看到与下图中类似的视图。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/define-search.png" alt="Discover interface showing search of flight sample data for Last 7 days"  width="700">

## 添加数据字段并查看数据详细信息

文档表包含文档数据。每行代表一个文档，每一列包含一个不同的文档字段，代表指标，例如飞行目的地，平均票价和飞行延迟。您可以根据需要在文档表中添加，删除或修改数据字段，以满足您的数据分析要求。

要在文档表中添加或删除字段，请按照以下步骤：

1. 查看下列出的数据字段**可用字段** 并选择加号图标（{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/icons/plus-icon.png" class ="inline-icon" alt ="plus icon"/> {:/}）将所需字段添加到文档表中。该字段将自动添加到两个**选定字段** 和文档表。对于此示例，选择字段`Carrier`，`AvgTicketPrice`， 和`Dest`。
2. 要安排或排序列，请选择**排序字段** >**选择要排序的字段** 然后按照您希望订购的顺序拖放字段。

您会看到与下图中类似的视图。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/add-data-fields.png" alt="Discover interface showing adding and sorting data fields"  width="700">

您可以在文档表中查看个人或多个字段。要在文档表中收集有关数据的信息，请按照以下步骤：

1. 从数据表的左边-侧列，选择Inspect ICON（{::nomarkdown} <IMG SRC ="{{site.url}}{{site.baseurl}}/images/icons/inspect-icon.png" class ="inline-icon" alt ="inspect icon"/> {:/}）打开**文档详细信息** 窗户。选择最小化图标（{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/icons/minimize-icon.png" class ="inline-icon" alt ="minimize icon"/> {:/}）关闭**文档详细信息** 窗户。
2. 查看数据详细信息。您可以在**桌子** 和**JSON** 标签以您首选格式查看数据。
3. 选择**查看周围的文件** 查看其他日志条目的数据，要么在当前文档之前或遵循您的当前文档或选择**查看单个文档** 查看特定的日志条目。

您会看到与下图中类似的视图。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/doc-details.png" alt="Document details interface"  width="700">

## 搜索数据

您可以使用搜索工具栏或输入[DQL]({{site.url}}{{site.baseurl}}/dashboards/discover/dql/) 使用**DevTools** 控制台搜索数据。尽管搜索工具栏最适合基本查询，例如字段名称查询，但DQL最适合复杂查询，例如术语，字符串，布尔值，日期，范围或嵌套查询。DQL在输入时为字段和操作员提供建议，以帮助您构建结构化查询。

要搜索数据，请执行以下步骤：

1. 在DQL搜索栏中输入简单的查询。例如，输入`FlightDelay:true`，搜索延迟的航班。
2. 选择**更新** 搜索栏右侧的按钮。
3. 在DQL搜索栏中输入更复杂的查询，然后选择**更新**。例如，输入`FlightDelay:true AND FlightDelayMin >= 60`，搜索数据是否延迟60分钟或更长时间。

## 过滤数据

过滤器允许您通过指定某些标准来缩小查询的结果。您可以按字段，值或范围过滤。这**添加过滤器** 流行音乐-UP建议可用的字段和运营商。

要过滤您的数据，请执行以下步骤：

1. 在DQL搜索栏下，选择**添加过滤器**。
2. 从**场地**，**操作员**， 和**价值** 下拉列表。例如，选择`Cancelled`，`is`， 和`true`。
3. 选择**节省**。
4. 要删除过滤器，请选择“交叉图标”（{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/icons/cross-icon.png" class ="inline-icon" alt ="cross icon"/> {:/}）旁边的过滤器名称。
5. 添加更多过滤器以进一步探索数据。

## 保存搜索

为了保存您的搜索，包括查询文本，过滤器和当前数据视图，请遵循以下步骤：

1. 选择**节省** 在上部-右上角。
2. 给搜索一个标题，然后选择**节省**。
3. 选择**打开** 访问保存的搜索。

## 通过发现创建数据可视化

使用**发现** 应用程序，请执行以下步骤：

1. 选择“检查”图标（{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/icons/inspect-icon.png" class ="inline-icon" alt ="inspect icon"/> {:/}）要可视化的字段旁边。

   您会看到类似于下图的视图。
   
   <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/visualize-discover.png" alt ="Visualize data findings interface" width="700"/>

2. 选择**可视化** 按钮。这**可视化** 打开应用程序并显示可视化。了解更多有关**可视化** 应用程序和数据可视化[构建数据可视化]({{site.url}}{{site.baseurl}}/dashboards/visualize/viz-index/)。

   您会看到类似于下图的视图。

   <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/visualization-flight.png" alt ="Data visualization of flight sample data field destination" width="700"/>

## 设置警报

当数据更改超出定义的阈值时，您可以设置警报以通知您。了解有关使用的更多信息**发现** 要创建和管理警报，请参阅[警告仪表板和可视化]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/dashboards-alerting/)。

