---
layout: default
title: 执行算法 
parent: ML Commons API
nav_order: 30
---

# 执行算法

一些算法，例如[本土化]({{site.url}}{{site.baseurl}}/ml-commons-plugin/algorithms#localization)，不需要训练有素的模型。你可以不运行-模型-使用的算法使用`execute` API。

## 路径和HTTP方法

```json
POST _plugins/_ml/_execute/<algorithm_name>
```

#### 示例请求：执行本地化

以下示例使用本地化算法查找子集-汇总数据的级别信息（例如，随着时间的推移汇总）证明了感兴趣的活动，例如尖峰，下降，变化或异常。

```json
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
{% include copy-curl.html %}

#### 示例响应

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
                "key" : [
                  "attr2"
                ],
                "contribution_value" : 1.0,
                "base_value" : 4.0,
                "new_value" : 5.0
              },
              {
                "key" : [
                  "attr3"
                ],
                "contribution_value" : 1.0,
                "base_value" : 5.0,
                "new_value" : 6.0
              },
              {
                "key" : [
                  "attr4"
                ],
                "contribution_value" : 1.0,
                "base_value" : 6.0,
                "new_value" : 7.0
              },
              {
                "key" : [
                  "attr5"
                ],
                "contribution_value" : 1.0,
                "base_value" : 7.0,
                "new_value" : 8.0
              },
              {
                "key" : [
                  "attr6"
                ],
                "contribution_value" : 1.0,
                "base_value" : 8.0,
                "new_value" : 9.0
              },
              {
                "key" : [
                  "attr7"
                ],
                "contribution_value" : 1.0,
                "base_value" : 9.0,
                "new_value" : 10.0
              },
              {
                "key" : [
                  "attr8"
                ],
                "contribution_value" : 1.0,
                "base_value" : 10.0,
                "new_value" : 11.0
              },
              {
                "key" : [
                  "attr9"
                ],
                "contribution_value" : 1.0,
                "base_value" : 11.0,
                "new_value" : 12.0
              }
            ]
          },
          ...
        ]
      }
    }
  ]
}
```


