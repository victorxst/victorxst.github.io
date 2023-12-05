---
layout: default
title: 支持的算法
has_children: false
nav_order: 30
---

# 支持的算法

ML Commons支持各种算法，以帮助培训和预测机器学习（ML）模型或测试数据-没有模型的驱动预测。此页面概述了ML Commons插件支持的算法及其支持的API操作。

## 共同的局限性

除本地化算法外，以下所有算法只能支持从索引中检索10,000个文档作为输入。

## k-方法

k-含义是一种简单而流行的无监督聚类ML算法[Tribuo](https://tribuo.org/) 图书馆。k-均值将随机选择质心，然后进行迭代计算以优化质心的位置，直到每个观察值属于群集的最接近平均值为止。

### 参数

范围| 类型| 描述| 默认值
：--- |:--- | :--- | :---
`centroids` | 整数| 将生成数据分组的群集数| `2` 
`iterations` | 整数| 对数据执行的迭代次数，直到平均生成| `10`
`distance_type` | 枚举，例如`EUCLIDEAN`，`COSINE`， 或者`L1` | 测量质心之间距离的测量类型| `EUCLIDEAN`

### 支持的API

*[火车]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/train-predict/train/)
*[预测]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/train-predict/predict/)
*[训练和预测]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/train-predict/train-and-predict/)

### 例子

以下示例使用虹膜数据索引来训练k-表示同步。

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

### 限制

训练过程支持多线程，但是线程的数量必须小于CPU数量的一半。

## 线性回归

线性回归映射输入和输出之间的线性关系。在ML Commons中，线性回归算法是从公共机器学习库中采用的[Tribuo](https://tribuo.org/)，提供多维线性回归模型。该模型支持训练中的线性优化器，包括诸如线性衰变，sqrt_decay，[艾达](https://www.jmlr.org/papers/volume12/duchi11a/duchi11a.pdf)，[亚当](https://tribuo.org/learn/4.1/javadoc/org/tribuo/math/optimisers/Adam.html)， 和[rms_drop](https://tribuo.org/learn/4.1/javadoc/org/tribuo/math/optimisers/RMSProp.html)。

### 参数

范围| 类型| 描述| 默认值
：--- |:--- | :--- | :---
`learningRate` | 双倍的| 迭代优化算法中使用的初始步骤大小。| `0.01`
`momentumFactor` | 双倍的| 加速调整重量速率的额外重量因子。这有助于将最小化的例程从当地的最小值中移出。| `0`
`epsilon` | 双倍的| 稳定梯度反转的值。| `1.00E-06`
`beta1` | 双倍的| 目的估计的指数衰减率。|  `0.9`
`beta2` | 双倍的| 目的估计的指数衰减率。|  `0.99`
`decayRate` | 双倍的| 根平方繁殖（RMSPROP）。| `0.9`
`momentumType` | 细绳| 定义的随机梯度下降（SGD）动量类型，有助于在正确的方向上加速梯度向量，从而导致快速收敛。| `STANDARD`
`optimizerType` | 细绳| 模型中使用的优化器。| `SIMPLE_SGD`


### 支持的API

*[火车]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/train-predict/train/)
*[预测]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/train-predict/predict/)

### 例子

以下示例基于先前训练的线性回归模型创建了一个新的预测。

#### 示例请求

```json
POST _plugins/_ml/_predict/LINEAR_REGRESSION/ROZs-38Br5eVE0lTsoD9
{
  "parameters": {
    "target": "price"
  },
  "input_data": {
    "column_metas": [
      {
        "name": "A",
        "column_type": "DOUBLE"
      },
      {
        "name": "B",
        "column_type": "DOUBLE"
      }
    ],
    "rows": [
      {
        "values": [
          {
            "column_type": "DOUBLE",
            "value": 3
          },
          {
            "column_type": "DOUBLE",
            "value": 5
          }
        ]
      }
    ]
  }
}
```

#### 示例响应

```json
{
  "status": "COMPLETED",
  "prediction_result": {
    "column_metas": [
      {
        "name": "price",
        "column_type": "DOUBLE"
      }
    ],
    "rows": [
      {
        "values": [
          {
            "column_type": "DOUBLE",
            "value": 17.25701855310131
          }
        ]
      }
    ]
  }
}
```

### 限制

ML Commons仅支持线性随机梯度培训师或优化器，这些梯度培训师或优化器无法有效地映射非-训练有素的数据中的线性关系。当与复杂的数据集一起使用时，线性随机培训师可能会导致一些收敛问题和结果不准确。

## RCF

[随机砍伐森林](https://github.com/aws/random-cut-forest-by-aws) （RCF）是一种主要用于无监督异常检测的概率数据结构。它的使用也扩展到密度估计和预测。OpenSearch利用RCF进行异常检测。ML Commons支持不同用例的两个新变体RCF：

*批处理RCF：检测非非异常-时间-系列数据。
*固定时间（fit）rcf：及时检测异常-系列数据。

### 参数

RCF支持以下参数。

#### 批次RCF

范围| 类型| 描述| 默认值
：--- |:--- | :--- | :---
`number_of_trees` | 整数| 森林中的树木数量。| `30`
`sample_size` | 整数| 森林中的流采样器使用的大小相同。| `256`
`output_after` | 整数| 结果返回之前，流采样器所需的点数。| `32`
`training_data_size` | 整数| 培训数据的大小。| 数据集大小
`anomaly_score_threshold` | 双倍的| 异常得分的阈值。| `1.0` 

#### 适合RCF

所有参数都是可选的，除了`time_field`。

范围| 类型| 描述| 默认值
：--- |:--- | :--- | :---
`number_of_trees` | 整数| 森林中的树木数量。| `30`
`shingle_size` | 整数| 最新记录的带状疱疹或连续的序列。| `8`
`sample_size` | 整数| 森林中流采样器使用的样本量。| `256`
`output_after` | 整数| 结果返回之前，流采样器所需的点数。| `32`
`time_decay` | 双倍的| 森林中流采样器使用的衰减因子。| `0.0001` 
`anomaly_rate` | 双倍的| 异常率。| `0.005`
`time_field` | 细绳| （（**必需的**）RCF用作时间的时间字段-系列数据。| N/A。
`date_format` | 细绳| 的日期和时间格式`time_field` 场地。| `yyyy-MM-ddHH:mm:ss`
`time_zone` | 细绳| 时区`time_field` 场地。| `UTC` 


### 支持的API

*[火车]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/train-predict/train/)
*[预测]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/train-predict/predict/)
*[训练和预测]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/train-predict/train-and-predict/)

### 限制

对于FIT RCF，您可以使用历史数据训练该模型，并将训练有素的模型存储在您的索引中。使用预测API时，该模型将进行应对并预测新的数据点。但是，索引中的模型不会用新数据刷新，因为该模型是及时固定的。

## RCF总结

RCF总结是一种基于代表（CURE）算法的聚类的聚类算法。相比[k-方法](#k-means)，使用随机迭代进行聚类，RCF总结使用层次聚类技术。该算法启动，一组随机选择的质心大于质心真实分布。在迭代期间，质心对彼此之间离得太近，会自动合并。因此，质心数（`max_k`）融合到适合地面真理的合理数量的群集，而不是固定的`k` 集群数。

### 参数

| 范围| 类型| 描述| 默认值|
|---|---|---|---|
| `max_k` | 整数| 最大允许的质心数。| 2|
| `distance_type` | 细绳。有效值是`EUCLIDEAN`，`L1`，`L2`， 和`LInfinity` | 用于测量质心之间距离的测量类型。| `EUCLIDEAN` |

### 支持的API

*[火车]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/train-predict/train/)
*[预测]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/train-predict/predict/)
*[训练和预测]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/train-predict/train-and-predict/)

### 示例：训练和预测

以下示例估计集群中心，并在给定数据框架中为每个样本提供群集标签。

#### 示例请求

```json
POST _plugins/_ml/_train_predict/RCF_SUMMARIZE
{
  "parameters": {
    "centroids": 3,
    "max_k": 15,
    "distance_type": "L2"
  },
  "input_data": {
    "column_metas": [
      {
        "name": "d0",
        "column_type": "DOUBLE"
      },
      {
        "name": "d1",
        "column_type": "DOUBLE"
      }
    ],
    "rows": [
      {
        "values": [
          {
            "column_type": "DOUBLE",
            "value": 6.2
          },
          {
            "column_type": "DOUBLE",
            "value": 3.4
          }
        ]
      }
    ]
  }
}
```

#### 示例响应

这`rows` 预测结果中的参数已修改为长度。在您的响应中，期望响应主体中包含更多的行和列。

```json
{
  "status": "COMPLETED",
  "prediction_result": {
    "column_metas": [
      {
        "name": "ClusterID",
        "column_type": "INTEGER"
      }
    ],
    "rows": [
      {
        "values": [
          {
            "column_type": "DOUBLE",
            "value": 0
          }
        ]
      }
    ]
  }
}
```
  

## 本土化

本地化算法找到子集-汇总数据的级别信息（例如，随着时间的推移汇总）证明了感兴趣的活动，例如尖峰，下降，变化或异常。可以在不同的情况（例如数据探索或根本原因分析）中应用定位，以暴露促进汇总数据中感兴趣活动的贡献者。

### 参数

除非`filter_query` 和`anomaly_start`。

范围| 类型| 描述| 默认值
:--- | :--- | :--- | :---
`index_name` | 细绳| 数据收集要分析。| N/A。
`attribute_field_names` | 列表| 实体密钥的字段。| N/A。
`aggregations` | 列表| 值的字段和汇总。| N/A。
`time_field_name` | 细绳| 时间戳字段。| `null`
`start_time` | 长的| 时间范围的开始。| `0` 
`end_time` | 长的| 时间范围的结束。| 0``
`min_time_interval` | 长的| 最小时间间隔/比例用于分析。| `0`
`num_outputs` | 整数| 来自本地化/切片的最大值数量。| `0`
`filter_query` | 长的| （可选）减少数据收集以进行分析。| N/A。
异常`y__star| 时间单元| （可选）将分析数据的时间。| N/A。

### 示例：执行本地化

以下示例针对RCA索引执行本地化。

#### 示例请求

```bash
POST /_plugins/_ml/_execute/anomaly_localization
{
  "index_name": "rca-index",
  "attribute_field_names": [
    "attribute"
  ],
  "aggregations": [
    {
      "sum": {
        "sum": {
          "field": "value"
        }
      }
    }
  ],
  "time_field_name": "timestamp",
  "start_time": 1620630000000,
  "end_time": 1621234800000,
  "min_time_interval": 86400000,
  "num_outputs": 10
}
```

#### 示例响应

每次在指定的时间间隔内执行算法时，API都以每个聚合的贡献和基本值的总和进行响应。

```json
{
  "results" : [
    {
      "name" : "sum",
      "result" : {
        "buckets" : [
          {
            "start_time" : 1620630000000,
            "end_time" : 1620716400000,
            "overall_aggregate_value" : 65.0
          },
          {
            "start_time" : 1620716400000,
            "end_time" : 1620802800000,
            "overall_aggregate_value" : 75.0,
            "entities" : [
              {
                "key" : [
                  "attr0"
                ],
                "contribution_value" : 1.0,
                "base_value" : 2.0,
                "new_value" : 3.0
              },
              {
                "key" : [
                  "attr1"
                ],
                "contribution_value" : 1.0,
                "base_value" : 3.0,
                "new_value" : 4.0
              },
              {
                ...
              },
             {
                "key" : [
                  "attr8"
                ],
                "contribution_value" : 6.0,
                "base_value" : 10.0,
                "new_value" : 16.0
              },
              {
                "key" : [
                  "attr9"
                ],
                "contribution_value" : 6.0,
                "base_value" : 11.0,
                "new_value" : 17.0
              }
            ]
          }
        ]
      }
    }
  ]
}
```  

### 限制

本地化算法只能直接执行。因此，它不能与ML Commons训练并预测API一起使用。

## 逻辑回归

分类算法，逻辑回归对给定输入变量的离散结果的概率进行了建模。在ML Commons中，这些分类包括二进制和多二-班级。最常见的是二进制分类，该分类需要两个值，例如"true/false" 或者"yes/no"，并根据指定的值预测结果。或者，多数-类输出可以基于类型对不同的输入进行分类。这使得逻辑回归对于您试图确定输入方式最适合指定类别的情况最有用。

### 参数

| 范围| 类型| 描述| 默认值|
|---|---|---|---|
| `learningRate` | 双倍的| 迭代优化算法中使用的初始步骤大小。| `1` |
| `momentumFactor` | 双倍的| 加速调整重量速率的额外重量因子。这有助于将最小化的例程从当地的最小值中移出。| `0` |
| `epsilon` | 双倍的| 稳定梯度反转的值。| `0.1` |
| `beta1` | 双倍的| 目的估计的指数衰减率。| `0.9` |
| `beta2` | 双倍的| 目的估计的指数衰减率。| `0.99` |
| `decayRate` | 双倍的| 根平方繁殖（RMSPROP）。| `0.9` |
| `momentumType` | 细绳| 随机梯度下降（SGD）动量，有助于沿正确方向加速梯度向量，从而导致向量之间更快地收敛。| `STANDARD` |
| `optimizerType` | 细绳| 模型中使用的优化器。| `AdaGrad` |
| `target` | 细绳| 目标字段。| 无效的|
| `objectiveType` | 细绳| 目标函数类型。| `LogMulticlass` |
| `epochs` | 整数| 迭代次数。| `5` |
| `batchSize` | 整数| 最小批次的大小。| `1` |
| `loggingInterval` | 整数| 经过许多迭代后，日志的间隔丢失。间隔是`1` 如果该算法不包含日志。| `1000` |

### 支持的API

*[火车]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/train-predict/train/)
*[预测]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/train-predict/predict/)

### 示例：使用虹膜数据进行火车/预测

以下示例在opensearch中创建了一个索引[虹膜数据集](https://en.wikipedia.org/wiki/Iris_flower_data_set)，然后使用逻辑回归训练数据。最后，它使用训练有素的模型预测由行分开的虹膜类型。

#### 创建虹膜指数

在使用此请求之前，请确保您已下载[虹膜数据](https://archive.ics.uci.edu/dataset/53/iris)。

```bash
PUT /iris_data
{
  "mappings": {
    "properties": {
      "sepal_length_in_cm": {
        "type": "double"
      },
      "sepal_width_in_cm": {
        "type": "double"
      },
      "petal_length_in_cm": {
        "type": "double"
      },
      "petal_width_in_cm": {
        "type": "double"
      },
      "class": {
        "type": "keyword"
      }
    }
  }
}
```

#### 来自iris_data.txt的摄入数据

```bash
POST _bulk
{ "index" : { "_index" : "iris_data" } }
{"sepal_length_in_cm":5.1,"sepal_width_in_cm":3.5,"petal_length_in_cm":1.4,"petal_width_in_cm":0.2,"class":"Iris-setosa"}
{ "index" : { "_index" : "iris_data" } }
{"sepal_length_in_cm":4.9,"sepal_width_in_cm":3.0,"petal_length_in_cm":1.4,"petal_width_in_cm":0.2,"class":"Iris-setosa"}
...
...
```

#### 训练逻辑回归模型

此示例使用多个-类逻辑回归分类方法。在这里，使用棕褐色，花瓣长度和宽度的输入来训练模型以基于质心分类`class`，如`target` 范围。

**要求**

```bash
{
  "parameters": {
    "target": "class"
  },
  "input_query": {
    "query": {
      "match_all": {}
    },
    "_source": [
      "sepal_length_in_cm",
      "sepal_width_in_cm",
      "petal_length_in_cm",
      "petal_width_in_cm",
      "class"
    ],
    "size": 200
  },
  "input_index": [
    "iris_data"
  ]
}
```

#### 示例响应

这`model_id` 将用于预测虹膜的类。

```json
{
  "model_id" : "TOgsf4IByBqD7FK_FQGc",
  "status" : "COMPLETED"
}
```

#### 预测结果

使用`model_id` 在训练有素的IRIS数据集中，逻辑回归将根据输入数据预测虹膜的类别。

```bash
POST _plugins/_ml/_predict/logistic_regression/SsfQaoIBEoC4g4joZiyD
{
  "parameters": {
    "target": "class"
  },
  "input_data": {
    "column_metas": [
      {
        "name": "sepal_length_in_cm",
        "column_type": "DOUBLE"
      },
      {
        "name": "sepal_width_in_cm",
        "column_type": "DOUBLE"
      },
      {
        "name": "petal_length_in_cm",
        "column_type": "DOUBLE"
      },
      {
        "name": "petal_width_in_cm",
        "column_type": "DOUBLE"
      }
    ],
    "rows": [
      {
        "values": [
          {
            "column_type": "DOUBLE",
            "value": 6.2
          },
          {
            "column_type": "DOUBLE",
            "value": 3.4
          },
          {
            "column_type": "DOUBLE",
            "value": 5.4
          },
          {
            "column_type": "DOUBLE",
            "value": 2.3
          }
        ]
      },
      {
        "values": [
          {
            "column_type": "DOUBLE",
            "value": 5.9
          },
          {
            "column_type": "DOUBLE",
            "value": 3.0
          },
          {
            "column_type": "DOUBLE",
            "value": 5.1
          },
          {
            "column_type": "DOUBLE",
            "value": 1.8
          }
        ]
      }
    ]
  }
}
```

#### 示例响应

```json
{
  "status" : "COMPLETED",
  "prediction_result" : {
    "column_metas" : [
      {
        "name" : "result",
        "column_type" : "STRING"
      }
    ],
    "rows" : [
      {
        "values" : [
          {
            "column_type" : "STRING",
            "value" : "Iris-virginica"
          }
        ]
      },
      {
        "values" : [
          {
            "column_type" : "STRING",
            "value" : "Iris-virginica"
          }
        ]
      }
    ]
  }
}
```

### 限制

融合指标不在Tribuo的培训师中内置。因此，ML Commons不能通过ML Commons API指示收敛状态。

## 指标相关

指标相关功能是OpenSearch 2.7中发布的实验功能。它不能在生产环境中使用。要留下有关改进功能的反馈，请在[ML Commons存储库](https://github.com/opensearch-project/ml-commons)。
{: .warning }

指标相关算法在一组指标数据中找到事件。该算法将事件定义为时间窗口，其中多个指标同时显示异常行为。当给出一组指标时，该算法计算发生的事件数量，每个事件发生时，并确定每个事件中涉及哪些指标。

要启用指标相关算法，请更新以下群集设置：

```json
PUT /_cluster/settings
{
  "persistent" : {
    "plugins.ml_commons.enable_inhouse_python_model": true
  }
}
```

### 参数

要使用指标相关算法，请包括以下参数。

| 范围| 类型| 描述| 默认值|
|---|---|---|---|
指标| 大批| 时间序列中可以与异常行为相关的指标列表| N/A。

### 输入

指标相关输入是指标数据的m x t阵列，其中m是指标的数量，t是每个度量值的长度。

将指标输入算法时，假设以下内容：

1. 对于每个度量标准，输入序列具有相同的长度。
2. 所有输入指标应具有相同的相应时间戳集。
3. 数据点的总数为m * t <= 10000。

### 示例：简单的指标相关性

以下示例将指标数（M）的数量作为3，时间步长（t）的数量为128：

```json
POST /_plugins/_ml/_execute/METRICS_CORRELATION
{"metrics": [[-1.1635416, -1.5003631, 0.46138194, 0.5308311, -0.83149344, -3.7009873, -3.5463789, 0.22571462, -5.0380244, 0.76588845, 1.236113, 1.8460795, 1.7576948, 0.44893077, 0.7363948, 0.70440894, 0.89451003, 4.2006273, 0.3697659, 2.2458954, -2.302939, -1.7706926, 1.7445002, -1.5246059, 0.07985192, -2.7756078, 1.0002468, 1.5977372, 2.9152713, 1.4172368, -0.26551363, -2.2883027, 1.5882446, 2.0145164, 3.4862874, -1.2486862, -2.4811826, -0.17609037, -2.1095612, -1.2184235, 0.63118523, -1.8909532, 2.039797, -0.5317177, -2.2922578, -2.0179775, -0.07992507, -0.12554549, -0.2553092, 1.1450123, -0.4640453, -2.190223, -4.671612, -1.5076426, 1.635445, -1.1394824, -0.7503817, 0.98424894, -0.38896716, 1.0328646, 1.9543738, -0.5236269, 0.14298044, 3.2963762, 8.1641035, 5.717064, 7.4869685, 2.5987444, 11.018798, 9.151356, 5.7354255, 6.862203, 3.0524514, 4.431755, 5.1481285, 7.9548607, 7.4519925, 6.09533, 7.634116, 8.898271, 3.898491, 9.447067, 8.197385, 5.8284273, 5.804283, 7.7688456, 10.574343, 7.5679493, 7.1888094, 7.1107903, 8.454468, 8.066334, 8.83665, 7.11204, 4.4898267, 8.614764, 6.336754, 11.577503, 3.3998494, 9.501525, 13.17289, 6.1116023, 5.143777, 2.7813284, 3.7917604, 7.1683135, 7.627272, 7.290255, 3.1299121, 7.089733, 9.140584, 8.844729, 9.403275, 10.220029, 8.039719, 8.85549, 4.034555, 4.412663, 7.54451, 7.2116737, 4.6346946, 7.0044127, 9.7557, 10.982841, 5.897937, 6.870126, 3.5638695, 5.7872133], [1.3037996, 2.7976995, -0.12042701, 1.3688855, 1.6955005, -2.2575269, 0.080582514, 3.011721, -0.4320283, 3.2440786, -1.0321085, 1.2346085, -2.3152106, -0.9783513, 0.6837618, 1.5320586, -1.6148578, -0.94538075, 0.55978125, -4.7430468, 3.466028, 2.3792691, 1.3269067, -0.35359794, -1.5547276, 0.5202475, 1.0269136, -1.7531714, 0.43987304, -0.18845831, 2.3086758, 2.519588, 2.0116413, 0.019745048, -0.010070452, 2.496933, 1.1557871, 0.08433053, 1.375894, -1.2135965, -1.2588277, -0.31454003, 0.045949124, -1.7518936, -2.3533764, -2.0125146, 0.10255043, 1.1782314, 2.4579153, -0.8780899, -4.1442213, 3.8300152, 2.772975, 2.6803262, 0.9867382, 0.77618766, 0.46541777, 3.8959959, -2.1713195, 0.10609512, -0.26438138, -2.145317, 3.6734529, 1.4830295, -5.3445525, -10.6427765, -8.300354, -1.9608921, -6.6779685, -10.019544, -8.341513, -9.607174, -7.2441607, -3.411102, -6.180552, -8.318714, -6.060591, -7.790343, -5.9695, -7.9429936, -3.775652, -5.2827606, -3.7168224, -6.729588, -9.761094, -7.4683576, -7.2595067, -6.6790915, -9.832726, -8.352172, -6.936336, -8.252518, -6.787475, -9.091013, -11.465944, -6.712504, -8.987438, -6.946672, -8.877166, -6.7854185, -3.6417139, -6.1036086, -5.360772, -4.0435786, -4.5864973, -6.971063, -10.522461, -6.3692527, -4.387658, -9.723745, -4.7020173, -5.097396, -9.903703, -4.882414, -4.1999683, -6.7829437, -6.2555966, -8.121125, -5.334131, -9.174302, -3.9752126, -4.179469, -8.335524, -9.359406, -6.4938803, -6.794677, -8.382997, -9.879416], [1.8792984, -3.1561708, -0.8443318, -1.998743, -0.6319316, 2.4614046, -0.44511616, 0.82785237, 1.7911717, -1.8172283, 0.46574894, -1.8691323, 3.9586513, 0.8078605, 0.9049874, 5.4086914, -0.7425967, -0.20115769, -1.197923, 2.741789, 0.85432875, -1.1688408, -1.7771784, 1.615249, -4.1103697, 0.4721327, -2.75669, -0.38393462, -3.1137516, -2.2572582, 0.9580673, -3.7139492, -0.68303126, 1.6007807, 0.6313973, -2.5115106, 0.703251, 2.4844077, -1.7405633, -3.007687, 2.372802, 2.4684637, 0.6443977, -3.1433117, 0.05976736, -1.9809214, 3.514713, 2.1880944, 1.242541, 1.8236228, 0.8642841, -0.17313614, 1.7042321, 0.8298376, 4.2443194, 0.13983983, 1.1940852, 2.5076652, 39.285202, 82.73858, 44.707516, -4.267148, 0.25930226, 0.20799652, -3.7213502, 1.475217, -1.2394199, -0.0034497892, 1.1413965, 55.18923, -2.2969518, -4.1400924, -2.4707043, 43.193188, -0.19258368, 3.471275, 1.1374166, 1.2147579, 4.13017, -2.0576499, 2.1529694, -0.28360432, 0.8477302, -0.63012695, 1.2569811, 1.943168, 0.17070436, 3.2358394, -2.3737662, 0.77060974, 4.99065, 3.1079204, 3.6347675, 0.6801177, -2.2205186, 1.0961101, -2.4445753, -2.0919478, -2.895031, 2.5458927, 0.38599384, 1.0492333, -0.081834644, -7.4079595, -2.1785216, -0.7277175, -2.7413428, -3.2083786, 3.2958643, -1.1839997, 5.4849496, 2.0259023, 5.607272, -1.0125756, 3.721461, 2.5715313, 0.7741753, -0.55034757, 0.7526307, -2.6758716, -2.964664, -0.57379586, -0.28817406, -3.2334063, -0.22387607, -2.0793931, -6.4562697, 0.80134094]]}
```

#### 示例响应

API返回以下信息：

- `event_window`：事件间隔
- `event_pattern`：整个时间窗口的强度得分和事件的整体严重性
- `suspected_metrics`：涉及的一组指标

在以下示例响应中，每个项目对应于指标数据中发现的事件。该算法在请求的输入数据中找到一个事件，如输出所示`event_pattern` 长度`1`。`event_window` 表明该事件发生在时间点$ t $ = 52和$ t $ = 72之间`suspected_metrics` 表明该活动涉及所有三个指标。

```json
{
  "function_name": "METRICS_CORRELATION",
  "output": {
    "inference_results": [
      {
        "event_window": [
          52,
          72
        ],
        "event_pattern": [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 3.99625e-05, 0.0001052875, 0.0002605894, 0.00064648513, 0.0014303402, 0.002980127, 0.005871893, 0.010885878, 0.01904726, 0.031481907, 0.04920215, 0.07283493, 0.10219432, 0.1361888, 0.17257516, 0.20853643, 0.24082609, 0.26901975, 0.28376183, 0.29364157, 0.29541212, 0.2832976, 0.29041746, 0.2574534, 0.2610143, 0.22938538, 0.19999361, 0.18074994, 0.15539801, 0.13064545, 0.10544432, 0.081248805, 0.05965102, 0.041305058, 0.027082501, 0.01676033, 0.009760197, 0.005362286, 0.0027713624, 0.0013381141, 0.0006126331, 0.0002634901, 0.000106459476, 4.0407333e-05, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        "suspected_metrics": [0,1,2]
      }
    ]
  }
}
```



