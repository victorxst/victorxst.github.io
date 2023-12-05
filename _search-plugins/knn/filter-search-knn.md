---
layout: default
title: k-NN搜索过滤器
nav_order: 20
parent: k-NN
has_children: false
has_math: true
---

# k-NN搜索过滤器

精炼k-nn结果，您可以过滤k-使用以下方法之一进行nn搜索：

- [高效k-NN过滤](#efficient-k-nn-filtering)：这种方法应用过滤_During_ k-nn搜索，而不是k之前或之后-nn搜索，这确保了`k` 返回结果（如果至少有`k` 总共结果）。这种方法得到以下引擎的支持：
  - Lucene Engine具有层次可通航的小世界（HNSW）算法（K-NN插件版本2.4及以后）
  - 带有HNSW算法的Faiss引擎（K-NN插件版本2.9及以后）或IVF算法（k-NN插件版本2.10及以后）

-  [邮政-过滤](#post-filtering)：因为它是在k之后执行的-nn搜索，这种方法的返回可能少于`k` 限制性过滤器的结果。您可以为此方法使用以下两个过滤策略：
    - [布尔邮报-筛选](#boolean-filter-with-ann-search)：这种方法运行[大约最近的邻居（ANN）]({{site.url}}{{site.baseurl}}/search-plugins/knn/approximate-knn/) 搜索然后将过滤器应用于结果。两个查询零件是独立执行的，然后根据查询操作员组合结果（`should`，，，，`must`在查询中提供的）。
    - [这`post_filter` 范围](#post-filter-parameter)：这种方法运行[安]({{site.url}}{{site.baseurl}}/search-plugins/knn/approximate-knn/) 在完整数据集上搜索，然后将过滤器应用于K-NN结果。

- [评分脚本过滤器](#scoring-script-filter)：这种方法涉及前-过滤文档集，然后运行精确的k-NN搜索过滤后的子集。它可能具有很高的延迟，并且在过滤后的子集很大时不会扩展。

下表总结了前面的过滤用例。

筛选| 应用过滤器时| 搜索类型| 支持的引擎和方法| 在哪里放置`filter` 条款
：--- | ：--- | ：--- | ：---
高效k-NN过滤| 在搜索期间（pre的混合体- 和张贴-过滤）| 近似| - `lucene` （（`hnsw`）<br>- `faiss` （（`hnsw`，，，，`ivf`）| 在K内-nn查询子句。
布尔过滤器| 搜索后（发布-过滤）| 近似| - `lucene`<br>- `nmslib`<br>- `faiss` | 在K外面-nn查询子句。必须是叶子子句。
这`post_filter` 范围| 搜索后（发布-过滤）| 近似| - `lucene`<br>- `nmslib`<br>- `faiss` | 在K外面-nn查询子句。
评分脚本过滤器| 搜索之前（pre-过滤）| 精确的| N/A。| 在脚本评分查询子句中。

## 过滤的搜索优化

根据您的数据集和用例，您可能对最大化召回或最大程度地减少延迟更感兴趣。下表提供了各种k的指导-NN搜索配置和用于优化更高召回或降低延迟的过滤方法。表的前三列提供了几个示例k-NN搜索配置。搜索配置包括：

- 索引中的文档数量，其中一个OpenSearch文档对应于一个K-nn矢量。
- 过滤后结果保留的文档百分比。该值取决于您在查询中提供的过滤器的限制性。表中最限制的过滤器返回索引中的2.5％的文档，而限制性最小的过滤器返回80％的文档。
- 所需的返回结果数（K）。

估计索引中的文档数量，过滤器的限制性以及所需的最近邻居数量，请使用下表选择一种优化召回或延迟的过滤方法。

| 索引中的文档数量| 过滤器返回的文档百分比| k| 用于更高召回的过滤方法| 用于较低延迟的过滤方法|
| ：-- | ：-- | ：-- | ：-- | ：-- |
| 10m| 2.5| 100| 高效k-NN过滤/评分脚本| 评分脚本|
| 10m| 38| 100| 高效k-NN过滤| 高效k-NN过滤|
| 10m| 80| 100| 高效k-NN过滤| 高效k-NN过滤|
| 1m| 2.5| 100| 高效k-NN过滤/评分脚本| 评分脚本|
| 1m| 38| 100| 高效k-NN过滤| 高效k-NN过滤|
| 1m| 80| 100| 高效k-NN过滤| 高效k-NN过滤|

## 高效k-NN过滤

您可以执行高效的K-NN过滤`lucene` 或者`faiss` 引擎。

### 卢肯·K-NN过滤器实现

k-NN插件2.2引入了运行K的支持-NN使用HNSW Graphs使用Lucene引擎进行搜索。从基于Lucene版本9.4的2.4版开始，您可以使用Lucene过滤器作为K-nn搜索。

当您为K指定Lucene过滤器时-nn搜索，Lucene算法决定是否执行精确的K-nn搜索-通过修改后的过滤或近似搜索-过滤。该算法使用以下变量：

- N：索引中的文档数量。
- P：应用过滤器后的文档子集中的文档数（p <= n）。
- K：响应中返回的最大向量数。

以下流程图概述了Lucene算法。

![Lucene算法用于过滤]({{site.url}}{{site.baseurl}}/images/lucene-algorithm.png)

有关Lucene过滤实施和基础的更多信息`KnnVectorQuery`，请参阅[Apache Lucene文档](https://lucene.apache.org/core/9_2_0/core/org/apache/lucene/search/KnnVectorQuery.html)。

### 使用Lucene K-NN过滤器

考虑一个包含12个包含酒店信息的文档的数据集。下图显示了按位置按XY坐标平面上的所有酒店。此外，用橙色点描绘了8至10之间的酒店的点，并用绿色圈子描绘了提供停车位的酒店。搜索点是红色的：

![带有过滤标准的文档图]({{site.url}}{{site.baseurl}}/images/knn-doc-set-for-filtering.png)

在此示例中，您将创建一个索引，并搜索最接近搜索位置的高评级和停车位的三个酒店。

**步骤1：创建一个新索引**

在运行k之前-NN搜索使用过滤器，您需要使用`knn_vector` 场地。对于此字段，您需要指定`lucene` 作为引擎和`hnsw` 作为`method` 在映射中。

以下请求创建了一个名为的新索引`hotels-index` 与`knn-filter` 字段称为`location`：

```json
PUT /hotels-index
{
  "settings": {
    "index": {
      "knn": true,
      "knn.algo_param.ef_search": 100,
      "number_of_shards": 1,
      "number_of_replicas": 0
    }
  },
  "mappings": {
    "properties": {
      "location": {
        "type": "knn_vector",
        "dimension": 2,
        "method": {
          "name": "hnsw",
          "space_type": "l2",
          "engine": "lucene",
          "parameters": {
            "ef_construction": 100,
            "m": 16
          }
        }
      }
    }
  }
}
```
{％包含副本-curl.html％}

**步骤2：将数据添加到您的索引**

接下来，将数据添加到您的索引中。

以下请求添加了12个文档，其中包含酒店位置，评级和停车信息：

```json
POST /_bulk
{ "index": { "_index": "hotels-index", "_id": "1" } }
{ "location": [5.2, 4.4], "parking" : "true", "rating" : 5 }
{ "index": { "_index": "hotels-index", "_id": "2" } }
{ "location": [5.2, 3.9], "parking" : "false", "rating" : 4 }
{ "index": { "_index": "hotels-index", "_id": "3" } }
{ "location": [4.9, 3.4], "parking" : "true", "rating" : 9 }
{ "index": { "_index": "hotels-index", "_id": "4" } }
{ "location": [4.2, 4.6], "parking" : "false", "rating" : 6}
{ "index": { "_index": "hotels-index", "_id": "5" } }
{ "location": [3.3, 4.5], "parking" : "true", "rating" : 8 }
{ "index": { "_index": "hotels-index", "_id": "6" } }
{ "location": [6.4, 3.4], "parking" : "true", "rating" : 9 }
{ "index": { "_index": "hotels-index", "_id": "7" } }
{ "location": [4.2, 6.2], "parking" : "true", "rating" : 5 }
{ "index": { "_index": "hotels-index", "_id": "8" } }
{ "location": [2.4, 4.0], "parking" : "true", "rating" : 8 }
{ "index": { "_index": "hotels-index", "_id": "9" } }
{ "location": [1.4, 3.2], "parking" : "false", "rating" : 5 }
{ "index": { "_index": "hotels-index", "_id": "10" } }
{ "location": [7.0, 9.9], "parking" : "true", "rating" : 9 }
{ "index": { "_index": "hotels-index", "_id": "11" } }
{ "location": [3.0, 2.3], "parking" : "false", "rating" : 6 }
{ "index": { "_index": "hotels-index", "_id": "12" } }
{ "location": [5.0, 1.0], "parking" : "true", "rating" : 3 }
```
{％包含副本-curl.html％}

**步骤3：使用过滤器搜索数据**

现在您可以创建一个K-NN搜索使用过滤器。在K中-nn查询子句，包括用于搜索最近邻居的兴趣点，返回的最近邻居的数量（`k`），以及具有限制标准的过滤器。根据您希望过滤器的限制程度，您可以在一个请求中添加多个查询子句。

以下请求会创建K-nn查询，搜索与坐标附近的前三家酒店`[5, 4]` 包括8到10之间的额定额定设施，并提供停车位：

```json
POST /hotels-index/_search
{
  "size": 3,
  "query": {
    "knn": {
      "location": {
        "vector": [
          5,
          4
        ],
        "k": 3,
        "filter": {
          "bool": {
            "must": [
              {
                "range": {
                  "rating": {
                    "gte": 8,
                    "lte": 10
                  }
                }
              },
              {
                "term": {
                  "parking": "true"
                }
              }
            ]
          }
        }
      }
    }
  }
}
```
{％包含副本-curl.html％}

响应返回最接近搜索点并符合过滤器标准的三个酒店：

```json
{
  "took" : 47,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 0.72992706,
    "hits" : [
      {
        "_index" : "hotels-index",
        "_id" : "3",
        "_score" : 0.72992706,
        "_source" : {
          "location" : [
            4.9,
            3.4
          ],
          "parking" : "true",
          "rating" : 9
        }
      },
      {
        "_index" : "hotels-index",
        "_id" : "6",
        "_score" : 0.3012048,
        "_source" : {
          "location" : [
            6.4,
            3.4
          ],
          "parking" : "true",
          "rating" : 9
        }
      },
      {
        "_index" : "hotels-index",
        "_id" : "5",
        "_score" : 0.24154587,
        "_source" : {
          "location" : [
            3.3,
            4.5
          ],
          "parking" : "true",
          "rating" : 8
        }
      }
    ]
  }
}
```

有关构建过滤器的更多方法，请参阅[构建过滤器](#constructing-a-filter)。

### faiss k-NN过滤器实现

叉-nn搜索，您可以使用`faiss` 使用HNSW算法的过滤器（K-NN插件版本2.9及以后）或IVF算法（k-NN插件版本2.10及以后）。

当您为k指定faiss滤波器时-nn搜索，faiss算法决定是否执行精确的k-nn搜索-通过修改后的过滤或近似搜索-过滤。该算法使用以下变量：

- N：索引中的文档数量。
- P：应用过滤器后的文档子集中的文档数（p <= n）。
- K：响应中返回的最大向量数。
- R：执行过滤后的近似邻居搜索后返回的结果数。
- ft（过滤阈值）：索引-在[`knn.advanced.filtered_exact_search_threshold` 环境]({{site.url}}{{site.baseurl}}/search-plugins/knn/settings/) 该指定要切换到精确的搜索。
- MDC（最大距离计算）：确切搜索中允许的最大距离计算数量`FT` （过滤阈值）未设置。该值无法更改。

以下流程图概述了FAISS算法。

![用于过滤的FAISS算法]({{site.url}}{{site.baseurl}}/images/faiss-algorithm.jpg)

### 使用faiss有效的过滤器

考虑包含有关E的不同衬衫信息的索引-商业应用。您想找到顶部-与您已经拥有但想限制衬衫尺寸的衬衫相似的衬衫。

在此示例中，您将创建一个索引并搜索与您提供的衬衫相似的衬衫。

**步骤1：创建一个新索引** 

在运行k之前-NN搜索使用过滤器，您需要使用`knn_vector` 场地。对于此字段，您需要指定`faiss` 和`hnsw` 作为`method` 在映射中。

以下请求创建一个索引，其中包含衬衫的向量表示：

```json
PUT /products-shirts
{
  "settings": {
    "index": {
      "knn": true
    }
  },
  "mappings": {
    "properties": {
      "item_vector": {
        "type": "knn_vector",
        "dimension": 3,
        "method": {
          "name": "hnsw",
          "space_type": "l2",
          "engine": "faiss"
        }
      }
    }
  }
}
```
{％包含副本-curl.html％}

**步骤2：将数据添加到您的索引**

接下来，将数据添加到您的索引中。

以下请求添加了12个文档，其中包含有关衬衫的信息，包括其向量表示，大小和评分：

```json
POST /_bulk?refresh
{ "index": { "_index": "products-shirts", "_id": "1" } }
{ "item_vector": [5.2, 4.4, 8.4], "size" : "large", "rating" : 5 }
{ "index": { "_index": "products-shirts", "_id": "2" } }
{ "item_vector": [5.2, 3.9, 2.9], "size" : "small", "rating" : 4 }
{ "index": { "_index": "products-shirts", "_id": "3" } }
{ "item_vector": [4.9, 3.4, 2.2], "size" : "xlarge", "rating" : 9 }
{ "index": { "_index": "products-shirts", "_id": "4" } }
{ "item_vector": [4.2, 4.6, 5.5], "size" : "large", "rating" : 6}
{ "index": { "_index": "products-shirts", "_id": "5" } }
{ "item_vector": [3.3, 4.5, 8.8], "size" : "medium", "rating" : 8 }
{ "index": { "_index": "products-shirts", "_id": "6" } }
{ "item_vector": [6.4, 3.4, 6.6], "size" : "small", "rating" : 9 }
{ "index": { "_index": "products-shirts", "_id": "7" } }
{ "item_vector": [4.2, 6.2, 4.6], "size" : "small", "rating" : 5 }
{ "index": { "_index": "products-shirts", "_id": "8" } }
{ "item_vector": [2.4, 4.0, 3.0], "size" : "small", "rating" : 8 }
{ "index": { "_index": "products-shirts", "_id": "9" } }
{ "item_vector": [1.4, 3.2, 9.0], "size" : "small", "rating" : 5 }
{ "index": { "_index": "products-shirts", "_id": "10" } }
{ "item_vector": [7.0, 9.9, 9.0], "size" : "xlarge", "rating" : 9 }
{ "index": { "_index": "products-shirts", "_id": "11" } }
{ "item_vector": [3.0, 2.3, 2.0], "size" : "large", "rating" : 6 }
{ "index": { "_index": "products-shirts", "_id": "12" } }
{ "item_vector": [5.0, 1.0, 4.0], "size" : "large", "rating" : 3 }

```
{％包含副本-curl.html％}

**步骤3：使用过滤器搜索数据**

现在您可以创建一个K-NN搜索使用过滤器。在K中-nn查询子句，包括用于搜索类似衬衫的衬衫的向量表示，最近的邻居数（`k`），以及按大小和评分的过滤器。

以下请求搜索尺寸的小衬衫在7到10之间，包括：

```json
POST /products-shirts/_search
{
  "size": 2,
  "query": {
    "knn": {
      "item_vector": {
        "vector": [
          2, 4, 3
        ],
        "k": 10,
        "filter": {
          "bool": {
            "must": [
              {
                "range": {
                  "rating": {
                    "gte": 7,
                    "lte": 10
                  }
                }
              },
              {
                "term": {
                  "size": "small"
                }
              }
            ]
          }
        }
      }
    }
  }
}
```
{％包含副本-curl.html％}

响应返回两个匹配文档：

```json
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.8620689,
    "hits": [
      {
        "_index": "products-shirts",
        "_id": "8",
        "_score": 0.8620689,
        "_source": {
          "item_vector": [
            2.4,
            4,
            3
          ],
          "size": "small",
          "rating": 8
        }
      },
      {
        "_index": "products-shirts",
        "_id": "6",
        "_score": 0.029691212,
        "_source": {
          "item_vector": [
            6.4,
            3.4,
            6.6
          ],
          "size": "small",
          "rating": 9
        }
      }
    ]
  }
}
```

有关构建过滤器的更多方法，请参阅[构建过滤器](#constructing-a-filter)。

### 构建过滤器

有多种方法可以为相同条件构建过滤器。例如，您可以使用以下构造来创建一个返回提供停车位的酒店的过滤器：

- A`term` 查询子句`should` 条款
- A`wildcard` 查询子句`should` 条款
- A`regexp` 查询子句`should` 条款
- A`must_not` 以消除酒店的条款`parking` 设置`false`。

以下请求说明了这四种与停车场寻找酒店的不同方式：

```json
POST /hotels-index/_search
{
  "size": 3,
  "query": {
    "knn": {
      "location": {
        "vector": [ 5.0, 4.0 ],
        "k": 3,
        "filter": {
          "bool": {
            "must": {
              "range": {
                "rating": {
                  "gte": 1,
                  "lte": 6
                }
              }
            },
            "should": [
            {
              "term": {
                "parking": "true"
              }
            },
            {
              "wildcard": {
                "parking": {
                  "value": "t*e"
                }
              }
            },
            {
              "regexp": {
                "parking": "[a-zA-Z]rue"
              }
            }
            ],
            "must_not": [
            {
              "term": {
                  "parking": "false"
              }
            }
            ],
            "minimum_should_match": 1
          }
        }
      }
    }
  }
} 
```
{％包含副本-curl.html％}

## 邮政-过滤

您可以达到帖子-用布尔过滤器或通过提供`post_filter` 范围。

### 布尔滤清器与安搜索

布尔过滤器由包含k的布尔查询组成-nn查询和过滤器。例如，以下查询搜索最接近指定的酒店`location` 然后过滤结果，以返回8到10之间的评级（包括）提供停车位：

```json
POST /hotels-index/_search
{
  "size": 3,
  "query": {
    "bool": {
      "filter": {
        "bool": {
          "must": [
            {
              "range": {
                "rating": {
                  "gte": 8,
                  "lte": 10
                }
              }
            },
            {
              "term": {
                "parking": "true"
              }
            }
          ]
        }
      },
      "must": [
        {
          "knn": {
            "location": {
              "vector": [
                5,
                4
              ],
              "k": 20
            }
          }
        }
      ]
    }
  }
}
```

响应包括包含匹配酒店的文件：

```json
{
  "took" : 95,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : 0.72992706,
    "hits" : [
      {
        "_index" : "hotels-index",
        "_id" : "3",
        "_score" : 0.72992706,
        "_source" : {
          "location" : [
            4.9,
            3.4
          ],
          "parking" : "true",
          "rating" : 9
        }
      },
      {
        "_index" : "hotels-index",
        "_id" : "6",
        "_score" : 0.3012048,
        "_source" : {
          "location" : [
            6.4,
            3.4
          ],
          "parking" : "true",
          "rating" : 9
        }
      },
      {
        "_index" : "hotels-index",
        "_id" : "5",
        "_score" : 0.24154587,
        "_source" : {
          "location" : [
            3.3,
            4.5
          ],
          "parking" : "true",
          "rating" : 8
        }
      }
    ]
  }
}
```

### 邮政-滤波器参数

如果您使用`knn` 与过滤器或其他条款一起查询（例如，`bool`，，，，`must`，，，，`match`），您可能收到的少于`k` 结果。在此示例中`post_filter` 将结果数量从2减少到1：

```json
GET my-knn-index-1/_search
{
  "size": 2,
  "query": {
    "knn": {
      "my_vector2": {
        "vector": [2, 3, 5, 6],
        "k": 2
      }
    }
  },
  "post_filter": {
    "range": {
      "price": {
        "gte": 5,
        "lte": 10
      }
    }
  }
}
```

## 评分脚本过滤器

评分脚本过滤器首先过滤文档，然后使用蛮力-力精确k-nn搜索结果。例如，以下查询搜索具有8到10之间的评级（包括）的酒店，可提供停车位，然后执行K-nn搜索以返回最接近指定的3家酒店`location`：

```json
POST /hotels-index/_search
{
  "size": 3,
  "query": {
    "script_score": {
      "query": {
        "bool": {
          "filter": {
            "bool": {
              "must": [
                {
                  "range": {
                    "rating": {
                      "gte": 8,
                      "lte": 10
                    }
                  }
                },
                {
                  "term": {
                    "parking": "true"
                  }
                }
              ]
            }
          }
        }
      },
      "script": {
        "source": "knn_score",
        "lang": "knn",
        "params": {
          "field": "location",
          "query_value": [
            5.0,
            4.0
          ],
          "space_type": "l2"
        }
      }
    }
  }
}
```
{％包含副本-curl.html％}

