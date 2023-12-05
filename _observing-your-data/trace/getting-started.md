---
layout: default
title: 入门
parent: 跟踪分析
nav_order: 1
redirect_from:
  - /observability-plugin/trace/get-started/
  - /monitoring-plugins/trace/get-started/
---

# 开始进行跟踪分析

OpenSearch Trace Analytics由两个组件组成---数据Prepper和Trace Analytics OpenSearch仪表板插件。数据Prepper存储库包含几个[样本应用](https://github.com/opensearch-project/data-prepper/tree/main/examples) 您可以用来开始。

## 基本数据流

![从分布式应用程序到OpenSearch的数据流程图]({{site.url}}{{site.baseurl}}/images/ta.svg)

1. 跟踪分析依赖于您将仪器添加到应用程序并生成跟踪数据。这[OpentElemetry文档](https://opentelemetry.io/docs/) 包含许多可以帮助您入门的编程语言的示例应用程序，包括Java，Python，Go和JavaScript。

   （在里面[Jaeger Hotrod](#jaeger-hotrod) 下面的示例，一个额外的组件，即jaeger代理，与应用程序一起运行，并将数据发送给OpentElemetry Collector，但该概念相似。）

1. 这[Opentelemetry收集器](https://opentelemetry.io/docs/collector/getting-started/) 从应用程序中接收数据，并将其格式化为OpenTelemetry数据。

1. [数据预先]({{site.url}}{{site.baseurl}}/clients/data-prepper/index/) 处理opentelemetry数据，将其转换为用于OpenSearch，并将其索引在OpenSearch集群上。

1. 这[跟踪分析opensearch仪表板插件]({{site.url}}{{site.baseurl}}/observing-your-data/trace/ta-dashboards/) 显示数据几乎真实-时间是一系列图表和表格，重点是服务体系结构，延迟，错误率和吞吐量。

## Jaeger Hotrod

一种迹线分析样本应用程序是Jaeger Hotrod演示，该演示通过分布式应用模仿数据流。

下载或克隆[数据PEPPER存储库](https://github.com/opensearch-project/data-prepper)。然后导航到`examples/jaeger-hotrod/` 并开放`docker-compose.yml` 在文本编辑器中。该文件包含一个用于每个元素的容器[基本数据流](#basic-flow-of-data)：

- 分布式应用程序（`jaeger-hot-rod`）与Jaeger代理商（`jaeger-agent`）
- 这[Opentelemetry收集器](https://opentelemetry.io/docs/collector/getting-started/) （（`otel-collector`）
- 数据prepper（`data-prepper`）
- 一个-节点OpenSearch cluster（`opensearch`）
- OpenSearch仪表板（`opensearch-dashboards`）。

关闭文件并运行`docker-compose up --build`。容器启动后，导航到`http://localhost:8080` 在网络浏览器中。

![Hotrod Web界面]({{site.url}}{{site.baseurl}}/images/hot-rod.png)

单击Web界面中的一个按钮之一，将请求发送到应用程序。每个请求始于构成应用程序的服务跨越一系列操作。从控制台日志中，您可以看到这些操作共享相同`trace-id`，它使您可以将请求中的所有操作跟踪为单个 *Trace *：

```
jaeger-hot-rod  | http://0.0.0.0:8081/customer?customer=392
jaeger-hot-rod  | 2020-11-19T16:29:53.425Z	INFO	frontend/server.go:92	HTTP request received	{"service": "frontend", "trace_id": "12091bd60f45ea2c", "span_id": "12091bd60f45ea2c", "method": "GET", "url": "/dispatch?customer=392&nonse=0.6509021735471818"}
jaeger-hot-rod  | 2020-11-19T16:29:53.426Z	INFO	customer/client.go:54	Getting customer{"service": "frontend", "component": "customer_client", "trace_id": "12091bd60f45ea2c", "span_id": "12091bd60f45ea2c", "customer_id": "392"}
jaeger-hot-rod  | 2020-11-19T16:29:53.430Z	INFO	customer/server.go:67	HTTP request received	{"service": "customer", "trace_id": "12091bd60f45ea2c", "span_id": "252ff7d0e1ac533b", "method": "GET", "url": "/customer?customer=392"}
jaeger-hot-rod  | 2020-11-19T16:29:53.430Z	INFO	customer/database.go:73	Loading customer{"service": "customer", "component": "mysql", "trace_id": "12091bd60f45ea2c", "span_id": "252ff7d0e1ac533b", "customer_id": "392"}
```

这些操作也有`span_id`。*跨度*是单个服务的工作单位。每个跟踪包含一些跨度。应用程序开始处理请求后不久，您可以看到OpenTelemetry Collector开始导出跨度：

```
otel-collector  | 2020-11-19T16:29:53.781Z	INFO	loggingexporter/logging_exporter.go:296	TraceExporter	{"#spans": 1}
otel-collector  | 2020-11-19T16:29:53.787Z	INFO	loggingexporter/logging_exporter.go:296	TraceExporter	{"#spans": 3}
```

然后，数据预先处理来自OpenTelemetry Collector的数据并将其索引：

```
data-prepper  | 1031918 [service-map-pipeline-process-worker-2-thread-1] INFO  com.amazon.dataprepper.pipeline.ProcessWorker  –  service-map-pipeline Worker: Processing 3 records from buffer
data-prepper  | 1031923 [entry-pipeline-process-worker-1-thread-1] INFO  com.amazon.dataprepper.pipeline.ProcessWorker  –  entry-pipeline Worker: Processing 1 records from buffer
```

最后，您可以看到响应索引请求的OpenSearch节点。

```
node-0.example.com  | [2020-11-19T16:29:55,064][INFO ][o.e.c.m.MetadataMappingService] [9fb4fb37a516] [otel-v1-apm-span-000001/NGYbmVD9RmmqnxjfTzBQsQ] update_mapping [_doc]
node-0.example.com  | [2020-11-19T16:29:55,267][INFO ][o.e.c.m.MetadataMappingService] [9fb4fb37a516] [otel-v1-apm-span-000001/NGYbmVD9RmmqnxjfTzBQsQ] update_mapping [_doc]
```

在新的终端窗口中，运行以下命令，以查看OpenSearch集群中的一个原始文档：

```bash
curl -X GET -u 'admin:admin' -k 'https://localhost:9200/otel-v1-apm-span-000001/_search?pretty&size=1'
```

导航`http://localhost:5601` 在网络浏览器中选择**跟踪分析**。您可以在Jaeger Hotrod Web界面中看到单击的结果-服务体系结构的编码地图，以及可以使用的跟踪ID列表，可用于对单个操作进行深入研究。

如果您看不到跟踪，请调整OpenSearch仪表板中的时间表。有关使用插件的更多信息，请参阅[OpenSearch仪表板插件]({{site.url}}{{site.baseurl}}/observing-your-data/trace/ta-dashboards/)。

