---
layout: default
title: 操作面板
nav_order: 60
redirect_from:
  - /observing-your-data/operational-panels/
  - /observability-plugin/operational-panels/
---

# 操作面板

OpenSearch仪表板中的操作面板是使用的可视化集合[管道处理语言]({{site.url}}{{site.baseurl}}/search-plugins/sql/ppl/index) （PPL）查询。

## 开始使用操作面板

如果您想开始使用操作面板而不添加任何数据，请扩展**行动** 菜单，选择**添加样品**，仪表板添加了一套具有保存可视化的操作面板供您探索。

## 创建一个操作面板

创建一个操作面板并添加可视化：

1. 来自**添加可视化** 下拉菜单，选择**选择现有的可视化** 或者**创建新的可视化**，这带你到[事件分析]({{site.url}}{{site.baseurl}}/observing-your-data/event-analytics) 资源管理器，您可以在其中使用PPL创建可视化。
1. 如果您要添加已经存在的可视化，请从下拉菜单中选择可视化。
1. 选择**添加**。

![样品操作面板]({{site.url}}{{site.baseurl}}/images/operational-panel.png)

要在操作面板中搜索特定的可视化，请使用PPL查询搜索您已经添加到面板中的数据。

