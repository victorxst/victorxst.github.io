---
layout: default
title: 监视
parent: 管理数据预先
nav_order: 25
---
 
# 用指标监视数据预先

您可以使用指标来监视数据预先[千分尺](https://micrometer.io/)。指标有两种类型：JVM/系统指标和插件指标。[普罗米修斯](https://prometheus.io/) 被用作默认指标后端。

## JVM和系统指标

JVM和系统指标是用于监视数据预先实例的运行时指标。它们包括用于类负载的指标，内存，垃圾收集，线程等。有关更多信息，请参阅[JVM和系统指标](https://micrometer.io/?/docs/ref/jvm)。

### 命名

JVM和系统指标遵循预定义的名称[千分尺](https://micrometer.io/?/docs/concepts#_naming_meters)。例如，用于内存使用的微米指标名称是`jvm.memory.used`。千分尺更改名称以匹配指标系统。遵循同一示例，`jvm.memory.used` 据报道是Prometheus`jvm_memory_used`，并报告给Amazon CloudWatch作为`jvm.memory.used.value`。

### 服务

默认情况下，指标是从**/指标/系统** Prometheus scrape格式的Data Prepper服务器上的端点。您可以将Prometheus配置为从数据prepper URL中刮下来。然后，Prometheus对指标进行调查，并将其存储在其数据库中。为了可视化数据，您可以设置任何接受Prometheus指标的前端，例如[格拉法纳](https://prometheus.io/docs/visualization/grafana/)。您可以更新配置以将指标提供给其他注册表，例如Amazon CloudWatch，该注册表不需要或托管端点，而是将指标直接发布到CloudWatch。

## 插件指标

插件报告自己的指标。Data Prepper使用命名约定来帮助指标的一致性。插件指标不使用尺寸。


1. AbstractBuffer
    - 柜台
        - `recordsWritten`：写入缓冲区的记录数量
        - `recordsRead`：从缓冲区读取的记录数量
        - `recordsProcessed`：从缓冲区读取的记录数量并标记为处理
        - `writeTimeouts`：在缓冲区中写入超时的计数
    - 仪菲尔
        - `recordsInBuffer`：缓冲区中的记录数
        - `recordsInFlight`：从缓冲区读取的记录数量并通过数据处理-在下游Prepper（例如，处理器，接收器）
    - 计时器
        - `readTimeElapsed`：从缓冲区阅读时经过的时间
        - `checkpointTimeElapsed`：检查点时经过的时间
2. 抽象处理器
    - 柜台
        - `recordsIn`：进入处理器的记录数量
        - `recordsOut`：从处理器中抽出的记录数量
    - 计时器
        - `timeElapsed`：在启动处理器期间经过的时间
3. AbstractSink
    - 柜台
        - `recordsIn`：进入水槽的记录数量
    - 计时器
        - `timeElapsed`：执行水槽期间经过的时间

### 命名

指标遵循命名惯例**pipeline_name_plugin_name_metric_name**。例如，**记录** 公制**OpenSearch-下沉** 插件中的插件名为**输出-管道** 有一个合格的名称**输出-pipeline_opensearch_sink_recordsin**。

### 服务

默认情况下，指标是从**/指标/系统** Prometheus scrape格式的Data Prepper服务器上的端点。您可以将Prometheus配置为从数据prepper URL中刮下来。数据Prepper服务器端口的默认值为`4900` 您可以修改，并且此端口可用于接受普罗米修斯指标的任何前端，例如[格拉法纳](https://prometheus.io/docs/visualization/grafana/)。您可以将配置更新以将指标提供给其他注册表，例如CloudWatch，不需要或托管端点，而是将指标直接发布到CloudWatch

