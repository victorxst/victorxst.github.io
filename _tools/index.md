---
layout: default
title: 工具
nav_order: 50
has_children: false
nav_exclude: true
redirect_from:
  - /clients/agents-and-ingestion-tools/index/
---

# OpenSearch工具

本节提供有关OpenSearch的文档-支持工具，包括：

- [代理商和摄取工具](#agents-and-ingestion-tools)
- [OpenSearch CLI](#opensearch-cli)
- [OpenSearch Kubernetes操作员](#opensearch-kubernetes-operator)
- [OpenSearch升级，迁移和比较工具](#opensearch-upgrade-migration-and-comparison-tools)

有关数据PEPPPEP的信息，服务器-用于过滤，丰富，转换，归一化和汇总数据的侧数据收集器，以进行下游分析和可视化，请参阅[数据预先]({{site.url}}{{site.baseurl}}/data-prepper/index/)。

## 代理商和摄取工具

从历史上看，许多流行的代理商和摄入工具都与Elasticsearch OSS合作，例如Beats，Logstash，Fluentd，Fluentbit和Opentelemetry。OpenSearch的目的是继续支持广泛的代理和摄入工具，但并非全部测试或明确添加了OpenSearch兼容性。

作为一种中间兼容性解决方案，OpenSearch具有一个设置，该设置指示群集返回版本7.10.2而不是其实际版本。

如果使用包括版本检查的客户端，例如7.x之间的logstash OSS或FileBeat OSS版本- 7.12.x，启用设置：

```json
PUT _cluster/settings
{
  "persistent": {
    "compatibility": {
      "override_main_response_version": true
    }
  }
}
```

[就像其他设置一样]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/)，另一种选择是将以下行添加到`opensearch.yml` 在每个节点上，然后重新启动节点：

```yml
compatibility.override_main_response_version: true
```

LogStash OSS 8.0引入了一个打破的变化，默认情况下，所有插件以ECS兼容模式运行。如果您使用兼容[OSS客户端](#compatibility-matrices) 您必须覆盖默认值以维持遗产行为：

```yml
ecs_compatibility => disabled
```

### 下载

您可以从[OpenSearch下载](https://opensearch.org/downloads.html)。LogStash输出插件与OpenSearch和Elasticsearch OSS兼容（7.10.2或更低）。

这些是带有OpenSearch兼容性的Beats OSS的最新版本。有关更多信息，请参见下面的兼容性矩阵部分。

- [Filebeat OSS 7.12.1](https://www.elastic.co/downloads/past-releases/filebeat-oss-7-12-1)
- [公制OSS 7.12.1](https://www.elastic.co/downloads/past-releases/metricbeat-oss-7-12-1)
- [PacketBeat OSS 7.12.1](https://www.elastic.co/downloads/past-releases/packetbeat-oss-7-12-1)
- [心跳OSS 7.12.1](https://elastic.co/downloads/past-releases/heartbeat-oss-7-12-1)
- [Winlogbeat OSS 7.12.1](https://www.elastic.co/downloads/past-releases/winlogbeat-oss-7-12-1)
- [AuditBeat OSS 7.12.1](https://elastic.co/downloads/past-releases/auditbeat-oss-7-12-1)

一些用户在这些版本的节拍上报告了与摄入管道的兼容性问题。如果您在OpenSearch中使用摄入管道，请考虑使用Beats的7.10.2版本。
{:.note}


## 兼容矩阵

*斜体*单元格未经测试，但指出理论上应该基于现有信息的值。


### logstash的兼容性矩阵

| | logstash OSS 7.0.0至7.11.x| logstash OSS 7.12.x \*| logstash 7.13.x-7.16.x无openSearch输出插件| logstash 7.13.x-7.16.x带有OpenSearch输出插件| logstash 8.x+带有OpenSearch输出插件
| ：---| :--- | :--- | :--- | :--- | :--- |
| Elasticsearch OSS 7.0.0至7.9.x| *是的*| *是的*| *不*| *是的*| *是的*|
| Elasticsearch OSS 7.10.2| *是的*| *是的*| *不*| *是的*| *是的*|
| ODFE 1.0至1.12| *是的*| *是的*| *不*| *是的*| *是的*|
| ODFE 1.13| *是的*| *是的*| *不*| *是的*| *是的*|
| OpenSearch 1.x至2.x| 是通过版本设置| 是通过版本设置| *不*| *是的*| 是的，使用弹性常见模式设置|

\*与Elasticsearch OSS的大多数当前兼容版本。


### 节拍的兼容性矩阵

| | 击败OSS 7.0.0至7.11.x \*\*| 击败OSS 7.12.x \*| 击败7.13.x|
| :--- | :--- | :--- | :--- |
| Elasticsearch OSS 7.0.0至7.9.x| *是的*| *是的*| 不|
| Elasticsearch OSS 7.10.2| *是的*| *是的*| 不|
| ODFE 1.0至1.12| *是的*| *是的*| 不|
| ODFE 1.13| *是的*| *是的*| 不|
| OpenSearch 1.x至2.x| 是通过版本设置| 是通过版本设置| 不|
| logstash OSS 7.0.0至7.11.x| *是的*| *是的*| *是的*|
| logstash OSS 7.12.x \*| *是的*| *是的*| *是的*|
| logstash 7.13.x带有OpenSearch输出插件| *是的*| *是的*| *是的*|

\*与Elasticsearch OSS的大多数当前兼容版本。

\* \* Beats OSS包括所有Apache 2.0 Beats代理（即FileBeat，MetricBeat，auditBeat，heartbeat，winlogbeat和packetBeat）。

OpenSearch不支持BEATS版本比7.12.x的新版本。如果您必须将环境中的Beats代理更新为较新版本，则可以通过将流量从Beats到LogStash并使用Logstash输出插件来摄取数据以将数据摄取到OpenSearch。
{: .warning }

## OpenSearch CLI

OpenSearch CLI命令行接口（OpenSearch-CLI）可让您从命令行管理OpenSearch Cluster并自动化任务。有关OpenSearch CLI的更多信息，请参阅[OpenSearch CLI]({{site.url}}{{site.baseurl}}/tools/cli/)。

## OpenSearch Kubernetes操作员

OpenSearch Kubernetes操作员是开放的-来源Kubernetes操作员，可以帮助自动化集装箱环境中OpenSearch和OpenSearch仪表板的部署和配置。有关如何使用操作员的信息，请参阅[OpenSearch Kubernetes操作员]({{site.url}}{{site.baseurl}}/tools/k8s-operator/)。

## OpenSearch升级，迁移和比较工具

OpenSearch迁移工具有助于迁移到Opensearch并升级到新版本的OpenSearch。这些可以帮助您设置证明-的-使用Docker容器本地概念环境或使用一个概念环境部署到AWS-单击部署脚本。这使您罚款-调整群集配置并在迁移之前更有效地管理工作负载。

有关OpenSearch迁移工具的更多信息，请参阅该文档[OpenSearch Migration GitHub存储库](https://github.com/opensearch-project/opensearch-migrations/tree/capture-and-replay-v0.1.0)

