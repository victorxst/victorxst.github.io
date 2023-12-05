---
layout: default
title: 训练 
parent: 训练和预测API
grand_parent: ML Commons API
has_children: true
nav_order: 10
---

# 训练 

训练API操作基于选定的算法训练模型。训练可以同步和异步进行。

#### 示例请求

以下示例使用k-表示训练索引数据的算法。

**与K一起训练-表示同步** 

```json
POST /_plugins/_ml/_train/kmeans
{
    "parameters": {
        "centroids": 3,
        "iterations": 10,
        "distance_type": "COSINE"
    },
    "input_query": {
        "_source": ["petal_length_in_cm", "petal_width_in_cm"],
        "size": 10000
    },
    "input_index": [
        "iris_data"
    ]
}
```
{% include copy-curl.html %}

**与K一起训练-表示异步**

```json
POST /_plugins/_ml/_train/kmeans?async=true
{
    "parameters": {
        "centroids": 3,
        "iterations": 10,
        "distance_type": "COSINE"
    },
    "input_query": {
        "_source": ["petal_length_in_cm", "petal_width_in_cm"],
        "size": 10000
    },
    "input_index": [
        "iris_data"
    ]
}
```
{% include copy-curl.html %}

#### 示例响应

**同步**

对于同步响应，API返回`model_id`，可用于获取或删除模型。

```json
{
  "model_id" : "lblVmX8BO5w8y8RaYYvN",
  "status" : "COMPLETED"
}
```

**异步**

对于异步响应，API返回`task_id`，可用于获取或删除任务。

```json
{
  "task_id" : "lrlamX8BO5w8y8Ra2otd",
  "status" : "CREATED"
}
```
