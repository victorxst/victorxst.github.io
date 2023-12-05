---
layout: default
title: 分析Jaeger Trace数据
parent: 跟踪分析
nav_order: 55
redirect_from:
  - /observability-plugin/trace/trace-analytics-jaeger/
---

# 分析Jaeger Trace数据

引入2.5
{: .label .label-purple }

现在，OpenSearch可观察性插件中的跟踪分析功能支持Jaeger跟踪数据。如果您使用OpenSearch作为Jaeger Trace Data的后端，则可以使用构建-在跟踪分析功能中。这提供了对OpenTelemetry（Otel）跟踪数据的支持。

执行跟踪分析时，您可以从两个数据源中选择：

- **数据预先**  - 通过数据Prepper摄入的数据将数据摄入
- **Jaeger**  - 跟踪存储在OpenSearch中作为后端的数据

如果将Jaeger跟踪数据存储在OpenSearch中，则可以使用构建的-在痕量分析功能中可以分析错误率和延迟。您还可以过滤跟踪并分析跟踪的跨度详细信息，以查明所有服务问题。

当您将jaeger数据摄入OpenSearch时，它将被存储在与Otel不同的索引中-通过Data Prepper运行数据时，生成的索引会创建。使用OpenSearch仪表板中的数据源选择器指示要执行跟踪分析的数据源。

您可以分析的Jaeger跟踪数据包括跨度数据以及服务和操作端点数据。<！-- 需要更多信息以获取下一个版本。添加如何配置以进行跨度分析。Jaeger跨度数据分析需要一些配置。-->

默认情况下，每次您摄入Jaeger的数据时，都会在当天创建一个单独的索引。

要了解有关Jaeger数据跟踪的更多信息，请参阅[Jaeger](https://www.jaegertracing.io/) 文档。

## 数据摄入要求

要在Jaeger数据上执行跟踪分析，您需要配置错误功能。

摄入openSearch的Jaeger数据必须具有环境变量`ES_TAGS_AS_FIELDS_ALL` 设置`true` 错误。如果未以这种格式摄入数据，则该数据将不适用于错误，并且使用OpenSearch的Trace Analytics中的痕迹将无法使用错误数据。

### 关于用Jaeger索引摄入的数据

痕量分析非-Jaeger Data使用命名约定的Otel索引`otel-v1-apm-span-*` 或者`otel-v1-apm-service-map*`。

Jaeger索引遵循命名约定`jaeger-span-*` 或者`jaeger-service-*`。

## 设置OpenSearch使用Jaeger数据

以下部分提供了一个示例Docker组成的文件，该文件包含为跟踪分析启用错误所需的配置。

### 步骤1：运行Docker撰写文件

使用以下Docker组合文件启用Jaeger数据以进行跟踪分析。设置`ES_TAGS_AS_FIELDS_ALL` 环境变量设置为`true` 为了使错误添加到跟踪数据。

复制以下docker撰写文件并将其另存为`docker-compose.yml`：

```
version: '3'
services:
  opensearch-node1: # This is also the hostname of the container within the Docker network (i.e. https://opensearch-node1/)
    image: opensearchproject/opensearch:latest # Specifying the latest available image - modify if you want a specific version
    container_name: opensearch-node1
    environment:
      - cluster.name=opensearch-cluster # Name the cluster
      - node.name=opensearch-node1 # Name the node that will run in this container
      - discovery.seed_hosts=opensearch-node1,opensearch-node2 # Nodes to look for when discovering the cluster
      - cluster.initial_cluster_manager_nodes=opensearch-node1,opensearch-node2 # Nodes eligible to serve as cluster manager
      - bootstrap.memory_lock=true # Disable JVM heap memory swapping
      - "OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m" # Set min and max JVM heap sizes to at least 50% of system RAM
    ulimits:
      memlock:
        soft: -1 # Set memlock to unlimited (no soft or hard limit)
        hard: -1
      nofile:
        soft: 65536 # Maximum number of open files for the opensearch user - set to at least 65536
        hard: 65536
    volumes:
      - opensearch-data1:/usr/share/opensearch/data # Creates volume called opensearch-data1 and mounts it to the container
    ports:
      - "9200:9200"
      - "9600:9600"
    networks:
      - opensearch-net # All of the containers will join the same Docker bridge network

  opensearch-node2:
    image: opensearchproject/opensearch:latest # This should be the same image used for opensearch-node1 to avoid issues
    container_name: opensearch-node2
    environment:
      - cluster.name=opensearch-cluster
      - node.name=opensearch-node2
      - discovery.seed_hosts=opensearch-node1,opensearch-node2
      - cluster.initial_cluster_manager_nodes=opensearch-node1,opensearch-node2
      - bootstrap.memory_lock=true
      - "OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    volumes:
      - opensearch-data2:/usr/share/opensearch/data
    networks:
      - opensearch-net
  opensearch-dashboards:
    image: opensearchproject/opensearch-dashboards:latest # Make sure the version of opensearch-dashboards matches the version of opensearch installed on other nodes
    container_name: opensearch-dashboards
    ports:
      - 5601:5601 # Map host port 5601 to container port 5601
    expose:
      - "5601" # Expose port 5601 for web access to OpenSearch Dashboards
    environment:
      OPENSEARCH_HOSTS: '["https://opensearch-node1:9200","https://opensearch-node2:9200"]' # Define the OpenSearch nodes that OpenSearch Dashboards will query
    networks:
      - opensearch-net

  jaeger-collector:
    image: jaegertracing/jaeger-collector:latest
    ports:
      - "14269:14269"
      - "14268:14268"
      - "14267:14267"
      - "14250:14250"
      - "9411:9411"
    networks:
      - opensearch-net
    restart: on-failure
    environment:
      - SPAN_STORAGE_TYPE=opensearch
      - ES_TAGS_AS_FIELDS_ALL=true
      - ES_USERNAME=admin
      - ES_PASSWORD=admin
      - ES_TLS_SKIP_HOST_VERIFY=true
    command: [
      "--es.server-urls=https://opensearch-node1:9200",
      "--es.tls.enabled=true",
    ]
    depends_on:
      - opensearch-node1

  jaeger-agent:
    image: jaegertracing/jaeger-agent:latest
    hostname: jaeger-agent
    command: ["--reporter.grpc.host-port=jaeger-collector:14250"]
    ports:
      - "5775:5775/udp"
      - "6831:6831/udp"
      - "6832:6832/udp"
      - "5778:5778"
    networks:
      - opensearch-net
    restart: on-failure
    environment:
      - SPAN_STORAGE_TYPE=opensearch
    depends_on:
      - jaeger-collector

  hotrod:
    image: jaegertracing/example-hotrod:latest
    ports: 
      - "8080:8080"
    command: ["all"]
    environment:
      - JAEGER_AGENT_HOST=jaeger-agent
      - JAEGER_AGENT_PORT=6831
    networks:
      - opensearch-net
    depends_on:
      - jaeger-agent

volumes:
  opensearch-data1:
  opensearch-data2:

networks:
  opensearch-net:
```

### 步骤2：开始群集

运行以下命令以部署Docker组成YAML文件：

```
docker compose up -d
```
要停止群集，请运行以下命令：

``` 
docker compose down
```

### 步骤3：生成样本数据

使用Docker文件中提供的示例应用程序生成数据。运行Docker撰写文件后，它将在本地主机端口8080上运行示例应用程序。要打开应用程序，请访问http：// localhost：8080。

![服务清单]({{site.url}}{{site.baseurl}}/images/trace-analytics/sample-app.png)

在示例应用程序中，HOT R.O.D.，选择任何生成数据的按钮。现在，您可以在仪表板中查看跟踪数据。

### 步骤4：在OpenSearch仪表板中查看跟踪数据

生成Jaeger跟踪数据后，您可以在仪表板中查看它。

去**跟踪分析** 在[http：// localhost：5601/app/可观察性-仪表板#/trace_analytics/home](http://localhost:5601/app/observability-dashboards#/trace_analytics/home)。

## 在OpenSearch仪表板中使用跟踪分析

为了分析仪表板中的Jaeger跟踪数据，首先设置Trace Analytics功能。要开始，请参阅[开始进行跟踪分析]({{site.url}}{{site.baseurl}}/observability-plugin/trace/get-started/)。

### 数据源

执行跟踪分析时，您可以将数据prepper或Jaeger指定为数据源。
从仪表板，去**可观察性>跟踪分析** 然后选择Jaeger。

![选择数据源]({{site.url}}{{site.baseurl}}/images/trace-analytics/select-data.png)

## 仪表板视图

选择jaeger作为数据源后，您可以查看所有索引数据**仪表板** 查看，包括**错误率** 和**吞吐量**。

### 错误率

您可以查看跟踪错误计数随着时间的流逝**仪表板** 查看，还可以看到具有非非服务和运营的前五名组合-零错误率。

![错误率]({{site.url}}{{site.baseurl}}/images/trace-analytics/error-rate.png)

### 吞吐量

和**吞吐量** 选择，您可以随着时间的推移看到Jaeger索引痕迹的吞吐量。

您可以从**前5名服务和操作延迟** 列出并查看详细的跟踪数据。

![吞吐量]({{site.url}}{{site.baseurl}}/images/trace-analytics/throughput.png)

您还可以看到延迟最高的服务和操作的组合。

如果您选择了服务和操作名称的条目之一，然后转到**迹线** 列选择跟踪，它将自动将服务和操作作为过滤器添加。

## 迹线

在**迹线**，您可以看到列表中每个单独的跟踪ID的过滤服务和操作的延迟和错误。

![选择数据源]({{site.url}}{{site.baseurl}}/images/trace-analytics/service-trace-data.png)

如果选择单个跟踪ID，则可以看到有关跟踪的更多详细信息，例如**服务花费的时间** 和**跨度**。您还可以以JSON格式查看索引有效载荷。

![选择数据源]({{site.url}}{{site.baseurl}}/images/trace-analytics/trace-details.png)

## 服务

您还可以查看每个服务的单个错误率和延迟。去**可观察性>跟踪分析>服务**。在**服务**，您可以看到列表中每个服务的平均延迟，错误率，吞吐量和跟踪。

![服务清单]({{site.url}}{{site.baseurl}}/images/trace-analytics/services-jaeger.png)

