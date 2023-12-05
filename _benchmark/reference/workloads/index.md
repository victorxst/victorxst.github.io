---
layout: default
title: 工作负载参考
nav_order: 60
parent: OpenSearch基准参考
has_children: true
redirect_from: /benchmark/workloads/index/
---

# OpenSearch基准工作负载参考

工作量是一个或多个基准测试方案的规范。工作负载通常包括以下内容：

- 一个或多个被摄入索引的数据流
- 作为基准的一部分调用的一组查询和操作

本节提供了您在自定义或使用工作负载时可以使用的选项和示例的列表。

有关构成工作负载的更多信息，请参阅[解剖]({{site.url}}{{site.baseurl}}/benchmark/user-guide/concepts#anatomy-of-a-workload)。


## 工作量示例

如果您想在创建自己的工作量之前尝试某些工作负载，请使用以下示例。

### 跑步

在下面的示例中，OpenSearch基准测试对未爆炸的批量操作进行1小时`movies` 指数：

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
    "operation": "bulk",
    "warmup-time-period": 120,
    "time-period": 3600,
    "clients": 8
  }
]
}
```

### 一项任务的工作量

以下工作负载运行具有一个任务的基准：a`match_all` 询问。因为没有`clients` 指示，仅使用一个客户。根据`schedule`，工作负载运行`match_all` 查询每秒10次操作，使用1个客户端，使用100个迭代进行预热，并使用接下来的100个迭代来测量基准：

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
{
  "schedule": [
    {
      "operation": {
        "operation-type": "search",
        "index": "_all",
        "body": {
          "query": {
            "match_all": {}
          }
        }
      },
      "warmup-iterations": 100,
      "iterations": 100,
      "target-throughput": 10
    }
  ]
}
}
```

## 下一步

- 有关配置OpenSearch基准测试的更多信息，请参阅[配置OpenSearch基准测试]({{site.url}}{{site.baseurl}}/benchmark/configuring-benchmark/)。
- 有关OpenSearch基准测试的预包装工作负载列表，请参阅[OpenSearch-基准-工作负载](https://github.com/opensearch-project/opensearch-benchmark-workloads) 存储库。

