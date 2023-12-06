---
layout: default
title: 入门
nav_order: 5
redirect_from:
  - /clients/data-prepper/get-started/
---

# 开始使用数据预先

Data Prepper是一个独立的组件，而不是OpenSearch插件，可将数据转换为供OpenSearch使用。它并没有捆绑-在-一个OpenSearch安装软件包。

如果您是从开放的发行数据中迁移的，请参见[从开放式发行中迁移]({{site.url}}{{site.baseurl}}/data-prepper/migrate-open-distro/)。
{: .note}

## 1.安装数据预先

安装数据预先的方法有两种：您可以运行Docker映像或从源构建。

使用Data Prepper的最简单方法是运行Docker Image。我们建议您使用这种方法[Docker](https://www.docker.com) 可用的。运行以下命令：

```
docker pull opensearchproject/data-prepper:latest
```
{% include copy.html %}

如果您有需要您从源头构建的特殊要求，或者您想贡献，请参阅[开发人员指南](https://github.com/opensearch-project/data-prepper/blob/main/docs/developer_guide.md)。

## 2.配置数据预先

运行数据预先实例需要两个配置文件。可选，您可以配置Log4J 2配置文件。看[配置log4j]({{site.url}}{{site.baseurl}}/data-prepper/managing-data-prepper/configuring-log4j/) 了解更多信息。以下列表描述了每个配置文件的目的：

*`pipelines.yaml`：此文件描述了要运行的数据管道，包括来源，处理器和接收器。
*`data-prepper-config.yaml`：此文件包含数据预先服务器的设置，使您可以与曝光数据Prepper Server API进行交互。
*`log4j2-rolling.properties` （可选）：此文件包含log4j 2配置选项，可以是JSON，YAML，XML或.properties文件类型。

对于早于2.0的数据预先版本，`.jar` 文件期望管道配置文件路径之后是服务器配置文件路径。请参阅以下配置路径示例：

```
java -jar data-prepper-core-$VERSION.jar pipelines.yaml data-prepper-config.yaml
```

可选，您可以添加`"-Dlog4j.configurationFile=config/log4j2.properties"` 到命令传递自定义log4j 2配置文件。如果您不提供属性文件，则数据将默认为默认为`log4j2.properties` 文件中的文件`shared-config` 目录。


从数据Prepper 2.0开始，您可以使用以下内容启动数据预先`data-prepper` 不需要任何其他命令行参数的脚本：

```
bin/data-prepper
```

配置文件是从应用程序主目录中的特定子目录中读取的：
1. `pipelines/`：用于管道配置。管道配置可以写在一个或多个YAML文件中。
2. `config/data-prepper-config.yaml`：用于数据Prepper服务器配置。

您可以提供自己的管道配置文件路径，然后提供服务器配置文件路径。但是，将来的版本将不支持此方法。请参阅以下示例：
```
bin/data-prepper pipelines.yaml data-prepper-config.yaml
```

log4j 2配置文件是从`config/log4j2.properties` 位于应用程序主目录中的文件。

要配置数据PEPPPEP，请参阅每种用例的以下信息：

*[跟踪分析]({{site.url}}{{site.baseurl}}/data-prepper/common-use-cases/trace-analytics/)：了解如何收集跟踪数据并自定义摄入和转换数据的管道。
*[日志分析]({{site.url}}{{site.baseurl}}/data-prepper/common-use-cases/log-analytics/)：了解如何为日志可观察性设置数据预备。

## 3.定义管道

创建一个名为Prepper管道文件的数据`pipelines.yaml` 使用以下配置：

```yml
simple-sample-pipeline:
  workers: 2
  delay: "5000"
  source:
    random:
  sink:
    - stdout:
```
{% include copy.html %}

## 4.运行数据预先

使用管道配置yaml运行以下命令。

```bash
docker run --name data-prepper \
    -v /${PWD}/pipelines.yaml:/usr/share/data-prepper/pipelines/pipelines.yaml \
    opensearchproject/data-prepper:latest
    
```
{% include copy.html %}

上面的示例管道配置演示了一个简单的管道（用源）（`random`）将数据发送到接收器（`stdout`）。有关更高级管道配置的示例，请参见[管道]({{site.url}}{{site.baseurl}}/clients/data-prepper/pipelines/)。

启动数据预先启动后，您应该在几秒钟后看到日志输出和一些UUID：

```yml
2021-09-30T20:19:44,147 [main] INFO  com.amazon.dataprepper.pipeline.server.DataPrepperServer - Data Prepper server running at :4900
2021-09-30T20:19:44,681 [random-source-pool-0] INFO  com.amazon.dataprepper.plugins.source.RandomStringSource - Writing to buffer
2021-09-30T20:19:45,183 [random-source-pool-0] INFO  com.amazon.dataprepper.plugins.source.RandomStringSource - Writing to buffer
2021-09-30T20:19:45,687 [random-source-pool-0] INFO  com.amazon.dataprepper.plugins.source.RandomStringSource - Writing to buffer
2021-09-30T20:19:46,191 [random-source-pool-0] INFO  com.amazon.dataprepper.plugins.source.RandomStringSource - Writing to buffer
2021-09-30T20:19:46,694 [random-source-pool-0] INFO  com.amazon.dataprepper.plugins.source.RandomStringSource - Writing to buffer
2021-09-30T20:19:47,200 [random-source-pool-0] INFO  com.amazon.dataprepper.plugins.source.RandomStringSource - Writing to buffer
2021-09-30T20:19:49,181 [simple-test-pipeline-processor-worker-1-thread-1] INFO  com.amazon.dataprepper.pipeline.ProcessWorker -  simple-test-pipeline Worker: Processing 6 records from buffer
07dc0d37-da2c-447e-a8df-64792095fb72
5ac9b10a-1d21-4306-851a-6fb12f797010
99040c79-e97b-4f1d-a70b-409286f2a671
5319a842-c028-4c17-a613-3ef101bd2bdd
e51e700e-5cab-4f6d-879a-1c3235a77d18
b4ed2d7e-cf9c-4e9d-967c-b18e8af35c90
```
此页面的其余部分提供了从Docker映像中运行数据预备的示例。如果你
从源构建，请参阅[开发人员指南](https://github.com/opensearch-project/data-prepper/blob/main/docs/developer_guide.md) 了解更多信息。

但是，您配置了管道，您会以相同的方式运行数据预备。你运行码头机
图像并修改`pipelines.yaml` 和`data-prepper-config.yaml` 文件。

对于PEPEPPER 2.0或更高版本，请使用此命令：

```
docker run --name data-prepper -p 4900:4900 -v ${PWD}/pipelines.yaml:/usr/share/data-prepper/pipelines/pipelines.yaml -v ${PWD}/data-prepper-config.yaml:/usr/share/data-prepper/config/data-prepper-config.yaml opensearchproject/data-prepper:latest
```
{% include copy.html %}

对于更早的2.0的数据预先版本，请使用此命令：

```
docker run --name data-prepper -p 4900:4900 -v ${PWD}/pipelines.yaml:/usr/share/data-prepper/pipelines.yaml -v ${PWD}/data-prepper-config.yaml:/usr/share/data-prepper/data-prepper-config.yaml opensearchproject/data-prepper:1.x
```
{% include copy.html %}

数据预先运行后，它将处理数据，直到关闭。完成后，使用以下命令将其关闭：

```
POST /shutdown
```
{% include copy-curl.html %}}

### 其他配置

对于数据Prepper 2.0或更高版本，log4j 2配置文件是从中读取的`config/log4j2.properties` 在应用程序的主目录中。默认情况下，它使用`log4j2-rolling.properties` 在 *共享-config*目录。

对于数据预先1.5或更早，可选地添加`"-Dlog4j.configurationFile=config/log4j2.properties"` 如果要传递自定义log4j2属性文件，请访问命令。如果没有提供属性文件，则数据预先默认为log4j2.properties文件 *共享-config*目录。

## 下一步

跟踪分析是一个重要的数据预先用例。如果您尚未配置，请参阅[跟踪分析]({{site.url}}{{site.baseurl}}/data-prepper/common-use-cases/trace-analytics/)。

日志摄入也是重要的数据预先用例。要了解更多，请参阅[日志分析]({{site.url}}{{site.baseurl}}/data-prepper/common-use-cases/log-analytics/)。

要了解如何使用LogStash配置运行数据预先Prepper，请参见[从logstash迁移]({{site.url}}{{site.baseurl}}/data-prepper/migrating-from-logstash-data-prepper/)。

有关如何监视数据预先数据的信息，请参阅[监视]({{site.url}}{{site.baseurl}}/data-prepper/managing-data-prepper/monitoring/)。

## 更多例子

有关更多数据预先示例，请参见[例子](https://github.com/opensearch-project/data-prepper/tree/main/examples/) 在数据预备存储库中。

