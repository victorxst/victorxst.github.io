---
layout: default
title: 数据准备者 
nav_order: 1
has_children: false
has_toc: false
nav_exclude: true
redirect_from: 
  - /clients/data-prepper/index/
  - /data-prepper/
  - /monitoring-plugins/trace/data-prepper/
---

# 数据准备者

数据PEPPEPER是服务器-侧数据收集器能够过滤，丰富，转换，归一化和汇总数据，以进行下游分析和可视化。

数据PEPPPER允许用户构建自定义管道以改善应用程序的操作视图。数据预先的两种常见用途是跟踪和日志分析。[跟踪分析]({{site.url}}{{site.baseurl}}/observability-plugin/trace/index/) 可以帮助您可视化事件的流程并确定性能问题，并且[日志分析]({{site.url}}{{site.baseurl}}/observability-plugin/log-analytics/) 可以改善搜索，分析并提供有关应用程序的见解。

## 概念

数据预珀包括一个或多个**管道** 该收集和过滤数据基于管道中设置的组件。每个组件都是可插入的，使您可以使用自己的每个组件的自定义实现。这些组件包括以下内容：

- 一[来源](#source)
- 一个或多个[水槽](#sink)
- （可选）一个[缓冲](#buffer)
- （可选）一个或多个[处理器](#processor)

数据预先的一个实例可以具有一个或多个管道。

每个管道定义包含两个必需的组件：**来源** 和**下沉**。如果数据Prepper管道中缺少缓冲区和处理器，则Data Prepper使用默认缓冲区和否-OP处理器。

### 来源

源是定义数据预先管道将消耗事件的机制的输入组件。管道只能有一个来源。源可以通过通过HTTP或HTTP接收事件或从外部端点（如Otel Collector for Trace and Metrics）以及Amazon Simple Storage Service（Amazon S3）等外部端点（Amazon collector）读取事件。源具有基于事件格式的配置选项（例如字符串，JSON，Amazon CloudWatch日志或打开的遥测跟踪）。源组件会消耗事件并将其写入缓冲区组件。

### 缓冲

缓冲区组件充当源和水槽之间的层。缓冲区可以在-基于内存或磁盘。默认缓冲区使用-记忆队列拨打`bounded_blocking` 这是由事件数量界定的。如果管道配置中未明确提及缓冲区组件，则数据prepper使用默认`bounded_blocking`。

### 下沉

接收器是定义数据Prepper管道发布事件的目的地的输出组件。接收器目的地可以是一项服务，例如OpenSearch或Amazon S3或其他数据Prepper管道。当使用其他数据Prepper管道作为水槽时，您可以根据数据的需求将多个管道链接在一起。水槽包含基于目标类型的自己的配置选项。

### 处理器

处理器是数据Prepper管道中的单位，可以在将记录发布到接收器组件之前，可以使用所需格式过滤，转换和丰富事件。处理器未在管道配置中定义；事件以源组件中定义的格式发布。您可以在管道中拥有多个处理器。使用多个处理器时，处理器按顺序运行，在管道规范中定义。

## 样本管道配置

要了解所有管道组件在数据预先配置中的功能，请参见以下示例。每个管道配置都使用`yaml` 文件格式。

### 最小成分

此管道配置从文件源读取，并在同一路径中写入另一个文件。它使用缓冲区和处理器的默认选项。

```yml
sample-pipeline:
  source:
    file:
        path: <path/to/input-file>
  sink:
    - file:
        path: <path/to/output-file>
```

### 所有组件

以下管道使用源读取字符串事件的源`input-file`。然后，来源将数据推到缓冲区，并由`1024`。管道配置为具有`4` 工人，他们每个人最多阅读`256` 每个缓冲区的事件`100 milliseconds`。每个工人运行`string_converter` 处理器并将处理器的输出写入`output-file`。

```yml
sample-pipeline:
  workers: 4 #Number of workers
  delay: 100 # in milliseconds, how often the workers should run
  source:
    file:
        path: <path/to/input-file>
  buffer:
    bounded_blocking:
      buffer_size: 1024 # max number of events the buffer will accept
      batch_size: 256 # max number of events the buffer will drain for each read
  processor:
    - string_converter:
       upper_case: true
  sink:
    - file:
       path: <path/to/output-file>
```

## 下一步

要开始使用Data Prepper构建自己的自定义管道，请参阅[入门]({{site.url}}{{site.baseurl}}/clients/data-prepper/get-started/)。

<！---删除此评论。--->


