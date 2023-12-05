---
layout: default
title: 异常检测可视化和仪表板
parent: 异常检测
nav_order: 50
---

# 异常检测仪表板和可视化
引入2.9
{: .label .label-purple }

OpenSearch提供了一种自动化手段，用于检测有害异常值并在启用异常检测时保护您的数据。当应用于指标时，OpenSearch使用算法来连续分析系统和应用，确定正常基准和表面异常。

您可以将数据可视化连接到OpenSearch数据集，然后创建，运行和查看异常警报，并根据在**仪表板** 界面。只有几个步骤，您就可以将轨迹，指标和日志汇总在一起，以使您的应用程序和基础架构完全可观察到。

## 入门

在开始之前，您必须有：

- 已安装的OpenSearch和OpenSearch仪表板版本2.9或更高版本。看[安装OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/install-opensearch/index/)。
- 安装了2.9或更高版本的异常检测插件。看[安装OpenSearch插件]({{site.url}}{{site.baseurl}}/install-and-configure/plugins)。
- 安装了2.9或更高版本的异常检测仪表板插件。看[管理OpenSearch仪表板插件]({{site.url}}{{site.baseurl}}/install-and-configure/install-dashboards/plugins/) 开始。

## 对异常检测可视化的一般要求

异常检测可视化显示为时间-串联图表可为您提供何时发生异常的快照，从您为可视化配置的不同异常检测器发生。您可以在图表上显示多达10个指标，每个系列可以显示为图表上的一行。

请记住，设置或创建异常检测可视化时，请记住以下要求。可视化：

- 必须是一个[Vizlib线图](https://community.vizlib.com/support/solutions/articles/35000107262-vizlib-line-chart-introduction)
- 必须至少包含一个y-轴度量聚集
- 一定不能没有-y-轴度量聚合类型
- 必须使用x的日期直方图聚合类型-轴桶
- 必须有x-底部的轴
- 必须定义一个x-轴聚集桶
- 必须有一个有效的时间-基于x-轴

## 配置管理设置

用户只能访问，创建或管理其具有权限的资源的异常检测器。OpenSearch和OpenSearch仪表板权限控制了对异常检测仪表板和可视化的访问。它默认为启用，并在**仪表板管理** >**高级设置** >**可视化**。如果设置被禁用，则不会出现在**仪表板管理**。您可以在群集级别禁用设置`opensearch-dashboards.yml` 文件。

## 创建异常检测器

首先，首先创建一个异常检测器：

1. 选择**仪表板** 从OpenSearch仪表板主菜单中。
2. 来自**仪表板** 窗口，选择**创造** 然后选择**仪表板**。
3. 选择**添加现有**，然后从中选择适当的可视化**添加面板** 列表。可视化添加到仪表板中。
4. 从可视化面板中，选择省略号图标（{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/ellipsis-icon.png" class ="inline-icon" alt ="ellipsis icon"/> {:/}）。
5. 来自**选项** 菜单，选择**异常检测** >**添加异常检测器**。
6. 选择**创建新的检测器**。
7. 输入信息**探测器细节** 和**模型功能**。最多允许五个型号功能。
8. 要预览飞行中的可视化，请切换**显示可视化** 按钮。
9. 选择**创建检测器**。创建新的检测器后，将检测器添加到可视化中，如下图所示。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/add-detector.png" alt="Interface of adding a detector" width="800" height="800">

## 将异常检测器添加到可视化中

使用单个接口添加，查看和编辑要与可视化相关联的异常检测器。继续前面的教程中的可视化和仪表板，请按照以下步骤将异常检测器与可视化相关联：
 
1. 选择省略号图标（{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/ellipsis-icon.png" class ="inline-icon" alt ="ellipsis icon"/> {:/}）从可视化面板中，然后选择**异常检测**。
2. 选择**关联检测器**。
3. 来自**选择要关联的检测器** 下拉菜单，选择检测器。仅在下拉菜单中列出了合格的检测器。
4. 查看有关检测器的基本信息。要查看全面的细节，请选择**查看探测器页面** 打开“异常检测插件”页面。
5. 选择**副检测器**。如下图所示，现有检测器现在与可视化相关联。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/anomaly-detect-dashboard.png" alt="Interface and confirmation message of associating a detector" width="800" height="800">

## 刷新可视化

根据阈值设置，可视化会在指定的间隔自动刷新。要手动刷新可视化，请选择**刷新** 仪表板页面上的按钮。

## 下一步

- [了解有关仪表板应用程序的更多信息]({{site.url}}{{site.baseurl}}/dashboards/dashboard/index/)。
- [了解有关呼吸检测的更多信息]({{site.url}}{{site.baseurl}}/observing-your-data/ad/index/)。

