---
layout: default
title: 训练和预测
parent: 训练和预测API
grand_parent: ML Commons API
has_children: true
nav_order: 10
---

## 训练和预测

用于训练，然后立即预测相同的培训数据集。只能与无监督的学习模型和以下算法一起使用：

- `BATCH_RCF`
- `FIT_RCF`
- `k-means`

#### 示例请求：用索引数据进行训练和预测

```json
POST /_plugins/_ml/_train_predict/kmeans
{
    "parameters": {
        "centroids": 2,
        "iterations": 10,
        "distance_type": "COSINE"
    },
    "input_query": {
        "query": {
            "bool": {
                "filter": [
                    {
                        "range": {
                            "k1": {
                                "gte": 0
                            }
                        }
                    }
                ]
            }
        },
        "size": 10
    },
    "input_index": [
        "test_data"
    ]
}
```
{% include copy-curl.html %}

#### 示例请求：直接使用数据训练和预测

```json
POST /_plugins/_ml/_train_predict/kmeans
{
    "parameters": {
        "centroids": 2,
        "iterations": 1,
        "distance_type": "EUCLIDEAN"
    },
    "input_data": {
        "column_metas": [
            {
                "name": "k1",
                "column_type": "DOUBLE"
            },
            {
                "name": "k2",
                "column_type": "DOUBLE"
            }
        ],
        "rows": [
            {
                "values": [
                    {
                        "column_type": "DOUBLE",
                        "value": 1.00
                    },
                    {
                        "column_type": "DOUBLE",
                        "value": 2.00
                    }
                ]
            },
            {
                "values": [
                    {
                        "column_type": "DOUBLE",
                        "value": 1.00
                    },
                    {
                        "column_type": "DOUBLE",
                        "value": 4.00
                    }
                ]
            },
            {
                "values": [
                    {
                        "column_type": "DOUBLE",
                        "value": 1.00
                    },
                    {
                        "column_type": "DOUBLE",
                        "value": 0.00
                    }
                ]
            },
            {
                "values": [
                    {
                        "column_type": "DOUBLE",
                        "value": 10.00
                    },
                    {
                        "column_type": "DOUBLE",
                        "value": 2.00
                    }
                ]
            },
            {
                "values": [
                    {
                        "column_type": "DOUBLE",
                        "value": 10.00
                    },
                    {
                        "column_type": "DOUBLE",
                        "value": 4.00
                    }
                ]
            },
            {
                "values": [
                    {
                        "column_type": "DOUBLE",
                        "value": 10.00
                    },
                    {
                        "column_type": "DOUBLE",
                        "value": 0.00
                    }
                ]
            }
        ]
    }
}
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "status" : "COMPLETED",
  "prediction_result" : {
    "column_metas" : [
      {
        "name" : "ClusterID",
        "column_type" : "INTEGER"
      }
    ],
    "rows" : [
      {
        "values" : [
          {
            "column_type" : "INTEGER",
            "value" : 1
          }
        ]
      },
      {
        "values" : [
          {
            "column_type" : "INTEGER",
            "value" : 1
          }
        ]
      },
      {
        "values" : [
          {
            "column_type" : "INTEGER",
            "value" : 1
          }
        ]
      },
      {
        "values" : [
          {
            "column_type" : "INTEGER",
            "value" : 0
          }
        ]
      },
      {
        "values" : [
          {
            "column_type" : "INTEGER",
            "value" : 0
          }
        ]
      },
      {
        "values" : [
          {
            "column_type" : "INTEGER",
            "value" : 0
          }
        ]
      }
    ]
  }
}
```
