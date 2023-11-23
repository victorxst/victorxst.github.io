---
layout: default
title: 关于 OpenSearch
nav_order: 2
parent: OpenSearch 文档
redirect_from:
  - /docs/opensearch/
  - /opensearch/
  - /opensearch/index/
---

{%- comment -%}The `/docs/opensearch/` redirect is specifically to support the UI links in OpenSearch Dashboards 1.0.0.{%- endcomment -%}

# OpenSearch 简介

OpenSearch 是一个基于[Apache Lucene](https://lucene.apache.org/).将数据添加到 OpenSearch 后，你可以使用你可能期望的所有功能对其执行全文搜索：按字段搜索、搜索多个索引、提升字段、按分数对结果进行排名、按字段对结果进行排序以及聚合结果。

不出所料，人们经常使用 OpenSearch 等搜索引擎作为搜索应用程序---思考[Wikipedia](https://en.wikipedia.org/wiki/Wikipedia:FAQ/Technical#What_software_is_used_to_run_Wikipedia?)或在线商店的后端。它提供了出色的性能，并且可以随着应用程序需求的增长或缩小而扩展和缩减。

日志分析是一个同样流行但不太明显的用例，你可以在其中从应用程序获取日志，将其馈送到 OpenSearch 中，然后使用丰富的搜索和可视化功能来识别问题。例如，出现故障的 Web 服务器可能会在 0.5% 的时间内抛出 500 错误，除非你拥有服务器在过去 4 小时内抛出的所有 HTTP 状态代码的实时图表，否则很难注意到这一点。你可以使用[OpenSearch 控制面板]({{site.url}}{{site.baseurl}}/dashboards/index/)从 OpenSearch 中的数据构建这些类型的可视化。


## 集群和节点

其分布式设计意味着你可以与 OpenSearch *集群*进行交互。每个集群是一个或多个*节点*服务器的集合，用于存储数据和处理搜索请求。

你可以在笔记本电脑上本地运行 OpenSearch---其系统要求最低---但你也可以将单个集群扩展到数据中心中的数百台功能强大的计算机。

在单节点集群（如笔记本电脑）中，一台计算机必须执行所有操作：管理集群的状态、索引和搜索数据，以及在索引数据之前对数据执行任何预处理。但是，随着集群的增长，你可以细分职责。具有快速磁盘和大量 RAM 的节点可能非常适合索引和搜索数据，而具有大量 CPU 能力和小磁盘的节点可以管理群集状态。有关设置节点类型的详细信息，请参阅[集群形成]({{site.url}}{{site.baseurl}}/opensearch/cluster/)。


## 索引和文档

OpenSearch 将数据组织成*指标*.每个索引都是 JSON *文件*的集合。如果你有一组原始百科全书文章或日志行要添加到 OpenSearch，则必须先将它们[JSON](https://www.json.org/)转换为 .电影的简单 JSON 文档可能如下所示：

```json
{
  "title": "The Wind Rises",
  "release_date": "2013-07-20"
}
```

当你将文档添加到索引时，OpenSearch 会添加一些元数据，例如唯一文档*编号*：

```json
{
  "_index": "<index-name>",
  "_type": "_doc",
  "_id": "<document-id>",
  "_version": 1,
  "_source": {
    "title": "The Wind Rises",
    "release_date": "2013-07-20"
  }
}
```

索引还包含映射和设置：

- A *映射*是索引中文档的集合*领域*。在本例中，这些字段为 `title` 和 `release_date`。
- 设置包括索引名称、创建日期和分片数等数据。

## 主分片和副本分片

OpenSearch 将索引拆分为*碎片*，以便在集群中的节点之间均匀分布。例如，400 GB 的索引对于集群中的任何单个节点来说可能太大，但拆分为 10 个分片，每个分片 40 GB，OpenSearch 可以将分片分布在 10 个节点上，并单独处理每个分片。

默认情况下，OpenSearch 会*主要*为每个分片创建一个*复制品*分片。例如，如果你将索引拆分为 10 个分片，则 OpenSearch 还会创建 10 个副本分片。这些副本分片在节点发生故障时充当备份---OpenSearch 将副本分片分发到与其对应的主分片不同的节点---但它们也提高了集群处理搜索请求的速度和速率。你可以为每个索引指定多个副本，以用于搜索密集型工作负载。

尽管是 OpenSearch 索引的一部分，但每个分片实际上都是一个完整的 Lucene 索引---我们知道，令人困惑。不过，这个细节很重要，因为每个 Lucene 实例都是一个消耗 CPU 和内存的正在运行的进程。分片不一定越多越好。例如，将 400 GB 索引拆分为 1,000 个分片会给集群带来不必要的压力。一个好的经验法则是将分片大小保持在 10--50 GB 之间。


## REST API 接口

你可以使用 REST API 与 OpenSearch 集群进行交互，这提供了很大的灵活性。你可以使用客户端或[curl](https://curl.se/)任何可以发送 HTTP 请求的编程语言。要将 JSON 文档添加到 OpenSearch 索引（即索引文档），请发送 HTTP 请求：

```json
PUT https://<host>:<port>/<index-name>/_doc/<document-id>
{
  "title": "The Wind Rises",
  "release_date": "2013-07-20"
}
```

要运行文档搜索，请执行以下操作：

```
GET https://<host>:<port>/<index-name>/_search?q=wind
```

要删除文档：

```
DELETE https://<host>:<port>/<index-name>/_doc/<document-id>
```

你可以使用 REST API 更改大多数 OpenSearch 设置、修改索引、检查集群的运行状况、获取统计信息---几乎所有内容。
