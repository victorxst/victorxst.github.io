---
layout: default
title: 从logstash迁移
nav_order: 25
redirect_from: 
  - /data-prepper/configure-logstash-data-prepper/
---

# 从logstash迁移

您可以使用LogStash配置来运行数据预先。

如前所述[开始使用数据预先]({{site.url}}{{site.baseurl}}/data-prepper/getting-started/)，您需要使用一个管道配置数据预先`pipelines.yaml` 文件。

另外，如果您有logstash配置`logstash.conf` 配置数据prepper而不是`pipelines.yaml`。

## 支持的插件

从数据Prepper 1.2版本开始时，支持LogStash配置中的以下插件：
* HTTP输入插件
* grok filter插件
* Elasticsearch输出插件
* Amazon Elasticsearch输出插件

## 限制
*除了受支持的插件外，LogStash配置中的所有其他插件都会抛出`Exception` 并且无法运行。
* logStash配置中的条件不受数据Prepper 1.2版本的支持。

## 使用LogStash配置运行数据预先

1. 要安装Data Prepper的Docker映像，请参见在[开始使用数据预先]({{site.url}}{{site.baseurl}}/data-prepper/getting-started#1-installing-data-prepper)。

2. 通过提供您的`logstash.conf` 配置。

```
docker run --name data-prepper -p 4900:4900 -v ${PWD}/logstash.conf:/usr/share/data-prepper/pipelines.conf opensearchproject/data-prepper:latest pipelines.conf
```

这`logstash.conf` 文件转换为`logstash.yaml` 通过将LogStash配置中的插件和属性映射到数据Prepper中的相应插件和属性。
您可以找到转换的`logstash.yaml` 在您存储的同一目录中文件`logstash.conf`。


终端中的以下输出表明数据PEPPER正常运行：

```
INFO  org.opensearch.dataprepper.pipeline.ProcessWorker - log-pipeline Worker: No records received from buffer
```

