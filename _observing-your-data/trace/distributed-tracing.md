---
layout: default
title: 分散的追踪
parent: 跟踪分析
nav_order: 65
---

# 分布式跟踪
这是一个实验特征，不建议在生产环境中使用。有关功能进度或要留下反馈的更新，请参阅关联[Github问题](https://github.com/opensearch-project/OpenSearch/issues/6750)。
{: .warning}

分布式跟踪用于监视和调试分布式系统。您可以通过系统跟踪请求流，并确定性能瓶颈和错误。_trace_是一个完整的结尾-到-当请求流过分布式系统时，请求的结束路径。它代表特定操作的旅程，因为它遍历了分布式体系结构中的各种组件和服务。在分布式跟踪中，单个跟踪包含一系列称为_spans_的标记时间间隔。跨度具有开始和结束时间，可能包括其他元数据（如日志或标签），以帮助对发生的事情进行分类。

将分布式跟踪用于以下目的：

- **优化性能：** 识别和解决瓶颈，减少应用程序的延迟。
- **故障排除错误：** 快速查明分布式系统中错误或意外行为的来源。
- **分配资源：** 通过了解不同服务的使用模式来优化资源分配。
- **可视化服务依赖性：** 可视化服务之间的依赖关系，帮助您管理体系结构。

## 分布式跟踪管道

OpenSearch提供了一个分布式的跟踪管道，可用于摄入，处理和可视化数据和警报功能的数据。[Opentelemetry](https://opentelemetry.io/) 是开放的-源可观察性框架提供了一组API，库，代理和收集器，用于生成，捕获和导出遥测数据。分布式跟踪管道由以下组件组成：

- **仪器：** 使用OpenTelemetry SDK仪器启动您的应用程序代码。
- **传播：** 将跟踪上下文注射到请求中，当它们通过您的系统传播时。
- **收藏：** 从您的应用程序中收集跟踪数据。
- **加工：** 从多个来源汇总跟踪数据，并用其他元数据丰富它。
- **出口：** 将跟踪数据发送到后端进行存储和分析。

opensearch通常被选为存储跟踪数据的水槽。

## 跟踪分析

OpenSearch提供了`trace-analytics` 用于实时可视化跟踪数据的插件。该插件包括用于分析跟踪数据的预制仪表板，例如服务图，延迟直方图和错误率。借助OpenSearch的分布式跟踪管道，您可以快速识别应用程序中的瓶颈和错误。看到[跟踪分析]({{site.url}}{{site.baseurl}}/observing-your-data/trace/index/) 文档以获取更多信息。

## 开始

从OpenSearch 2.10开始，分布式跟踪功能是实验性的。要开始使用分布式跟踪功能，您需要首先使用`opensearch.experimental.feature.telemetry.enabled` 功能标志并随后使用动态设置激活示踪剂`telemetry.tracer.enabled`。启用此功能时，请谨慎行事，因为它可以消耗系统资源。有关启用和配置分布式跟踪的详细信息，包括-以下各节描述了需求故障排除和请求采样。

### 使用TARBALL在节点上启用标志

使用新的Java虚拟机（JVM）参数将启用标志切换`OPENSEARCH_JAVA_OPTS` 或IN`config/jvm.options`。

#### 选项1：启用实验功能标志`opensearch.yml` 文件

1. 更改为OpenSearch安装的顶级目录：

```bash
cd \path\to\opensearch
```

2. 打开OpenSearch配置文件夹，然后打开`opensearch.yml` 用文本编辑器文件。
3. 添加以下行：

```bash
opensearch.experimental.feature.telemetry.enabled=true
```
{% include copy.html %}

4. 保存更改并关闭文件。

#### 选项2：修改JVM.Options

将以下行添加到`config/jvm.options` 在启动OpenSearch流程以启用功能及其依赖性之前：

```bash
-Dopensearch.experimental.feature.telemetry.enabled=true
```
{% include copy.html %}

运行OpenSearch：

```bash
./bin/opensearch
```
{% include copy.html %}

#### 选项3：从环境变量启用

作为直接修改的替代方案`config/jvm.options`，您可以使用环境变量来定义属性。当您启动OpenSearch或设置环境变量时，您可以在单个命令中启用此功能。

要在启动OpenSearch时添加这些标志，请运行以下命令：

```bash
OPENSEARCH_JAVA_OPTS="-Dopensearch.experimental.feature.telemetry.enabled=true" ./opensearch-2.9.0/bin/opensearch
```
{% include copy.html %}

要分别定义环境变量，在运行OpenSearch之前，运行以下命令：

```bash
export OPENSEARCH_JAVA_OPTS="-Dopensearch.experimental.feature.telemetry.enabled=true"
 ./bin/opensearch
```
{% include copy.html %}

### 使用Docker容器启用

如果您正在运行Docker，请将以下行添加到`docker-compose.yml` 在下面`environment`：

```bash
OPENSEARCH_JAVA_OPTS="-Dopensearch.experimental.feature.telemetry.enabled=true"
```
{% include copy.html %}

### 启用开发开发

要启用分布式跟踪功能，您必须首先将正确的属性添加到`run.gradle` 在构建OpenSearch之前。看到[开发人员指南](https://github.com/opensearch-project/OpenSearch/blob/main/DEVELOPER_GUIDE.md#gradle-build) 有关如何使用Gradle构建OpenSearch的信息。

将以下属性添加到`run.gradle` 启用该功能：

```json
testClusters {
  runTask {
    testDistribution = 'archive'
 if (numZones > 1) numberOfZones = numZones
    if (numNodes > 1) numberOfNodes = numNodes
    systemProperty 'opensearch.experimental.feature.telemetry.enabled', 'true'
    }
 }
 ```
 {% include copy.html %}

### Enable distributed tracing

Once you've enabled the feature flag, you can enable the tracer (which is disabled by default) by using the following dynamic setting that enables tracing in the running cluster:

```bash
telemetry.tracer.enabled = true
```
{% include copy.html %}

## Install the OpenSearch OpenTelemetry plugin

The OpenSearch distributed tracing framework aims to support various telemetry solutions through plugins. The OpenSearch OpenTelemetry plugin `telemetry-otel` is available and must be installed to enable tracing. The following guide provides you with the installation instructions.

### Exporters

Currently, the distributed tracing feature generates traces and spans for HTTP requests and a subset of transport requests. These traces and spans are initially kept in memory using the OpenTelemetry `BatchSpanProcessor` and are then sent to an exporter based on configured settings. The following are the key components:

1. **Span processors:** As spans conclude on the request path, OpenTelemetry provides them to the `SpanProcessor` for processing and exporting. The OpenSearch distributed tracing framework uses the `BatchSpanProcessor`, which batches spans for specific configurable intervals and then sends them to the exporter. The following configurations are available for the `BatchSpanProcessor`:
    - `telemetry.otel.tracer.exporter.max_queue_size`: Defines the maximum queue size. When the queue reaches this value, it will be written to the exporter. Default is `2048`. 
    - `telemetry.otel.tracer.exporter.delay`: Defines the delay---a time period after which spans in the queue will be flushed, even if there are not enough spans to fill the `max_queue_size`. Default is `2 seconds`.
    - `telemetry.otel.tracer.exporter.batch_size`: Configures the maximum batch size for each export to reduce input/output. This value should always be less than the `max_queue_size`. Default is `512`.
2. **Exporters:** Exporters are responsible for persisting the data. OpenTelemetry provides several out-of-the-box exporters, and OpenSearch supports the following:
    - `LoggingSpanExporter`: Exports spans to a log file, generating a separate file in the logs directory `_otel_traces.log`. Default is `telemetry.otel.tracer.span.exporter.class=io.opentelemetry.exporter.logging.LoggingSpanExporter`.
    - `OtlpGrpcSpanExporter`: Exports spans through gRPC. To use this exporter, you need to install the `otel-collector` on the node. By default, it writes to the http://localhost:4317/ endpoint. To use this exporter, set the following static setting: `telemetry.otel.tracer.span.exporter.class=org.opensearch.telemetry.tracing.exporter.OtlpGrpcSpanExporterProvider`.
    - `LoggingSpanExporter`: Exports spans to a log file, generating a separate file in the logs directory `_otel_traces.log`. Default is `telemetry.otel.tracer.span.exporter.class=io.opentelemetry.exporter.logging.LoggingSpanExporter`.

### Sampling

Distributed tracing can generate numerous spans, consuming system resources unnecessarily. To reduce the number of traces, also called _samples_, you can configure different sampling thresholds. By default, sampling is configured to include only 1% of all HTTP requests. Sampling has the following types:

1. **Head sampling:** Sampling decisions are made before initiating the root span of a request. OpenSearch supports two head sampling methods:
    - **Probabilistic:** A blanket limit on incoming requests, dynamically adjustable with the `telemetry.tracer.sampler.probability` setting. This setting ranges between 0 and 1. Default is 0.01, which indicates that 1% of incoming requests are sampled.
    - **On-Demand:** For debugging specific requests, you can send the `trace=true` attribute as part of the HTTP headers, causing those requests to be sampled regardless of the probabilistic sampling setting.
2. **Tail sampling:** To configure tail sampling, follow the instructions in [OpenTelemetry tail sampling documentation](https://opentelemetry.io/docs/concepts/sampling/#tail-sampling). Configuration depends on the type of collector you choose.

### Collection of spans

The `SpanProcessor` writes spans to the exporter, and the choice of exporter defines the endpoint, which can be logs or gRPC. To collect spans by using gRPC, you need to configure the collector as a sidecar process running on each OpenSearch node. From the collectors, these spans can be written to the sink of your choice, such as Jaeger, Prometheus, Grafana, or FileStore, for further analysis.

