---
layout: default
title: 快速入门指南
nav_order: 2
has_children: false
redirect_from:
   - /dashboards/quickstart-dashboards/
---

# 快速入门指南

该快速启动指南涵盖了您需要了解的核心概念，以便从OpenSearch仪表板开始。您将学习如何：

- 添加示例数据。
- 探索和检查数据。
- 可视化数据。

这是您打开时看到的视图**仪表板** 或者**发现** 工具。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/new-look.png" alt="Light and dark mode UI on Discover and Dashboard tools' home page" width="700">

{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/icons/alert-icon.png" class ="inline-icon" alt ="alert icon"/> {:/}**笔记**<br>在开始之前，请确保已安装了OpenSearch和OpenSearch仪表板。有关安装和配置的信息，请参阅[安装和配置OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/install-opensearch/index/) 和[安装和配置OpenSearch仪表板]({{site.url}}{{site.baseurl}}/install-and-configure/install-dashboards/index/)。
{:.note}
# 添加样本数据

示例数据集配备了可视化，仪表板和其他工具，可以在添加自己的数据之前帮助您探索仪表板。要添加示例数据，请执行以下步骤：

1. 通过连接到OpenSearch仪表板的访问[http：// localhost：5601](http://localhost:5601) 从浏览器。默认用户名和密码是`admin`。
1. 在OpenSearch仪表板上**家** 页面，选择**添加示例数据**。
2. 选择**添加数据** 添加数据集，如下图所示。

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/add-sample-data.png" alt ="Sample datasets" width="700">

# 探索和检查数据

在[**发现**]({{site.url}}{{site.baseurl}}/dashboards/discover/index-discover/)， 你可以：

- 选择要探索的数据，为该数据设置时间范围，请使用[仪表板查询语言（DQL）]({{site.url}}{{site.baseurl}}/dashboards/dql/)并过滤结果。
- 探索数据，查看各个文档，并创建表总结数据内容。
- 可视化您的发现。

## 尝试：熟悉发现

1. 在OpenSearch仪表板上**家** 页面，选择**发现**。
1. 更改[时间过滤器]({{site.url}}{{site.baseurl}}/dashboards/discover/time-filter/) 到**最近7天**，如下图所示。

    <img src ="{{site.url}}{{site.baseurl}}/images/last-7--days.png" alt ="Time filter interface" width="250"/>

2. 使用DQL查询搜索`FlightDelay:true AND DestCountry: US AND FlightDelayMin >= 60` 然后选择**更新**。你应该为我们看到结果-如下图所示，绑定的航班延迟了60分钟或更长时间。
   
    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/dql-search-field.png" alt ="DQL search field example" width="700"/>

3. 要过滤数据，请选择**添加过滤器** 然后选择一个**可用字段**。例如，选择`FlightDelayType`，**是**， 和**天气延迟** 来自**场地**，**操作员**， 和**价值** 下拉列表，如下图所示。

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/filter-data-discover.png" alt ="Filter data by FlightDelayType field" width="250"/>

# 可视化数据

原始数据可能很难理解和使用。数据可视化可帮助您以视觉形式准备和呈现数据。在**仪表板** 你可以：

- 在单个视图中显示数据。
- 构建动态仪表板。
- 创建和共享报告。
- 嵌入分析以区分您的应用程序。

## 尝试：熟悉仪表板

1. 在OpenSearch仪表板上**家** 页面，选择**仪表板**。
1. 选择**[航班]全球航班数据** 在里面**仪表板** 窗口，如下图所示。

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/dashboard-flight-quickstart.png" alt ="Data visualization dashboard" width="700"/>

1. 要将面板添加到仪表板中，请选择**编辑** 进而**添加** 从工具栏。
1. 在里面**添加面板** 窗口，选择现有面板**[航班]延迟存储桶**。您会看到流行音乐-向上右下方的窗口确认您已经添加了面板。
1. 选择`x` 关闭**添加面板** 窗户。
1. 查看添加的面板**[航班]延迟存储桶**，如下图所示，它是仪表板上的最后一个面板。

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/add-panel.png" alt ="Add panel to dashboard" width="700"/>

## 尝试：创建一个可视化面板

继续前面的仪表板，您将创建一个条形图，比较取消的航班数量和延迟航班的延迟类型，然后将面板添加到仪表板中：

1. 更改默认值[时间范围]({{site.url}}{{site.baseurl}}/dashboards/discover/time-filter/) 从**24小时** 到**最近7天**。
1. 在工具栏中，选择**编辑**， 然后**创建新的**。
1. 选择**visbuilder** 在里面**新的可视化** 窗户。
1. 在里面**数据源** 下拉列表，选择`opensearch_dashboards_sample_data_flights`。
1. 拖动字段**取消** 和**飞机延迟** 到y-轴列。
1. 拖动字段**FlightDelayType** 到X-轴列。
1. 选择**节省** 并在**标题** 场地。
2. 选择**保存并返回**。如下图所示，以下条形图添加为仪表板上的最后一个面板。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/viz-panel-quickstart.png" alt="Creating a visualization panel" width="700"/>

# 与数据互动

交互式仪表板使您可以更深入地分析数据，并以多种方式过滤数据。在仪表板中，您可以使用仪表板直接与仪表板上的数据进行交互-级过滤器。例如，继续使用前面的仪表板，您可以过滤以显示特定航空公司的延迟和取消。

## 尝试：与样品飞行数据进行互动

1. 在**[航班]航空公司航空公司** 面板，选择**OpenSearch-空气**。仪表板自动更新。
1. 选择**节省** 保存自定义的仪表板。

另外，您可以使用仪表板工具栏应用过滤器：

1. 在仪表板工具栏中，选择**添加过滤器**。
1. 来自**场地**，**操作员**， 和**价值** 下拉列表，选择**载体**，**是**， 和**OpenSearch-空气**，分别如下图所示。

    <img src ="{{site.url}}{{site.baseurl}}/images/edit-filter.png" alt ="Edit field interface" width="400"/>

1. 选择**节省**。仪表板自动更新，结果是下图中显示的仪表板。

  <img src ="{{site.url}}{{site.baseurl}}/images/interact-filter-dashboard.png" alt ="Dashboard view after applying Carrier filter" width="700"/>

# 下一步

- **可视化数据**。要了解有关OpenSearch仪表板中数据可视化的更多信息，请参见[**构建数据可视化**]({{site.url}}{{site.baseurl}}/dashboards/visualize/viz-index/)。
- **创建仪表板**。要了解有关在OpenSearch仪表板中创建仪表板的更多信息，请参见[**创建仪表板**]({{site.url}}{{site.baseurl}}/dashboards/quickstart-dashboards/)。
- **探索数据**。要了解有关在OpenSearch仪表板中探索数据的更多信息，请参见[**探索数据**]({{site.url}}{{site.baseurl}}/dashboards/discover/index-discover/)。

