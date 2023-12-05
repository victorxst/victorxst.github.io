---
layout: default
title: 指标分析
nav_order: 40
redirect_from:
  - /observing-your-data/prometheusmetrics/
---

# 指标分析
引入2.4
{: .label .label-purple }

从OpenSearch 2.4开始，您可以从OpenSearch中汇总的日志数据中摄入和可视化度量数据，从而使您可以分析和关联跨日志，跟踪和指标的数据。以前，您可以从监视的环境中摄入和可视化日志和痕迹。借助此功能，您可以更详细地观察自己的数字资产，更深入地了解基础设施的健康，并更好地告知您的根本原因分析。

下图显示了从普罗米修斯摄入指标并在仪表板中可视化的过程。

![Prometheus和指标]({{site.url}}{{site.baseurl}}/images/metrics/metricsgif.gif)

## 配置从Prometheus到OpenSearch的数据源连接

您可以通过首先从OpenSearch Dashboards中的Prometheus查看指标[普罗米修斯](https://prometheus.io/) 使用SQL插件进行搜索。

要配置与Prometheus的连接，您可以使用`_datasources` 配置API端点。

以下示例请求配置了没有身份验证的Prometheus数据源：

```json
POST _plugins/_query/_datasources 
{
    "name" : "my_prometheus",
    "connector": "prometheus",
    "properties" : {
        "prometheus.uri" : "http://localhost:9090"
    }
}
```

以下示例请求使用AWS SIGV4身份验证配置Prometheus数据源：

```json
POST _plugins/_query/_datasources
{
    "name" : "my_prometheus",
    "connector": "prometheus",
    "properties" : {
        "prometheus.uri" : "http://localhost:8080",
        "prometheus.auth.type" : "awssigv4",
        "prometheus.auth.region" : "us-east-1",
        "prometheus.auth.access_key" : "{{accessKey}}"
        "prometheus.auth.secret_key" : "{{secretKey}}"
    }
}
```

配置从Prometheus到OpenSearch的连接后，Prometheus指标将显示在仪表板中**可观察性** >**指标分析** 窗口，如下图所示。

![指标UI示例1]({{site.url}}{{site.baseurl}}/images/metrics/metrics1.png)

*有关数据源API的身份验证和授权的更多信息，请参见[GitHub上的数据源文档](https://github.com/opensearch-project/sql/blob/main/docs/user/ppl/admin/datasources.rst)。
*有关Prometheus连接器的更多信息，请参阅[Prometheus连接器](https://github.com/opensearch-project/sql/blob/main/docs/user/ppl/admin/connectors/prometheus_connector.rst) Github页面。

## 基于指标创建可视化

您可以根据Prometheus指标和OpenSearch群集收集的其他指标创建可视化。

要创建可视化，请执行以下操作：

1. 在**可观察性** >**指标分析** >**可用指标**，选择要在可视化中包含的指标。
1. 这些可视化现在可以保存。
1. 来自**指标分析** 窗口，选择**节省**。
1. 当提示**自定义操作仪表板/应用程序**，选择可用选项之一。
1. 可选地，您可以在下面编辑预定义的名称值**公制名称** 适合您需求的字段。
1. 选择**节省**。

下图显示了显示在**可观察性** >**指标分析** 窗户。

![指标UI示例2]({{site.url}}{{site.baseurl}}/images/metrics/metrics2.png)

## 定义用于与Prometheus一起使用的PPL查询

您可以定义[管道处理语言]({{site.url}}{{site.baseurl}}/search-plugins/sql/ppl/index) （PPL）对Prometheus收集的指标的查询。以下示例显示了度量标准-选择具有特定维度的查询：

```
source = my_prometheus.prometheus_http_requests_total | stats avg(@value) by span(@timestamp,15s), handler, code
```

此外，您可以通过执行以下步骤来创建自定义可视化：

1. 来自**事件分析** 窗口，输入您的PPL查询并选择**刷新**。这**Explorer页面** 现在显示。
1. 来自**Explorer页面**， 选择**节省**。
1. 当提示**自定义操作仪表板/应用程序**，选择可用选项之一。
1. 可选地，您可以在下面编辑预定义的名称值**公制名称** 适合您需求的字段。
1. 可选地，您可以选择将可视化作为度量保存。
1. 选择**节省**。

注意：只有包含时间的查询-串联可视化和统计/跨度可以保存为度量，如下图所示。

![指标UI示例3]({{site.url}}{{site.baseurl}}/images/metrics/metrics3.png)

