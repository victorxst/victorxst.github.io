---
layout: default
title: 跟踪分析
parent: 常见用例
nav_order: 5
---

# 跟踪分析

Trace Analytics允许您收集跟踪数据并自定义一条管道，该管道摄入并转换数据以用于OpenSearch。以下内容概述了数据PEPPPER中的Trace Analytics工作流程，如何配置它以及如何可视化跟踪数据。

## 介绍

当使用Data Prepper作为服务器时-侧面组件要收集跟踪数据，您可以自定义数据预先管道，以摄入和转换用于OpenSearch的数据。转换后，您可以可视化转换的跟踪数据，以与OpenSearch仪表板内部的可观察性插件一起使用。跟踪数据提供了对应用程序性能的可见性，并帮助您获得有关单个痕迹的更多信息。

以下流程图说明了跟踪分析工作流程，从运行OpentElemetry Collector到使用OpenSearch仪表板进行可视化。

<img src="{{site.url}}{{site.baseurl}}/images/data-prepper/trace-analytics/trace-analytics-components.jpg" alt="Trace analyticis component overview">{: .img-fluid}

要监视跟踪分析，您需要在服务环境中设置以下组件：
- 添加**仪器** 到您的应用程序，以便它可以生成遥测数据并将其发送给OpentElemetry Collector。
- 运行**Opentelemetry收集器** 作为Amazon Elastic Kubernetes服务（Amazon EKS）的边缘或Daemonset，Amazon弹性容器服务（Amazon ECS）的Sidecar或Amazon Elastic Compute Compute Cloud（Amazon EC2）上的代理商。您应该将收集器配置为将跟踪数据导出到数据PEPPER。
- 部署**数据预先** 作为开放搜索的摄入收集器。将其配置为将富集的跟踪数据发送到您的OpenSearch群集或Amazon OpenSearch Service Service域。
- 使用**OpenSearch仪表板** 可视化和检测分布式应用程序中的问题。

## 跟踪分析管道

为了监视数据预先数据的跟踪分析，我们提供了三个管道：`entry-pipeline`，`raw-trace-pipeline`， 和`service-map-pipeline`。以下图像概述了管道如何共同工作以监视跟踪分析。

<img src="{{site.url}}{{site.baseurl}}/images/data-prepper/trace-analytics/trace-analytics-pipeline.jpg" alt="Trace analytics pipeline overview">{: .img-fluid}


### OpenTelemetry Trace源
 
这[OpentElemetry源]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/processors/otel-trace-raw/) 接受来自OpenTelemetry收集器的跟踪数据。来源遵循[OpentElemetry协议](https://github.com/open-telemetry/opentelemetry-specification/tree/master/specification/protocol) 并正式支持对GRPC的运输和行业的使用-标准加密（TLS/HTTPS）。

### 处理器

Trace Analytics功能有三个处理器：

* * otel_traces_raw *- * otel_traces_raw *处理器收到的集合[跨度](https://github.com/opensearch-project/data-prepper/blob/fa65e9efb3f8d6a404a1ab1875f21ce85e5c5a6d/data-prepper-api/src/main/java/org/opensearch/dataprepper/model/trace/Span.java) 记录来自[*Otel-痕迹-来源*]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sources/otel-trace/)，并执行迹线的状态处理，提取和完成-团体-相关字段。
* * otel_traces_group *- * otel_traces_group *处理器填充缺少的跟踪-团体-集合中的相关领域[跨度](https://github.com/opensearch-project/data-prepper/blob/298e7931aa3b26130048ac3bde260e066857df54/data-prepper-api/src/main/java/org/opensearch/dataprepper/model/trace/Span.java) 通过查找OpenSearch后端记录。
* * service_map_stateful *  -  * service_map_statefl *处理器对跟踪数据执行所需的预处理，并构建元数据以显示`service-map` 仪表板。


### OpenSearch水槽

OpenSearch提供了一个通用的水槽，将数据写入OpenSearch作为目的地。这[OpenSearch水槽]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sinks/opensearch/) 具有与OpenSearch集群相关的配置选项，例如端点，SSL，用户名/密码，索引名称，索引模板和索引状态管理。

该水槽为跟踪分析功能提供了特定的配置。这些配置允许接收器使用特定于跟踪分析的索引和索引模板。以下OpenSearch索引特定于跟踪分析：

* * Otel-V1-APM-跨度 *  -  * Otel-V1-APM-跨度*索引将输出存储在[otel_traces_raw]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/processors/otel-trace-raw/) 处理器。
* * Otel-V1-APM-服务-地图 *  -  * Otel-V1-APM-服务-MAP*索引存储输出[service_map_state]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/processors/service-map-stateful/) 处理器。

## 跟踪调整

从0.8.x版本开始，Data Prepper支持跟踪分析的垂直和水平缩放。您可以调整单个数据PEPPER实例的大小，以满足工作量的需求并垂直扩展。

您可以使用核心水平扩展[同行前锋]({{site.url}}{{site.baseurl}}/data-prepper/managing-data-prepper/peer-forwarder/) 部署多个数据预先实例以形成集群。这使数据预先实例可以与集群中的实例进行通信，并且是水平扩展部署所必需的。

### 扩展建议

使用以下建议的配置来扩展数据预先缩放。我们建议您根据要求修改参数。我们还建议您监视PEPPPER主机指标和OpenSearch指标，以确保配置按预期工作。

#### 缓冲

数据预先处理处理的跟踪请求总数等于`buffer_size` 值`otel-trace-pipeline` 和`raw-pipeline`。发送给OpenSearch的跟踪请求的总数等于`batch_size` 和`workers` 在`raw-trace-pipeline`。有关有关的更多信息`raw-pipeline`， 看[跟踪分析管道]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/pipelines)。


在更改缓冲区设置时，我们建议以下以下内容：
 * 这`buffer_size` 价值`otel-trace-pipeline` 和`raw-pipeline` 应该是一样的。
 * 这`buffer_size` 应该大于或等于`workers` *`batch_size` 在里面`raw-pipeline`。
 

#### 工人

这`workers` 设置确定Data Prepper用于从缓冲区处理请求的线程数。我们建议您设置`workers` 基于CPU利用率。此值可以高于可用处理器的数量，因为数据Prepper在将数据发送到OpenSearch时会使用大量的输入/输出时间。

#### 堆

通过设置数据来配置数据预先堆`JVM_OPTS` 环境变量。我们建议您将堆值设置为最小值`4` *`batch_size` *`otel_send_batch_size` *`maximum size of indvidual span`。

如[Opentelemetry收集器](#opentelemetry-collector) 部分，设置`otel_send_batch_size` 价值`50` 在您的OpentElemetry集合配置中。

#### 本地磁盘

Data Prepper使用本地磁盘来存储服务地图处理所需的元数据，因此我们建议仅存储以下关键字段：`traceId`，`spanId`，`parentSpanId`，`spanKind`，`spanName`， 和`serviceName`。这`service-map` 插件仅存储两个文件，每个文件存储`window_duration` 秒的数据。例如，用吞吐量进行测试`3000 spans/second` 导致总磁盘使用的`4 MB`。

Data Prepper还使用本地磁盘编写日志。在最新版本的Data Prepper中，您可以将日志重定向到首选的路径。


### AWS CloudFormation模板和Kubernetes/Amazon EKS配置文件

这[AWS云形式](https://github.com/opensearch-project/data-prepper/blob/main/deployment-template/ec2/data-prepper-ec2-deployment-cfn.yaml) 模板提供用户-友好的机制，用于配置缩放属性[跟踪调整](#trace-tuning) 部分。

这[Kubernetes配置文件](https://github.com/opensearch-project/data-prepper/blob/main/examples/dev/k8s/README.md) 和[亚马逊EKS配置文件](https://github.com/opensearch-project/data-prepper/blob/main/deployment-template/eks/README.md) 可用于在集群部署中配置这些属性。

### 基准测试

基准测试是在`r5.xlarge` 带有以下配置的EC2实例：
 
 *`buffer_size`：4096
 *`batch_size`：256
 *`workers`：8
 *`Heap`：10 GB
 
此设置能够处理`2100` 跨度/秒`20` CPU利用率百分比。

## 管道配置

以下各节提供了不同类型的管道以及如何配置每种类型的示例。

### 示例：跟踪分析管道

以下示例演示了如何构建支持该管道的管道[OpenSearch仪表板可观察性插件]({{site.url}}{{site.baseurl}}/observability-plugin/trace/ta-dashboards/)。该管道从OpenTelemetry Collector中获取数据，并使用其他两个管道作为水槽。这两个单独的管道有两个不同的目的，并写入不同的OpenSearch索引。第一个管道为opensearch准备跟踪数据，并丰富跨度文档，并将跨度文档摄入OpenSearch中的跨度索引。第二管道将轨迹汇总到服务映射中，并将服务图文档写入OpenSearch中的服务地图索引。

从数据Prepper版本2.0开始，Data Prepper不再支持`otel_traces_raw_prepper` 处理器。这`otel_traces_raw` 处理器替换`otel_traces_raw_prepper` 处理器并支持Prepper最近的一些数据模型更改。相反，您应该使用`otel_traces_raw` 处理器。请参阅以下YAML文件示例：

```yml
entry-pipeline:
  delay: "100"
  source:
    otel_traces_source:
      ssl: false
  buffer:
    bounded_blocking:
      buffer_size: 10240
      batch_size: 160
  sink:
    - pipeline:
        name: "raw-trace-pipeline"
    - pipeline:
        name: "service-map-pipeline"
raw-pipeline:
  source:
    pipeline:
      name: "entry-pipeline"
  buffer:
    bounded_blocking:
      buffer_size: 10240
      batch_size: 160
  processor:
    - otel_traces_raw:
  sink:
    - opensearch:
        hosts: ["https://localhost:9200"]
        insecure: true
        username: admin
        password: admin
        index_type: trace-analytics-raw
service-map-pipeline:
  delay: "100"
  source:
    pipeline:
      name: "entry-pipeline"
  buffer:
    bounded_blocking:
      buffer_size: 10240
      batch_size: 160
  processor:
    - service_map_stateful:
  sink:
    - opensearch:
        hosts: ["https://localhost:9200"]
        insecure: true
        username: admin
        password: admin
        index_type: trace-analytics-service-map
```

要保持相似的摄入吞吐量和潜伏期，请扩展`buffer_size` 和`batch_size` 按照客户端请求有效载荷中估计的最大批量大小。
{:.tip}

#### 例子：`otel trace`

以下是一个例子`otel-trace-source` .YAML文件带有SSL并启用基本身份验证。请注意，您需要修改您的`otel-collector-config.yaml` 文件以便使用您自己的凭据。

```yaml
source:
  otel_traces_source:
    #record_type: event  # Add this when using Data Prepper 1.x. This option is removed in 2.0
    ssl: true
    sslKeyCertChainFile: "/full/path/to/certfile.crt"
    sslKeyFile: "/full/path/to/keyfile.key"
    authentication:
      http_basic:
        username: "my-user"
        password: "my_s3cr3t"
```

#### 示例：pipeline.yaml

以下是一个例子`pipeline.yaml` 没有SSL的文件，并启用了基本身份验证`otel-trace-pipeline` 管道：

```yaml
otel-trace-pipeline:
  # workers is the number of threads processing data in each pipeline. 
  # We recommend same value for all pipelines.
  # default value is 1, set a value based on the machine you are running Data Prepper
  workers: 8 
  # delay in milliseconds is how often the worker threads should process data.
  # Recommend not to change this config as we want the entry-pipeline to process as quick as possible
  # default value is 3_000 ms
  delay: "100" 
  source:
    otel_traces_source:
      #record_type: event  # Add this when using Data Prepper 1.x. This option is removed in 2.0
      ssl: false # Change this to enable encryption in transit
      authentication:
        unauthenticated:
  buffer:
    bounded_blocking:
       # buffer_size is the number of ExportTraceRequest from otel-collector the data prepper should hold in memeory. 
       # We recommend to keep the same buffer_size for all pipelines. 
       # Make sure you configure sufficient heap
       # default value is 512
       buffer_size: 512
       # This is the maximum number of request each worker thread will process within the delay.
       # Default is 8.
       # Make sure buffer_size >= workers * batch_size
       batch_size: 8
  sink:
    - pipeline:
        name: "raw-trace-pipeline"
    - pipeline:
        name: "entry-pipeline"
raw-pipeline:
  # Configure same as the otel-trace-pipeline
  workers: 8 
  # We recommend using the default value for the raw-pipeline.
  delay: "3000" 
  source:
    pipeline:
      name: "entry-pipeline"
  buffer:
      bounded_blocking:
         # Configure the same value as in entry-pipeline
         # Make sure you configure sufficient heap
         # The default value is 512
         buffer_size: 512
         # The raw processor does bulk request to your OpenSearch sink, so configure the batch_size higher.
         # If you use the recommended otel-collector setup each ExportTraceRequest could contain max 50 spans. https://github.com/opensearch-project/data-prepper/tree/v0.7.x/deployment/aws
         # With 64 as batch size each worker thread could process upto 3200 spans (64 * 50)
         batch_size: 64
  processor:
    - otel_traces_raw:
    - otel_traces_group:
        hosts: [ "https://localhost:9200" ]
        # Change to your credentials
        username: "admin"
        password: "admin"
        # Add a certificate file if you are accessing an OpenSearch cluster with a self-signed certificate  
        #cert: /path/to/cert
        # If you are connecting to an Amazon OpenSearch Service domain without
        # Fine-Grained Access Control, enable these settings. Comment out the
        # username and password above.
        #aws_sigv4: true
        #aws_region: us-east-1
  sink:
    - opensearch:
        hosts: [ "https://localhost:9200" ]
        index_type: trace-analytics-raw
        # Change to your credentials
        username: "admin"
        password: "admin"
        # Add a certificate file if you are accessing an OpenSearch cluster with a self-signed certificate  
        #cert: /path/to/cert
        # If you are connecting to an Amazon OpenSearch Service domain without
        # Fine-Grained Access Control, enable these settings. Comment out the
        # username and password above.
        #aws_sigv4: true
        #aws_region: us-east-1
service-map-pipeline:
  workers: 8
  delay: "100"
  source:
    pipeline:
      name: "entry-pipeline"
  processor:
    - service_map_stateful:
        # The window duration is the maximum length of time the data prepper stores the most recent trace data to evaluvate service-map relationships. 
        # The default is 3 minutes, this means we can detect relationships between services from spans reported in last 3 minutes.
        # Set higher value if your applications have higher latency. 
        window_duration: 180 
  buffer:
      bounded_blocking:
         # buffer_size is the number of ExportTraceRequest from otel-collector the data prepper should hold in memeory. 
         # We recommend to keep the same buffer_size for all pipelines. 
         # Make sure you configure sufficient heap
         # default value is 512
         buffer_size: 512
         # This is the maximum number of request each worker thread will process within the delay.
         # Default is 8.
         # Make sure buffer_size >= workers * batch_size
         batch_size: 8
  sink:
    - opensearch:
        hosts: [ "https://localhost:9200" ]
        index_type: trace-analytics-service-map
        # Change to your credentials
        username: "admin"
        password: "admin"
        # Add a certificate file if you are accessing an OpenSearch cluster with a self-signed certificate  
        #cert: /path/to/cert
        # If you are connecting to an Amazon OpenSearch Service domain without
        # Fine-Grained Access Control, enable these settings. Comment out the
        # username and password above.
        #aws_sigv4: true
        #aws_region: us-east-1
```

您需要修改OpenSearch集群的前面配置，以使配置匹配您的环境。请注意，它有两个`opensearch` 需要修改的水槽。
{:.note}

您必须进行以下更改：
*`hosts`  - 设置为您的主机。
*`username`  - 提供您的OpenSearch用户名。
*`password`  - 提供您的OpenSearch密码。
*`aws_sigv4`  - 如果您使用的是带有AWS签名的Amazon OpenSearch服务，请将此值设置为`true`。它将与默认AWS凭据提供商签署请求。
*`aws_region`  - 如果您使用的是带有AWS签名的Amazon OpenSearch服务，请将此值设置为AWS区域。

有关可用于OpenSearch水槽的其他配置，请参见[数据预先开放搜索水槽]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sinks/opensearch/)。

## Opentelemetry收集器

您需要在服务环境中运行OpenTelemetry收集器。跟随[入门](https://opentelemetry.io/docs/collector/getting-started/#getting-started) 安装opentelemetry收集器。确保使用为数据预先实例配置的出口商配置收集器。以下示例`otel-collector-config.yaml` 文件从各种仪器中接收数据，并将其导出到数据PEPPPER中。

### 示例Otel-集电极-config.yaml文件

以下是一个例子`otel-collector-config.yaml` 文件：

```
receivers:
  jaeger:
    protocols:
      grpc:
  otlp:
    protocols:
      grpc:
  zipkin:

processors:
  batch/traces:
    timeout: 1s
    send_batch_size: 50

exporters:
  otlp/data-prepper:
    endpoint: localhost:21890
    tls:
      insecure: true

service:
  pipelines:
    traces:
      receivers: [jaeger, otlp, zipkin]
      processors: [batch/traces]
      exporters: [otlp/data-prepper]
```

在服务环境中运行OpenTelemetry后，必须配置应用程序以使用OpenTelemetry Collector。OpentElemetry收集器通常与您的应用程序一起运行。

## 下一步和更多信息

这[OpenSearch仪表板可观察性插件]({{site.url}}{{site.baseurl}}/observability-plugin/trace/ta-dashboards/) 文档提供了有关配置Opensearch以查看OpenSearch仪表板中的跟踪分析的其他信息。

有关如何调整和缩放数据预先痕迹分析的更多信息，请参见[跟踪调整](#trace-tuning)。

## 迁移到数据Prepper 2.0

从数据Prepper版本1.4开始，Trace Processing使用Data Prepper的事件模型。这允许管道作者配置其他处理器来修改跨度或跟踪。为了提供迁移路径，Data Prepper版本1.4引入了以下更改：

*`otel_traces_source` 有一个可选的`record_type` 可以设置为`event`。配置后，它将输出事件对象。
*`otel_traces_raw` 替换`otel_traces_raw_prepper` 事件-基于跨度。
*`otel_traces_group` 替换`otel_traces_group_prepper` 事件-基于跨度。

在Data Prepper 2.0版中`otel_traces_source` 只会输出事件。Data Prepper版本2.0也已删除`otel_traces_raw_prepper` 和`otel_traces_group_prepper` 完全。要迁移到数据Prepper 2.0版，您可以使用事件模型配置跟踪管道。
 
