---
layout: default
title: 构建数据可视化
nav_order: 40
has_children: true
---

# 构建数据可视化

通过可视化数据，您可以翻译复杂，高-卷或数值数据中的视觉表示，更易于处理。OpenSearch仪表板为您提供数据可视化工具，以改进和自动化视觉通信过程。通过使用图表，图形或地图之类的视觉元素来表示数据，您可以推进商业智能并支持数据-驱动的决定-制定和战略计划。

## 了解OpenSearch仪表板中的可视化类型

仪表板有几种可视化类型，可满足您的数据分析需求。以下各节概述了仪表板及其常见用例中的可视化类型。

### 区域图

区域图描绘了随着时间的变化，它们通常用于显示趋势。区域图表更有效地识别日志数据中的模式，例如时间范围的销售数据以及在此期间的趋势。看[使用区域图]({{site.url}}{{site.baseurl}}/dashboards/visualize/area/) 要了解有关如何在仪表板中创建和使用它们的更多信息。

 <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/area-chart-1.png" width="600" 高度="600" alt ="Example area chart in OpenSearch Dashboards">

### 条形图

条形图（垂直或水平）比较了分类数据，并描绘了一段时间内变量的变化。

垂直条形图|  水平条形图
：-------------------------：|：-------------------------：
<img src="{{site.url}}{{site.baseurl}}/images/dashboards/bar-chart-1.png" width="300" height="300" alt="Example vertical bar chart in OpenSearch Dashboards">  |  <img src="{{site.url}}{{site.baseurl}}/images/dashboards/bar-horizontal-1.png" width="300" height="300" alt="Example horizontal bar chart in OpenSearch Dashboards">

### 控件

控件是一个面板，而不是可视化类型，添加到仪表板中以过滤数据。控件使用户能够在仪表板中添加交互式输入。您可以在仪表板中创建两种类型的控件：**选项列表** 和**范围滑块**。**选项列表** 是一个下拉选项列表，允许通过术语聚合过滤数据，例如`machine.os.keyword`。**范围滑块** 允许在指定值范围内进行过滤，例如`hour_of_day`。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/controls-1.png" width="600" height="600" alt="Example visualization using controls to filter data in OpenSearch Dashboards">

### 数据表

数据表或表以表格形式显示您的原始数据。

<img src="{{site.url}}{{site.baseurl}}/images/data-table-1.png" width="600" height="600" alt="Example data table in OpenSearch Dashboards">

### 甘特图

甘特图显示了序列中独特事件的开始，结束和持续时间。甘特图在痕量分析，遥测和异常检测用例中很有用，在这些用例中，您想了解时间表中各种事件之间的互动和依赖关系。**甘特图** 目前是一个插件，而不是构建-在仪表板中的可视化类型。看[甘特图]({{site.url}}{{site.baseurl}}/dashboards/visualize/gantt/) 学习如何在仪表板中创建和使用它们。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/gantt-chart.png" width="600" height="600" alt="Example Gantt chart in OpenSearch Dashboards">

### 量规图

量规图看起来类似于模拟速度表，该模拟速度表从零读到右。他们显示了您正在测量的东西有多少，并且这种测量可以单独存在，也可以与其他测量值有关，例如针对基准或目标跟踪性能。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/gauge-1.png" width="400" height="400" alt="Example gauge chart in OpenSearch Dashboards">

### 加热地图

热图是一段时间内直方图（数值数据分布的图形表示）的视图。加热地图不使用条形图表示频率的表示，而是使用颜色以颜色区分范围的颜色显示数据。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/heat-map-1.png" width="600" height="600" alt="Example heat map in OpenSearch Dashboards">

### 线图

线图比较了一段时间内测量值的变化，例如按月销售总额或按月按月销售总额和净销售额。

<img src="{{site.url}}{{site.baseurl}}/images/line-1.png" width="600" height="600" alt="Example line graph in OpenSearch Dashboards">

### 地图

您可以在仪表板中创建两种类型的地图：坐标地图和区域地图。坐标图显示了每个位置按大小的数据值之间的差异。区域图显示了每个位置的数据值之间通过不同的颜色阴影之间的差异。看[使用地图]({{site.url}}{{site.baseurl}}/dashboards/visualize/maps/) 要了解有关仪表板中地图功能的更多信息。

#### 坐标图

坐标地图显示位置-基于地图的数据。使用坐标图可视化地图上的GPS数据（纬度和经度坐标）。有关OpenSearch的信息-支持的坐标场类型，请参阅[地理领域类型]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/geo-shape/) 和[笛卡尔现场类型]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/xy/)。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/coordinate-1.png" width="600" height="600" alt="Example coordinate map in OpenSearch Dashboards">

#### 区域图

区域地图显示了地理位置之间的模式和趋势。区域图是仪表板中的基础图之一。有关在仪表板中创建自定义向量图的信息，请参见[使用坐标和区域图]({{site.url}}{{site.baseurl}}/dashboards/visualize/geojson-regionmaps/) 学习如何在仪表板中创建和使用地图。

<img src="{{site.url}}{{site.baseurl}}/images/map-1.png" width="600" height="600" alt="Example region map in OpenSearch Dashboards">

### 降价

Markdown是仪表板中使用的标记语言，可为您的数据可视化提供上下文。使用Markdown，您可以显示信息和说明以及可视化。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/markdown.png" width="600" height="600" alt="Example coordinate map in OpenSearch Dashboards">

### 度量值

公制值或数字图比较不同度量中的值。例如，您可以创建一个指标可视化以比较两个值，例如实际销售与销售目标相比。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/metric-chart-1.png" width="400" height="400" alt="Example metric chart in OpenSearch Dashboards">

### 饼状图

饼图比较一个维度的项目值，例如总量的百分比。

<img src="{{site.url}}{{site.baseurl}}/images/pie-1.png" width="600" height="600" alt="Example pie chart in OpenSearch Dashboards">

### TSVB

时间-串联Visual Builder（TSVB）是用于创建详细时间的仪表板中的数据可视化工具-串联可视化。例如，您可以使用TSVB来构建显示数据随着时间的时间显示数据的可视化，例如随着时间的推移按状态进行飞行或按时间延迟类型随着时间的推移而延迟。当前，TSVB可用于创建以下仪表板可视化类型：区域，线路，度量，量规，邮票和数据表。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/TSVB-1.png" width="600" height="600" alt="Example TSVB in OpenSearch Dashboards">

### 标签云

标签（或单词）云是显示单词与数据集中其他单词相关的频率的一种方式。这种类型的视觉效果的最佳用途是显示单词或短语频率。

<img src="{{site.url}}{{site.baseurl}}/images/word-cloud-1.png" width="600" height="600" alt="Example Tag cloud in OpenSearch Dashboards">

### 时间线

时间轴是仪表板中的数据可视化工具，您可以用来创建时间-串联可视化。当前，时间轴可用于创建以下仪表板可视化类型：区域和线路。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/timeline-1.png" width="600" height="600" alt="Example Timeline in OpenSearch Dashboards">

### visbuilder

visbuilder是一个阻力-和-将数据可视化工具放在仪表板中。它可以立即查看数据，而无需预先选择数据源或可视化类型输出。当前，Visbuilder可用于创建以下仪表板可视化类型：区域，条，线，度量表和数据表。看[visbuilder]({{site.url}}{{site.baseurl}}/dashboards/visualize/visbuilder/) 学习如何创建和使用阻力-和-在仪表板中删除可视化。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/vis-builder-2.png" width="600" height="600" alt="Example VisBuilder in OpenSearch Dashboards">

### 维加

[维加](https://vega.github.io/vega/) 和[维加-Lite](https://vega.github.io/vega-lite/) 开放-来源，声明性语言可视化语法，用于创建，共享和保存交互式数据可视化。VEGA可视化可以使您灵活使用分层方法可视化多维数据，以便以结构化的方式构建和操纵可视化。VEGA可用于使用任何仪表板可视化类型来创建自定义的可视化。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/vega-1.png" width="600" height="600" alt="Example Vega visualization with JSON specification in OpenSearch Dashboards">

