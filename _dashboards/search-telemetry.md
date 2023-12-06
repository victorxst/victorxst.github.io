---
layout: default
title: 搜索遥测
nav_order: 140
---


# 搜索遥测

您可以使用搜索遥测来通过OpenSearch仪表板中的成功或失败来分析搜索请求性能。OpenSearch将遥测数据存储在`.kibana_1` 指数。

由于OpenSearch仪表板有成千上万的并发搜索请求，因此流量繁忙可能会在OpenSearch集群中造成重大负载。

OpenSearch群集在关闭搜索遥测时性能更好。
{: .tip }

## 打开搜索遥测

默认情况下，搜索用法遥测将关闭。要打开它，您需要设置`data.search.usageTelemetry.enabled` 到`true` 在里面`opensearch_dashboards.yml` 文件。

你可以找到[OpenSearch Dashboards yaml文件](https://github.com/opensearch-project/OpenSearch-Dashboards/blob/main/config/opensearch_dashboards.yml) 在OpenSearch中-GitHub上的项目存储库。

打开遥测`opensearch_dashboards.yml` 文件覆盖默认搜索遥测设置`false` 在里面[数据插件配置文件](https://github.com/opensearch-project/OpenSearch-Dashboards/blob/main/src/plugins/data/config.ts)。
{:.note}

### 打开或关闭搜索遥测

下表显示了`data.search.usageTelemetry.enabled` 您可以设置的值`opensearch_dashboards.yml` 打开或关闭搜索遥测。

OpenSearch仪表板YAML值| 搜索遥测状态：开机或关闭
：--- |  ：---
 `true`  | 在
 `false` | 离开
 `none`  | 离开

#### sample opensearch_dashboards.yml启用了遥测

 此OpenSearch Dashboards yaml文件摘录显示遥测设置设置为`true` 打开搜索遥测：

 ```JSON
# 将此设置的值设置为false以抑制
# 搜索用法遥测以减少OpenSearch群集的负载。
 data.search.usagetelemetry.enabled：true
```
