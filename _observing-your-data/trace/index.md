---
layout: default
title: 跟踪分析
nav_order: 40
has_children: true
has_toc: false
redirect_from:
  - /observability-plugin/trace/index/
  - /monitoring-plugins/trace/index/
---

# 跟踪分析

跟踪分析提供了一种摄入和可视化的方法[Opentelemetry](https://opentelemetry.io/) opensearch中的数据。这些数据可以帮助您在分布式应用程序中找到并解决性能问题。

单个操作（例如选择按钮的用户）可以触发一系列扩展的事件。前端可能会调用后端服务，该服务调用另一个服务，该服务查询数据库，处理数据并将其发送到原始服务，该服务将确认发送到前端。

Trace Analytics可以帮助您可视化事件的流程并确定性能问题，如下图所示。

![详细的跟踪视图]({{site.url}}{{site.baseurl}}/images/ta-trace.png)

## 使用Jaeger数据进行跟踪分析

Trace Analytics支持OpenSearch可观察性插件中的Jaeger跟踪数据。如果您使用OpenSearch作为Jaeger Trace Data的后端，则可以使用构建-在跟踪分析功能中。

要设置您的环境以使用跟踪分析，请参见[分析Jaeger Trace数据]({{site.url}}{{site.baseurl}}/observability-plugin/trace/trace-analytics-jaeger/)。

