---
layout: default
title: 警告仪表板和可视化
parent: 警报
nav_order: 50
---

# 警告仪表板和可视化
引入2.9
{: .label .label-purple }

在单个，合并的视图中创建，管理和采取行动，并迅速识别和解决问题。使用**仪表板** 接口：

- 设置，添加和调整触发警报和通知的规则和条件。
- 创建显示趋势和模式的图形，并建立直观的仪表板，以实时了解重要的指标和数据点。
- 在一个地方监视您的警报-A-一眼景色。

以下图像为您提供了仪表板接口的快照。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/alerting-dashboard.png" alt="Example alerting visualization" width="800" height="800">

## 入门

在开始之前，您必须有：

- 已安装的OpenSearch和OpenSearch仪表板版本2.9或更高版本。看[安装OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/install-opensearch/index/)。
- 安装了警报和通知仪表板插件。看[管理OpenSearch仪表板插件]({{site.url}}{{site.baseurl}}/install-and-configure/install-dashboards/plugins/) 开始。

## 配置管理设置

用户只能访问，创建或管理其拥有权限的资源的警报。OpenSearch和OpenSearch仪表板权限控制对仪表板和可视化的访问。它默认为启用，并在**仪表板管理** >**高级设置** >**可视化**。如果设置被禁用，则不会出现。您可以在群集级别禁用设置`opensearch-dashboards.yml` 文件。

## 警报可视化的一般要求

警报可视化显示为时间-系列图表可为您提供警报，警报状态，最后更新的时间以及警报的原因的快照。您可以在图表上显示多达10个指标，每个系列可以显示为图表上的一行。

请记住，设置或创建警报可视化时，请记住以下要求。可视化：

- 必须是一个[Vizlib线图](https://community.vizlib.com/support/solutions/articles/35000107262-vizlib-line-chart-introduction)
- 必须至少包含一个y-轴度量聚集
- 一定不能没有-y-轴度量聚合类型
- 必须使用x的日期直方图聚合类型-轴桶
- 必须有x-底部的轴
- 必须定义一个x-轴聚集桶
- 必须有一个有效的时间-基于x-轴

## 创建警报监视器

默认情况下，当您开始使用仪表板接口创建警报监视器工作流程时，您会向您提供菜单-驱动界面。该界面提供了一系列选项，这些选项全屏显示在POP中-UPS，拉动-跌落或下拉次数。它们允许您定义可以监视的指标，设置阈值，自定义自动化工作流程的触发器并在满足条件时生成操作。目前，您只能创建[每个查询监视器]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/monitors/)。

创建警报监视器：

1. 选择**仪表板** 从OpenSearch仪表板主菜单中。
2. 来自**仪表板** 窗口，选择**创造** 然后选择**仪表板**。
3. 选择**添加现有**，然后从中选择适当的警报可视化**添加面板** 列表。可视化添加到仪表板中。
4. 从可视化面板中，选择省略号图标（{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/ellipsis-icon.png" class ="inline-icon" alt ="ellipsis icon"/> {:/}）。
5. 来自**选项** 菜单，选择**添加警报显示器**。
6. 输入信息**监视详细信息** 和**触发器**。
7. 选择**创建监视器**。监视器被添加到可视化中。

以下屏幕截图显示了这些步骤的一个示例。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/create-monitor-menu.png" alt="Create monitor interface" width="400" height="400">

## 关联监视器

您可以使用仪表板接口而不是插件页面将某些监视器类型与可视化相关联，从而为您提供一个可以添加，查看和编辑监视器数据的接口。

继续在上一节中创建的警报可视化和仪表板，通过遵循以下步骤将现有监视器与可视化相关联：

1. 从可视化面板中，选择省略号图标（{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/ellipsis-icon.png" class ="inline-icon" alt ="ellipsis icon"/> {:/}）。
2. 选择**相关监视器**。
3. 来自**选择监视器** 下拉菜单，选择监视器。仅在下拉菜单中列出了合格的显示器。
4. 查看监视器的基本信息。要查看全面的细节，请选择**查看监视器页面** 打开警报插件页面。
5. 选择**副监视器**。现有监视器与可视化相关联。

## 探索警报详细信息

创建或关联的警报监视器后，请验证监视器正在生成警报并通过以下各个步骤探索警报详细信息：

1. 打开警报仪表板。使用三角图标（{::nomarkdown} <img src =可视化表示警报。"{{site.url}}{{site.baseurl}}/images/dashboards/triangle-icon.png" class ="inline-icon" alt ="triangle icon"/> {:/}）。
2. 悬停在三角图标上（{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/triangle-icon.png" class ="inline-icon" alt ="triangle icon"/> {:/}）查看高-级别数据，例如警报数量。要调查警报详细信息，请选择“三角图”图标（{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/triangle-icon.png" class ="inline-icon" alt ="triangle icon"/> {:/}）打开带有更详细的监视器信息的飞行。另外，选择省略号图标（{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/ellipsis-icon.png" class ="inline-icon" alt ="ellipsis icon"/> {:/}）在可视化面板中，然后选择**查看事件**。
3. 选择省略号图标（{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/ellipsis-icon.png" class ="inline-icon" alt ="ellipsis icon"/> {:/}），然后**警报** >**相关监视器**。
4. 从列表中选择一个警报监视器。可视化面板中显示了诸如历史记录，警报和相关可视化的信息。
5. UNINK或编辑监视器。
   1.通过选择链接图标（{::nomarkdown} <img src =，将监视器与可视化链接到可视化。"{{site.url}}{{site.baseurl}}/images/dashboards/link-icon.png" class ="inline-icon" alt ="link icon"/> {:/}）**动作**。这只会将监视器与可视化链接起来。它不会删除监视器。
   2.通过选择“编辑图标”（{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/edit-icon.png" class ="inline-icon" alt ="edit icon"/> {:/}）。

## 下一步

- [了解有关仪表板应用程序的更多信息](https://opensearch.org/docs/latest/dashboards/dashboard/index/)。
- [了解有关警报的更多信息](https://opensearch.org/docs/latest/observing-your-data/alerting/index/)。

