---
layout: default
title: 概念
nav_order: 3
parent: 用户指南
---

# 概念

在使用OpenSearch基准测试之前，请熟悉以下概念。

## 核心概念和定义

- **工作量**：使用特定文档语料库对您的群集执行基准测试的一个或多个基准测试方案的描述。文档语料库包含工作流程运行时调用的任何索引，数据文件和操作。您可以使用`opensearch-benchmark list workloads` 或查看任何包含的工作负载[OpenSearch基准工作负载存储库](https://github.com/opensearch-project/opensearch-benchmark-workloads/)。有关工作量元素的更多信息，请参见[解剖](#anatomy-of-a-workload)。有关构建自定义工作量的信息，请参阅[创建自定义工作负载]({{site.url}}{{site.baseurl}}/benchmark/creating-custom-workloads/)。

- **管道**：运行工作负载之前和之后发生的一系列步骤，以确定基准结果。OpenSearch基准测试支持三个管道：
  - `from-sources`：构建和规定OpenSearch，运行基准测试，然后发布结果。
  - `from-distribution`：下载opensearch发行版，规定，运行基准，然后发布结果。
  - `benchmark-only`：默认管道。假设已经运行的OpenSearch实例，在该实例上运行基准，然后发布结果。

- **测试**：OpenSearch基准二进制的单一调用。

工作量是一个或多个基准测试方案的规范。工作负载通常包括以下内容：

- 一个或多个被摄入索引的数据流。
- 作为基准的一部分调用的一组查询和操作。

## 解剖

以下示例工作负载显示创建一个所需的所有基本要素`workload.json` 文件。您可以在自己的基准配置中运行此工作量，以了解所有元素如何一起工作：

```json
{
  "description": "Tutorial benchmark for OpenSearch Benchmark",
  "indices": [
    {
      "name": "movies",
      "body": "index.json"
    }
  ],
  "corpora": [
    {
      "name": "movies",
      "documents": [
        {
          "source-file": "movies-documents.json",
          "document-count": 11658903, # Fetch document count from command line
          "uncompressed-bytes": 1544799789 # Fetch uncompressed bytes from command line
        }
      ]
    }
  ],
  "schedule": [
    {
      "operation": {
        "operation-type": "create-index"
      }
    },
    {
      "operation": {
        "operation-type": "cluster-health",
        "request-params": {
          "wait_for_status": "green"
        },
        "retry-until-success": true
      }
    },
    {
      "operation": {
        "operation-type": "bulk",
        "bulk-size": 5000
      },
      "warmup-time-period": 120,
      "clients": 8
    },
    {
      "operation": {
        "name": "query-match-all",
        "operation-type": "search",
        "body": {
          "query": {
            "match_all": {}
          }
        }
      },
      "iterations": 1000,
      "target-throughput": 100
    }
  ]
}
```

工作量通常包括以下元素：

- [指数]({{site.url}}{{site.baseurl}}/benchmark/workloads/indices/)：定义用于工作负载的相关索引和索引模板。
- [语料库]({{site.url}}{{site.baseurl}}/benchmark/workloads/corpora/)：定义用于工作量的所有文档Corpora。
- `schedule`：定义操作和操作内联行动的顺序。或者，您可以使用`operations` 进行小组操作和`test_procedures` 参数指定操作顺序。
- `operations`：**选修的**。描述哪些操作可用于工作负载以及如何对其进行参数化。

### 指数

要创建索引，请指定其`name`。要在您的索引中添加定义，请使用`body` 选项并将其指向包含索引定义的JSON文件。有关更多信息，请参阅[指数]({{site.url}}{{site.baseurl}}/benchmark/workloads/indices/)。

### 语料库

这`corpora` 元素需要包含文档语料库的索引的名称，例如`movies`，以及定义文档语料库的参数列表。此列表包括以下参数：

-  `source-file`：包含工作负载的相应文档的文件名。在本地使用OpenSearch基准测试时，文档包含在JSON文件中。提供一个`base_url`，使用压缩文件格式：`.zip`，`.bz2`，`.gz`，`.tar`，`.tar.gz`，`.tgz`， 或者`.tar.bz2`。压缩文件必须具有一个包含名称的JSON文件。
-  `document-count`：在`source-file`，确定哪个客户索引与文档语料库的哪些部分相关。每个n客户端都会收到文档语料库的nth。当使用包含父母文档的源时-儿童关系，指定家长文档的数量。
- `uncompressed-bytes`：解压缩后源文件的大小，字节，指示解压缩源文件所需的磁盘空间。
- `compressed-bytes`：解压缩之前的源文件的大小，字节。这可以帮助您评估集群摄入文档所需的时间。

### 运营

这`operations` 元素列出了工作负载执行的OpenSearch API操作。例如，您可以将操作设置为`create-index`，OpenSearch基准标准可以编写文档的测试集群中的索引。通常在内部列出操作`schedule`。

### 日程

这`schedule` 元素包含由工作负载运行的操作和操作列表。操作根据它们在`schedule`。以下示例说明了`schedule` 具有多个操作，每个操作都由其定义`operation-type`：

```json
  "schedule": [
    {
      "operation": {
        "operation-type": "create-index"
      }
    },
    {
      "operation": {
        "operation-type": "cluster-health",
        "request-params": {
          "wait_for_status": "green"
        },
        "retry-until-success": true
      }
    },
    {
      "operation": {
        "operation-type": "bulk",
        "bulk-size": 5000
      },
      "warmup-time-period": 120,
      "clients": 8
    },
    {
      "operation": {
        "name": "query-match-all",
        "operation-type": "search",
        "body": {
          "query": {
            "match_all": {}
          }
        }
      },
      "iterations": 1000,
      "target-throughput": 100
    }
  ]
}
```

根据此时间表，这些操作将按以下顺序进行：

1. 这`create-index` 操作创建索引。索引保持空，直到`bulk` 操作添加了具有基准数据的文档。
2. 这`cluster-health` 操作在运行工作量之前评估集群的健康状况。在此示例中，工作量等待直到集群健康状况为`green`。
   - 这`bulk` 操作运行`bulk` API到索引`5000` 同时文件。
   - 在基准测试之前，工作量等待直到指定`warmup-time-period` 通过。在此示例中，热身时期是`120` 秒。
5. 这`clients` 字段定义了将同时在时间表中运行其余操作的客户端数量。
6. 这`search` 运行a`match_all` 查询所有文档已被所有文档索引之后`bulk` 使用指定的8个客户端的API。
   - 这`iterations` 字段指示每个客户端运行的次数`search` 手术。基准生成的报告会根据此数字自动调整百分位数。要产生精确的百分位数，基准需要至少进行1,000次迭代。
   - 最后，`target-throughput` 字段定义每个客户端执行的每秒请求数，设置后可以帮助减少基准的延迟。例如，`target-throughput` 在100个请求除以8个客户端的要求中，每个客户端将每秒发出12个请求。

