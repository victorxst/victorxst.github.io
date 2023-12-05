---
layout: default
title: 日志摄入
nav_order: 30
redirect_from:
  - /observability-plugin/log-analytics/
---

# 日志摄入

日志摄入提供了一种将非结构化日志数据转换为结构化数据并摄入opensearch的方法。结构化日志数据允许在搜索事件日志时根据数据格式进行改进的查询和过滤。

## 开始摄入日志

OpenSearch Log摄入由三个组件组成---[数据预先]({{site.url}}{{site.baseurl}}/clients/data-prepper/index/)，，，，[OpenSearch]({{site.url}}{{site.baseurl}}/quickstart/)， 和[OpenSearch仪表板]({{site.url}}{{site.baseurl}}/dashboards/index/)。数据Prepper存储库包含几个[样本应用](https://github.com/opensearch-project/data-prepper/tree/main/examples) 您可以用来开始。

### 基本数据流

![从分布式应用程序到OpenSearch的日志数据流程图]({{site.url}}{{site.baseurl}}/images/la.png)

1. 日志摄入依赖于您将日志集合添加到应用程序的环境中以收集并发送日志数据。

   （在里面[例子](#example) 以下，[fluentbit](https://docs.fluentbit.io/manual/) 用作日志收集器，从文件收集日志数据并将日志数据发送到数据Prepper）。

2. [数据预先]({{site.url}}{{site.baseurl}}/clients/data-prepper/index/) 接收日志数据，将数据转换为结构格式，然后将其索引在OpenSearch群集上。

3. 然后可以通过OpenSearch搜索查询或**发现** opensearch仪表板中的页面。

### 例子

此示例模仿了日志文件的日志条目的写入，然后通过数据预先处理并存储在OpenSearch中。

下载或克隆[数据PEPPER存储库](https://github.com/opensearch-project/data-prepper)。然后导航到`examples/log-ingestion/` 并开放`docker-compose.yml` 在文本编辑器中。该文件包含一个容器：

- [流利的位](https://docs.fluentbit.io/manual/) （（`fluent-bit`）
- 数据prepper（`data-prepper`）
- 一个-节点OpenSearch cluster（`opensearch`）
- OpenSearch仪表板（`opensearch-dashboards`）。

关闭文件并运行`docker-compose up --build` 启动容器。

容器启动后，设置了摄入管道并准备摄入日志数据。这`fluent-bit` 容器配置为从`test.log`。运行以下命令以生成日志数据以发送到日志摄入管道。

```
echo '63.173.168.120 - - [04/Nov/2021:15:07:25 -0500] "GET /search/tag/list HTTP/1.0" 200 5003' >> test.log
```

流利-位会收集日志数据并将其发送给数据Prepper：

```angular2html
[2021/12/02 15:35:41] [ info] [output:http:http.0] data-prepper:2021, HTTP status=200
200 OK
```

数据Prepper将处理日志并索引：

```
2021-12-02T15:35:44,499 [log-pipeline-processor-worker-1-thread-1] INFO  com.amazon.dataprepper.pipeline.ProcessWorker -  log-pipeline Worker: Processing 1 records from buffer
```

这应该导致将单个文档写入openSearch集群`apache-logs` 索引如`log_pipeline.yaml` 文件。

运行以下命令，以查看OpenSearch cluster中的一个原始文档：

```bash
curl -X GET -u 'admin:admin' -k 'https://localhost:9200/apache_logs/_search?pretty&size=1'
```

响应应显示解析的日志数据：

```
    "hits" : [
      {
        "_index" : "apache_logs",
        "_type" : "_doc",
        "_id" : "yGrJe30BgI2EWNKtDZ1g",
        "_score" : 1.0,
        "_source" : {
          "date" : 1.638459307042312E9,
          "log" : "63.173.168.120 - - [04/Nov/2021:15:07:25 -0500] \"GET /search/tag/list HTTP/1.0\" 200 5003",
          "request" : "/search/tag/list",
          "auth" : "-",
          "ident" : "-",
          "response" : "200",
          "bytes" : "5003",
          "clientip" : "63.173.168.120",
          "verb" : "GET",
          "httpversion" : "1.0",
          "timestamp" : "04/Nov/2021:15:07:25 -0500"
        }
      }
    ]
```

可以在OpenSearch仪表板中查看相同的数据，通过访问**发现** 页面并搜索`apache_logs` 指数。请记住，如果这是您第一次搜索索引，则必须在OpenSearch仪表板中创建索引

