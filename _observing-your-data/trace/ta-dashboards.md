---
layout: default
title: OpenSearch仪表板插件
parent: 跟踪分析
nav_order: 50
redirect_from:
  - /observability-plugin/trace/ta-dashboards/
  - /monitoring-plugins/trace/ta-dashboards/
---

# 跟踪分析opensearch仪表板插件

OpenSearch仪表板的Trace Analytics插件在-A-浏览您的应用程序性能的可见性，以及在单个轨迹上钻取的能力。有关安装说明，请参阅[独立opensearch仪表板插件安装]({{site.url}}{{site.baseurl}}/install-and-configure/install-dashboards/plugins/)。

这**仪表板** 查看组通过HTTP方法和路径一起跟踪，以便您可以看到与特定操作相关的平均延迟，错误率和趋势。要进行更集中的视图，请尝试通过跟踪组名称进行过滤。

![仪表板视图]({{site.url}}{{site.baseurl}}/images/ta-dashboard.png)

要在构成痕量组的轨迹上钻取，请选择右侧列中的轨迹数量。然后选择单个跟踪以进行详细摘要。

![详细的跟踪视图]({{site.url}}{{site.baseurl}}/images/ta-trace.png)

这**服务** 查看列出了应用程序中的所有服务，以及一个交互式地图，显示各种服务如何相互连接。与仪表板相反，该仪表板有助于通过操作确定问题，服务图有助于通过服务识别问题。尝试按错误率或延迟进行排序，以了解应用程序的潜在问题领域。

![服务视图]({{site.url}}{{site.baseurl}}/images/ta-services.png)

