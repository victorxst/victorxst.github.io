---
layout: default
title: 使用区域图
parent: 构建数据可视化
nav_order: 5
---

# 使用区域图

区域图是一个线图，其区域在线和轴之间用颜色阴影，是用于显示时间序列数据的主要可视化类型。您可以使用区域可视化类型在仪表板中创建区域图表，也可以使用时间序列的Visual Visual Builder（TSVB），VEGA或Visbuilder可视化工具创建区域图。对于本教程，您将使用区域可视化类型。

![区域图教程步骤的演示]({{site.url}}{{site.baseurl}}/images/dashboards/area-tutorial.gif)

# 尝试：创建一个简单的聚合-基于区域图

在本教程中[http：// localhost：5601](http://localhost:5601) 从浏览器。

您在仪表板中有几个聚合选项，选择会影响您的分析。聚合的用例从实时分析数据到使用仪表板创建可视化仪表板的用例不等。如果您需要OpenSearch中的聚合概述，请参见[聚合]({{site.url}}{{site.baseurl}}/opensearch/aggregations/) 在启动本教程之前。

确保你有[安装了最新版本的仪表板](https://opensearch.org/docs/latest/install-and-configure/install-dashboards/index/) 并在继续本教程之前添加了示例数据。_本教程使用仪表板版本2.4.1_。
{: .note}

## 设置区域图表

1. 连接到[http：// localhost：5601](http://localhost:5601) 从浏览器。
1. 选择**可视化** 从菜单中选择**创建可视化**。
1. 选择**区域** 从窗户。
1. 选择**opensearch_dashboards_sample_data_flights** 在里面**新区域/选择一个来源** 窗户。
1. 选择日历图标并将时间过滤器设置为**最近7天**。
1. 选择**更新**。

## 将聚合添加到区域图表

继续使用前面步骤中创建的区域图表，您将创建一个可视化，该可视化显示过去7天内每三个小时延迟的航班前五个日志：

1. 添加一个**指标** 聚合。
   1.下**指标**，选择**聚合** 下拉列表并选择**平均的** 然后选择**场地** 下拉列表并选择**FlightDelaymin**。
   1.下**指标**， 选择**添加** 添加另一个y-轴聚集。
   1.选择**聚合** 下拉列表并选择**最大限度** 然后选择**场地** 下拉列表并选择**FlightDelaymin**。
1. 添加一个**水桶** 聚合。
   1.选择**添加** 打开**添加水桶** 窗口，然后选择**X-轴**。
   2.从**聚合** 下拉列表，选择**日期直方图**。
   3.从**场地** 下拉列表，选择**时间戳**。
   4.选择**更新**。
2. 添加一个子-聚合。
   1.选择**添加** 打开**添加子-水桶** 窗口，然后选择**拆分系列**。
   2.从**子聚集** 下拉列表，选择**术语**。
   3.从**场地** 下拉列表，选择**飞机延迟**。
   4.选择**更新** 在图中反映这些参数。

您现在创建了以下汇总-基于区域图。

![产生的聚合-基于区域图]({{site.url}}{{site.baseurl}}/images/area-aggregation-tutorial.png)

# 相关链接

- [可视化]({{site.url}}{{site.baseurl}}/dashboards/visualize/viz-index/)
- [OpenSearch仪表板中的可视化类型]({{site.url}}{{site.baseurl}}/dashboards/visualize/viz-index/)
- [安装和配置OpenSearch仪表板]({{site.url}}{{site.baseurl}}/install-and-configure/install-dashboards/index/)
- [gentregations]（{{stite.url}}} {{site.baseurl}}/openSearch/gentregations/[聚合]({{site.url}}{{site.baseurl}}/opensearch/aggregations/)[gentregations]（{{stite.url}}} {{site.baseurl}}/openSearch/gentregations/

