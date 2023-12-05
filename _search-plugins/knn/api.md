---
layout: default
title: API
nav_order: 30
parent: k-NN
has_children: false
---

# k-NN插件API

k-NN插件添加了几个用于管理，监视和优化K的API-NN工作量。


## 统计
引入1.0
{：.label .label-紫色的 }

k-nn`stats` API提供有关K当前状态的信息-NN插件。该插件可以跟踪两个群集-级别和节点-水平统计。簇-级别统计信息对于整个群集具有一个值。节点-级别统计信息在群集中的每个节点都有一个值。您可以通过`nodeId` 和`statName`：
```
GET /_plugins/_knn/nodeId1,nodeId2/stats/statName1,statName2
```

统计|  描述
：--- | ：---
`circuit_breaker_triggered` | 指示是否触发断路器。该统计量仅与近似K有关-nn搜索。
`total_load_time` | 纳秒k的时间-NN已将本地库索引加载到缓存中。该统计量仅与近似K有关-nn搜索。
`eviction_count` | 由于内存约束或空闲时间，已从缓存驱逐的本机库索引数量。该统计量仅与近似K有关-nn搜索。<br />**笔记**：由于索引删除而发生的显式驱逐不计算。
`hit_count` | 命中量的数量。当用户查询已经加载到内存中的本机库索引时，就会发生缓存。该统计量仅与近似K有关-nn搜索。
`miss_count` | 缓存的数量错过。当用户查询尚未加载到内存中的本机库索引时，会发生缓存失误。该统计量仅与近似K有关-nn搜索。
`graph_memory_usage` | 本地内存本机库索引的量在千字中使用。
`graph_memory_usage_percentage` | 本机内存库索引的数量在节点上使用的最大缓存容量的百分比。
`graph_index_requests` | 添加的请求数`knn_vector` 文档的字段到本地库索引中。
`graph_index_errors` | 添加的请求数`knn_vector` 文档的字段到本机库索引中产生错误。
`graph_query_requests` | 已经进行的本地库索引查询数量。
`graph_query_errors` | 本机库索引查询的数量已产生错误。
`knn_query_requests` | K的数量-收到的NN查询请求。
`cache_capacity_reached` | 无论`knn.memory.circuit_breaker.limit` 已达到。该统计量仅与近似K有关-nn搜索。
`load_success_count` | k的次数k-NN成功将本机库索引加载到缓存中。该统计量仅与近似K有关-nn搜索。
`load_exception_count` | 试图将本机库索引加载到缓存中时发生异常的次数。该统计量仅与近似K有关-nn搜索。
`indices_in_cache` | 对于每个OpenSearch索引，`knn_vector` 场和近似K-nn打开，此统计量提供了OpenSearch索引所具有的本机库索引的数量和总计`graph_memory_usage` OpenSearch索引以千字节使用。
`script_compilations` | k的次数-NN脚本已编译。此值通常应为1或0，但是如果填充了包含编译脚本的缓存，则K-NN脚本可能会被重新编译。该统计量仅与K有关-NN得分脚本搜索。
`script_compilation_errors` | 脚本汇编过程中的错误数量。该统计量仅与K有关-NN得分脚本搜索。
`script_query_requests` | 脚本查询总数。该统计量仅与K有关-NN得分脚本搜索。
`script_query_errors` | 脚本查询期间的错误数。该统计量仅与K有关-NN得分脚本搜索。
`nmslib_initialized` | 布尔值表示是否已在节点上加载和初始化 * nmslib * jni库。
`faiss_initialized` | 布尔值表示是否已在节点上加载和初始化 * faiss * jni库。
`model_index_status` | 模型系统索引的状态。有效值是"red"，，，，"yellow"，，，，"green"。如果索引不存在，则将是无效的。
`indexing_from_model_degraded` | 布尔值表示是否降级了来自模型的索引。如果没有足够的JVM内存来缓存模型，这将发生。
`training_requests` | 向节点提出的培训请求数。
`training_errors` | 节点上发生的训练错误的数量。
`training_memory_usage` | 本机记忆训练的数量是在千字中的节点上使用的。
`training_memory_usage_percentage` | 天然内存训练的数量是在节点上使用的最大缓存容量的百分比。

**笔记**：某些统计数据包含名称中的 * Graph *。在这些情况下， *Graph *是 *本机库索引 *的代名词。术语 * Graph *是一个旧细节，来自插件仅支持由层次图组成的HNSW算法时。


### 用法

```json
GET /_plugins/_knn/stats?pretty
{
    "_nodes" : {
        "total" : 1,
        "successful" : 1,
        "failed" : 0
    },
    "cluster_name" : "my-cluster",
    "circuit_breaker_triggered" : false,
    "model_index_status" : "YELLOW",
    "nodes" : {
      "JdfxIkOS1-43UxqNz98nw" : {
        "graph_memory_usage_percentage" : 3.68,
        "graph_query_requests" : 1420920,
        "graph_memory_usage" : 2,
        "cache_capacity_reached" : false,
        "load_success_count" : 179,
        "training_memory_usage" : 0,
        "indices_in_cache" : {
            "myindex" : {
                "graph_memory_usage" : 2,
                "graph_memory_usage_percentage" : 3.68,
                "graph_count" : 2
            }
        },
        "script_query_errors" : 0,
        "hit_count" : 1420775,
        "knn_query_requests" : 147092,
        "total_load_time" : 2436679306,
        "miss_count" : 179,
        "training_memory_usage_percentage" : 0.0,
        "graph_index_requests" : 656,
        "faiss_initialized" : true,
        "load_exception_count" : 0,
        "training_errors" : 0,
        "eviction_count" : 0,
        "nmslib_initialized" : false,
        "script_compilations" : 0,
        "script_query_requests" : 0,
        "graph_query_errors" : 0,
        "indexing_from_model_degraded" : false,
        "graph_index_errors" : 0,
        "training_requests" : 17,
        "script_compilation_errors" : 0
    }
  }
}
```

```json
GET /_plugins/_knn/HYMrXXsBSamUkcAjhjeN0w/stats/circuit_breaker_triggered,graph_memory_usage?pretty
{
    "_nodes" : {
        "total" : 1,
        "successful" : 1,
        "failed" : 0
    },
    "cluster_name" : "my-cluster",
    "circuit_breaker_triggered" : false,
    "nodes" : {
        "HYMrXXsBSamUkcAjhjeN0w" : {
            "graph_memory_usage" : 1
        }
    }
}
```


## 热身操作
引入1.0
{：.label .label-紫色的 }

用于执行近似K的本地库索引-最近的邻居（K-nn）搜索作为特殊文件和其他Apache Lucene细分文件存储。为了您使用K对这些索引进行搜索-NN插件，插件需要将这些文件加载到本机内存中。

如果插件尚未将文件加载到本机内存中，则在收到搜索请求时将其加载。加载时间可能会在初始查询期间导致高潜伏期。为了避免这种情况，用户经常在热身期间进行随机查询。在此热身期之后，将文件加载到本机内存中，并且可以开始其生产工作负载。这种加载过程是间接的，需要额外的努力。

作为替代方案，您可以通过运行K来避免此延迟问题-NN插件热身API在您有兴趣搜索的任何索引上操作。此操作将所有在请求中指定的索引的所有碎片（原始和副本）加载到本机内存中。

过程完成后，您可以开始搜索索引，而没有初始延迟惩罚。热身API操作是有掌握的，因此，如果已经将部分的本机库文件加载到内存中，则此操作不会影响。它仅加载当前不在内存中的文件。


### 用法

此请求对三个索引进行热身：

```json
GET /_plugins/_knn/warmup/index1,index2,index3?pretty
{
  "_shards" : {
    "total" : 6,
    "successful" : 6,
    "failed" : 0
  }
}
```

`total` 表示K-NN插件试图热身。响应还包括插件成功的碎片数量，无法热身。

在热身操作完成或请求时间之后，该呼叫不会返回结果。如果请求时间耗尽，则该操作仍在集群上继续。要监视热身操作，请使用OpenSearch`_tasks` API：

```json
GET /_tasks
```

操作完成后，请使用[k-nn`_stats` API操作](#stats) 看看K-NN插件加载到图中。


### 最佳实践

要使热身操作正常运行，请遵循以下最佳实践：

*不要在要热身的索引上进行合并操作。在合并期间，K-NN插件会创建新的细分市场，有时会删除旧段。例如，您可能会遇到一种情况，在这种情况下，热身API操作将本机库将A和B加载到本机内存中，但是C cement c是从段A和B合并的。A和B的本地库索引将不再在内存中，而本机库索引C也不会在内存中。在这种情况下，仍然存在加载本机库索引C的初始罚款。

*确认您要热身的所有本地库索引都可以适合本地记忆。有关本地内存限制的更多信息，请参阅[knn.memory.circuit_breaker.limit统计]({{site.url}}{{site.baseurl}}/search-plugins/knn/settings#cluster-settings)。高图内存储器的使用导致缓存thrashing，这可能导致操作不断失败并尝试再次运行。

*不要为要加载到缓存中的任何文档索引。编写新信息以防止热身API操作加载本机库索引，直到可以搜索为止。这意味着您必须在索引完成后再次进行热身操作。

## 获取模型
引入了1.2
{：.label .label-紫色的 }

用于检索集群中存在的模型的信息。一些本地库索引配置需要
在索引和查询之前，培训步骤可以开始。培训的输出是一个模型，然后可以用来
在索引期间初始化本机库索引文件。该模型在K中序列化-NN模型系统索引。

```
GET /_plugins/_knn/models/{model_id}
```

响应字段|  描述
：--- | ：---
`model_id` | 获取模型的ID。
`model_blob` | 基本64编码序列化模型的字符串。
`state` | 模型的当前状态。任何一个"created"，，，，"failed"，，，，"training"。
`timestamp` | 创建模型的时间。
`description` | 用户提供了模型的描述。
`error` | 错误消息说明为什么模型处于失败状态。
`space_type` | 空间类型此型号经过训练。
`dimension` | 维度此模型是为了。
`engine` | 用于创建模型的本地库。任何一个"faiss" 或者"nmslib"。

### 用法

```json
GET /_plugins/_knn/models/test-model?pretty
{
  "model_id" : "test-model",
  "model_blob" : "SXdGbIAAAAAAAAAAAA...",
  "state" : "created",
  "timestamp" : "2021-11-15T18:45:07.505369036Z",
  "description" : "Default",
  "error" : "",
  "space_type" : "l2",
  "dimension" : 128,
  "engine" : "faiss" 
}
```

```json
GET /_plugins/_knn/models/test-model?pretty&filter_path=model_id,state
{
  "model_id" : "test-model",
  "state" : "created"
}
```

## 搜索模型
引入了1.2
{：.label .label-紫色的 }

使用OpenSearch查询在索引中搜索模型。

### 用法
```json
GET/POST /_plugins/_knn/models/_search?pretty&_source_excludes=model_blob
{
    "query": {
         ...
     }
}

{
    "took" : 0,
    "timed_out" : false,
    "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
    },
    "hits" : {
      "total" : {
          "value" : 1,
          "relation" : "eq"
      },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : ".opensearch-knn-models",
        "_id" : "test-model",
        "_score" : 1.0,
        "_source" : {
          "engine" : "faiss",
          "space_type" : "l2",
          "description" : "Default",
          "model_id" : "test-model",
          "state" : "created",
          "error" : "",
          "dimension" : 128,
          "timestamp" : "2021-11-15T18:45:07.505369036Z"
        }
      }
    ]
  }
}
```

## 删除模型
引入了1.2
{：.label .label-紫色的 }

用于删除集群中的特定模型。

### 用法

```json
DELETE /_plugins/_knn/models/{model_id}
{
  "model_id": {model_id},
  "acknowledged": true
}
```

## 火车模型
引入了1.2
{：.label .label-紫色的 }

创建并训练可用于初始化k的模型-NN本地库在索引期间索引。这个API会
从一个`knn_vector` 在培训指数中进行字段，然后创建和训练模型，然后序列化
到模型系统索引。训练数据必须与传递到请求正文的维度相匹配。这个请求
训练开始时会返回。要监视模型的状态，请使用[获取模型API](#get-model)。

查询参数|  描述
：--- | ：---
`model_id` | （可选）获取模型的ID。如果未指定，将生成随机ID。
`node_id` | （可选）首选节点用于执行培训。如果设置，则该节点将被视为有能力进行培训。

请求参数|  描述
：--- | ：---
`training_index` | 从哪里培训数据的索引。
`training_field` | `knn_vector` 字段来自`training_index` 从中获取培训数据。该领域的维度必须匹配`dimension` 传递到此请求。
`dimension` | 维度此模型是为了。
`max_training_vector_count` | （可选）训练指数的最大向量数量用于训练。默认为索引中的所有向量。
`search_size` | （可选的）培训数据从带有滚动查询的培训指数中获取。定义每个滚动查询返回的结果数。默认为10,000。
`description` | （可选）用户提供了模型的描述。
`method` | ANN方法的配置用于搜索。有关可能方法的更多信息，请参阅[方法文档]({{site.url}}{{site.baseurl}}/search-plugins/knn/knn-index#method-definitions)。方法必须需要培训才能有效。
   

### 用法

```json
POST /_plugins/_knn/models/{model_id}/_train?preference={node_id}
{
    "training_index": "train-index-name",
    "training_field": "train-field-name",
    "dimension": 16,
    "max_training_vector_count": 1200,
    "search_size": 100,
    "description": "My model",
    "method": {
        "name":"ivf",
        "engine":"faiss",
        "space_type": "l2",
        "parameters":{
            "nlist":128,
            "encoder":{
                "name":"pq",
                "parameters":{
                    "code_size":8
                }
            }
        }
    }
}

{
    "model_id": "model_x"
}
```

```json
POST /_plugins/_knn/models/_train?preference={node_id}
{
    "training_index": "train-index-name",
    "training_field": "train-field-name",
    "dimension": 16,
    "max_training_vector_count": 1200,
    "search_size": 100,
    "description": "My model",
    "method": {
        "name":"ivf",
        "engine":"faiss",
        "space_type": "l2",
        "parameters":{
            "nlist":128,
            "encoder":{
                "name":"pq",
                "parameters":{
                    "code_size":8
                }
            }
        }
    }
}

{
    "model_id": "dcdwscddscsad"
}
```

