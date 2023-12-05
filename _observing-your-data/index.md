---
layout: default
title: 可观察性
nav_order: 1
has_children: true
nav_exclude: true
redirect_from:
  - /observability-plugin/index/
---

# 可观察性

可观察性是插件和应用程序的收集，可让您可视化数据-通过使用管道处理语言来探索，发现和查询opensearch中存储的数据。

您探索数据的经验可能会有所不同，但是如果您是探索数据以创建可视化的新手，我们建议尝试以下工作流程：

1. 使用一定时间范围探索数据[管道处理语言]({{site.url}}{{site.baseurl}}/search-plugins/sql/ppl/index)。
2. 使用[事件分析]({{site.url}}{{site.baseurl}}/observing-your-data/event-analytics) 转动数据-将事件驱动到可视化中。
  ![样本事件分析视图]({{site.url}}{{site.baseurl}}/images/event-analytics.png)
3. 创造[操作面板]({{site.url}}{{site.baseurl}}/observing-your-data/operational-panels) 并添加可视化以按照自己喜欢的方式比较数据。
  ![样品操作面板视图]({{site.url}}{{site.baseurl}}/images/operational-panel.png)
4. 使用[日志分析]({{site.url}}{{site.baseurl}}/observing-your-data/log-ingestion/) 转换非结构化日志数据。
5. 使用[跟踪分析]({{site.url}}{{site.baseurl}}/observing-your-data/trace/index) 创建痕迹并深入研究数据。
  ![样品跟踪分析视图]({{site.url}}{{site.baseurl}}/images/observability-trace.png)
6. 杠杆作用[笔记本]({{site.url}}{{site.baseurl}}/observing-your-data/notebooks) 结合可以与团队成员共享的不同可视化和代码块。
  ![示例笔记本视图]({{site.url}}{{site.baseurl}}/images/notebooks.png)

