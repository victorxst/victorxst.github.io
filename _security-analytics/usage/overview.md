---
layout: default
title: 概述页面
parent: 使用安全分析
nav_order: 25
---

# 概述页面

当您选择时**安全分析** 从顶部菜单中，显示概述页面。概述页面由五个部分组成：
*调查结果和警报计数
*最近的警报
*最近的发现
*最常见的检测规则
*探测器

每个部分都为安全分析的每个元素提供了一个摘要说明，以及可让您对每个项目采取行动的控件。

---
## Overview and getting started

概述页面的上部包含两个控制按钮，以刷新信息并开始使用安全分析。您可以选择**刷新** 按钮以刷新页面上的所有信息。

您也可以选择**入门** 链接以扩展使用安全分析窗口的启动，其中包括设置步骤以及控制按钮的摘要，使您可以跳到任何步骤。

<img src ="{{site.url}}{{site.baseurl}}/images/Security/overview.png" alt ="The overview page with getting started quick launch window" width="85%">

*在设置的步骤1中，选择**创建检测器** 定义检测器。
*在步骤2中，选择**查看发现** 转到发现页面。有关此页面的详细信息，请参见[{{site.url}} {{site.baseurl}}/securit-分析/用法/发现/）。
*在步骤3中，选择**查看警报** 转到“安全警报”页面。有关此页面的详细信息，请参见[使用警报]（{{site.url}}} {{site.baseurl}}/security-分析/用法/警报/）。
*在步骤4中，选择**管理规则** 转到规则页面。有关规则的更多信息，请参见[使用规则]（{{stite.url}}} {{site.baseurl}}/security-分析/用法/规则/）。

---
## 调查结果和警报计数

发现和警报计数部分提供了一个图形，显示了有关最新发现的数据。使用**通过...分组** 下拉列表要选择任何一个**所有发现** 或者**日志类型**。

<img src="{{site.url}}{{site.baseurl}}/images/Security/count.png" alt="A graph showing counts for findings and alerts." width="75%">

---
## Recent alerts

最近的警报表显示了按时间，触发名称和警报严重性的最新警报。选择**查看警报** 转到“警报”页面。

<img src ="{{site.url}}{{site.baseurl}}/images/Security/recent-alerts.png" alt ="A table showing the most recent alerts." width="50%">

---
## 最新发现

最近的发现表显示了按时间，规则名称，规则严重性和检测器划分的最新发现。选择**查看所有发现** 转到发现页面。

<img src="{{site.url}}{{site.baseurl}}/images/Security/recent-findings.png" alt="A table showing the most recent findings." width="50%">

---
## Most frequent detection rules

本节提供了检测规则的图形表示，该规则最常触发发现以及它们与其他人的比较（占整体的百分比）。图表表示的规则名称在右侧列出。您可以将悬停在图表上的每种颜色上，以查看有关其代表的检测规则的详细信息。

<img src ="{{site.url}}{{site.baseurl}}/images/Security/rule_graph.png" alt ="The detection rule graph on the Overview page" width="50%">

---
## 探测器

“检测器”部分显示了可用检测器名称，状态（活动/非活动）和日志类型的可用检测器列表。选择**查看所有检测器** 转到探测器页面。选择**创建检测器** 直接转到定义检测器页面。

<img src="{{site.url}}{{site.baseurl}}/images/Security/detector-overview.png" alt="A table showing available detectors." width="50%">


