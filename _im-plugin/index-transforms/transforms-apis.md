---
layout: default
title: 转换 APIs
nav_order: 45
parent: 索引转换
has_toc: true
---

# 转换 API

除了使用 OpenSearch 控制面板之外，你还可以使用 REST API 创建、启动、停止和完成与转换作业相关的其他操作。

#### 目录
- 目录
{:toc}

## 创建转换作业
引入 1.0 {：.label .label-purple }

创建转换作业。

### 请求格式

```json
PUT _plugins/_transform/<transform_id>
```

### 路径参数

参数 | 数据类型 | 描述
:--- | :--- | :---
transform_id | 字符串 | 转换 ID |

### 请求正文字段

你可以在 HTTP 请求正文中指定以下选项：

选项 | 数据类型 | 描述 | 必填
:--- | :--- | :--- | :---
已启用 | 布尔值 | 如果为 true，则在创建时启用转换作业。| 不
连续 | 布尔值 | 指定转换作业是否应是连续的。每次根据 `schedule` 字段进行调度时，都会执行连续作业，并根据新转换的存储桶以及添加到源索引的任何新数据运行。非连续作业仅执行一次。默认值为 false。| 不
日程安排 | 对象 | 转换作业的计划。| 是的
start_time | 整数 | 转换作业开始时间的 Unix 纪元时间。| 是的
描述 | 字符串 | 描述转换作业。| 不
metadata_id | 字符串 | 要与转换作业关联的任何元数据。| 不
source_index | 字符串 | 包含要转换的数据的源索引。| 是的
target_index | 字符串 | 将新转换的数据添加到的目标索引。你可以创建新索引或更新现有索引。| 是的
data_selection_query | 对象 | 用于筛选转换作业的源索引子集的查询 DSL。有关详细信息，请参阅[查询域特定语言（DSL）]({{site.url}}{{site.baseurl}}/opensearch/query-dsl)。| 是的
page_size | 整数 |IM 并发处理和索引的存储桶数。数字越大，性能越好，但需要更多的内存。如果计算机内存不足，索引管理（IM）会自动调整此字段并重试，直到操作成功。| 是的
团体 | 阵列 | 指定要在转换作业中使用的分组。支持的组为 `terms`、 `histogram` 和 `date_histogram`。有关详细信息，请参见[存储桶聚合]({{site.url}}{{site.baseurl}}/opensearch/bucket-agg). | 如果不使用聚合，则可以。
source_field | 字符串 | 要转换的字段。| 是的
聚合 | 对象 | 要在转换作业中使用的聚合。支持的聚合包括 `sum`、、、 `value_count`、 `max` `min` `avg` 和 `scripted_metric` `percentiles`。有关详细信息，请参见[指标聚合]({{site.url}}{{site.baseurl}}/opensearch/metric-agg). | 如果不使用组，则可以。

#### 样品申请

以下请求创建一个 id `sample` 为：

```json
PUT _plugins/_transform/sample
{
  "transform": {
    "enabled": true,
    "continuous": true,
    "schedule": {
      "interval": {
        "period": 1,
        "unit": "Minutes",
        "start_time": 1602100553
      }
    },
    "description": "Sample transform job",
    "source_index": "sample_index",
    "target_index": "sample_target",
    "data_selection_query": {
      "match_all": {}
    },
    "page_size": 1,
    "groups": [
      {
        "terms": {
          "source_field": "customer_gender",
          "target_field": "gender"
        }
      },
      {
        "terms": {
          "source_field": "day_of_week",
          "target_field": "day"
        }
      }
    ],
    "aggregations": {
      "quantity": {
        "sum": {
          "field": "total_quantity"
        }
      }
    }
  }
}
```

#### 示例响应

```json
{
  "_id": "sample",
  "_version": 7,
  "_seq_no": 13,
  "_primary_term": 1,
  "transform": {
    "transform_id": "sample",
    "schema_version": 7,
    "continuous": true,
    "schedule": {
      "interval": {
        "start_time": 1621467964243,
        "period": 1,
        "unit": "Minutes"
      }
    },
    "metadata_id": null,
    "updated_at": 1621467964243,
    "enabled": true,
    "enabled_at": 1621467964243,
    "description": "Sample transform job",
    "source_index": "sample_index",
    "data_selection_query": {
      "match_all": {
        "boost": 1.0
      }
    },
    "target_index": "sample_target",
    "roles": [],
    "page_size": 1,
    "groups": [
      {
        "terms": {
          "source_field": "customer_gender",
          "target_field": "gender"
        }
      },
      {
        "terms": {
          "source_field": "day_of_week",
          "target_field": "day"
        }
      }
    ],
    "aggregations": {
      "quantity": {
        "sum": {
          "field": "total_quantity"
        }
      }
    }
  }
}
```

## 更新转换作业
引入 1.0 {：.label .label-purple }

更新转换作业（如果 `transform_id` 已存在）。对于此请求，必须指定要更新的转换的序列号和主要术语。若要获取这些内容，请使用[获取转换作业的详细信息](#get-a-transform-jobs-details) API 调用。

### 请求格式

```json
PUT _plugins/_transform/<transform_id>?if_seq_no=<seq_no>&if_primary_term=<primary_term>
```

### 查询参数

更新操作支持以下查询参数：

参数 | 描述 | 必填
:---| :--- | :---
 `seq_no` | 仅当更改转换作业的最后一个操作具有指定的序列号时，才执行转换操作。| 是的
 `primary_term` | 仅当更改转换作业的最后一个操作具有指定的序列项时，才执行转换操作。| 是的

### 请求正文字段

你可以更新以下字段。

选项 | 数据类型 | 描述
:--- | :--- | :---
日程安排 | 对象 | 转换作业的计划。包含字段 `interval.start_time`、 `interval.period` 和 `interval.unit`。
start_time | 整数 | 转换作业的 Unix 纪元开始时间。
期间 | 整数 | 执行转换作业的频率。
单位 | 字符串 | 与执行周期关联的时间单位。可用选项包括 `Minutes`、 `Hours` 和 `Days`。
描述 | 整数 | 描述转换作业。
page_size | 整数 |IM 并发处理和索引的存储桶数。数字越大，性能越好，但需要更多的内存。如果计算机内存不足，IM 会自动调整此字段并重试，直到操作成功。

#### 样品申请

以下请求使用 id `sample`、序列号 `13` 和 primary 术语 `1` 更新转换作业：

```json
PUT _plugins/_transform/sample?if_seq_no=13&if_primary_term=1
{
  "transform": {
  "enabled": true,
  "schedule": {
    "interval": {
    "period": 1,
    "unit": "Minutes",
    "start_time": 1602100553
    }
  },
  "description": "Sample transform job",
  "source_index": "sample_index",
  "target_index": "sample_target",
  "data_selection_query": {
    "match_all": {}
  },
  "page_size": 1,
  "groups": [
    {
    "terms": {
      "source_field": "customer_gender",
      "target_field": "gender"
    }
    },
    {
    "terms": {
      "source_field": "day_of_week",
      "target_field": "day"
    }
    }
  ],
  "aggregations": {
    "quantity": {
    "sum": {
      "field": "total_quantity"
    }
    }
  }
  }
}
```

#### 示例响应

```json
PUT _plugins/_transform/sample?if_seq_no=13&if_primary_term=1
{
  "transform": {
    "enabled": true,
    "schedule": {
      "interval": {
        "period": 1,
        "unit": "Minutes",
        "start_time": 1602100553
      }
    },
    "description": "Sample transform job",
    "source_index": "sample_index",
    "target_index": "sample_target",
    "data_selection_query": {
      "match_all": {}
    },
    "page_size": 1,
    "groups": [
      {
        "terms": {
          "source_field": "customer_gender",
          "target_field": "gender"
        }
      },
      {
        "terms": {
          "source_field": "day_of_week",
          "target_field": "day"
        }
      }
    ],
    "aggregations": {
      "quantity": {
        "sum": {
          "field": "total_quantity"
        }
      }
    }
  }
}
```

## 获取转换作业的详细信息
引入 1.0 {：.label .label-purple }

返回转换作业的详细信息。

### 请求格式

```json
GET _plugins/_transform/<transform_id>
```

#### 样品申请

以下请求返回 ID `sample` 为以下转换作业的详细信息：

```json
GET _plugins/_transform/sample
```

#### 示例响应

```json
{
  "_id": "sample",
  "_version": 7,
  "_seq_no": 13,
  "_primary_term": 1,
  "transform": {
    "transform_id": "sample",
    "schema_version": 7,
    "continuous": true,
    "schedule": {
      "interval": {
        "start_time": 1621467964243,
        "period": 1,
        "unit": "Minutes"
      }
    },
    "metadata_id": null,
    "updated_at": 1621467964243,
    "enabled": true,
    "enabled_at": 1621467964243,
    "description": "Sample transform job",
    "source_index": "sample_index",
    "data_selection_query": {
      "match_all": {
        "boost": 1.0
      }
    },
    "target_index": "sample_target",
    "roles": [],
    "page_size": 1,
    "groups": [
      {
        "terms": {
          "source_field": "customer_gender",
          "target_field": "gender"
        }
      },
      {
        "terms": {
          "source_field": "day_of_week",
          "target_field": "day"
        }
      }
    ],
    "aggregations": {
      "quantity": {
        "sum": {
          "field": "total_quantity"
        }
      }
    }
  }
}
```

你还可以通过省略 `transform_id` 来获取所有转换作业的详细信息。

#### 样品申请

以下请求返回所有转换作业的详细信息：

```json
GET _plugins/_transform/
```

#### 示例响应

```json
{
  "total_transforms": 1,
  "transforms": [
    {
      "_id": "sample",
      "_seq_no": 13,
      "_primary_term": 1,
      "transform": {
        "transform_id": "sample",
        "schema_version": 7,
        "continuous": true,
        "schedule": {
          "interval": {
            "start_time": 1621467964243,
            "period": 1,
            "unit": "Minutes"
          }
        },
        "metadata_id": null,
        "updated_at": 1621467964243,
        "enabled": true,
        "enabled_at": 1621467964243,
        "description": "Sample transform job",
        "source_index": "sample_index",
        "data_selection_query": {
          "match_all": {
            "boost": 1.0
          }
        },
        "target_index": "sample_target",
        "roles": [],
        "page_size": 1,
        "groups": [
          {
            "terms": {
              "source_field": "customer_gender",
              "target_field": "gender"
            }
          },
          {
            "terms": {
              "source_field": "day_of_week",
              "target_field": "day"
            }
          }
        ],
        "aggregations": {
          "quantity": {
            "sum": {
              "field": "total_quantity"
            }
          }
        }
      }
    }
  ]
}
```

### 查询参数

你可以指定以下 GET API 操作的查询参数来筛选结果。

参数 | 描述 | 必填
:--- | :--- | :---
起 | 要返回的起始转换。默认值为 0. | 不
尺寸 | 指定要返回的转换数。默认值为 10. | 不
搜索 | 用于筛选结果的搜索词。| 不
sort 字段 | 用于对结果进行排序的字段。| 不
排序方向 | 指定对结果进行排序的方向。可以是 `ASC` 或 `DESC`.默认值为 ASC。| 不

#### 样品申请

以下请求从 transform `8` 开始返回两个结果：

```json
GET _plugins/_transform?size=2&from=8
```

#### 示例响应

```json
{
  "total_transforms": 18,
  "transforms": [
    {
      "_id": "sample8",
      "_seq_no": 93,
      "_primary_term": 1,
      "transform": {
        "transform_id": "sample8",
        "schema_version": 7,
        "schedule": {
          "interval": {
            "start_time": 1622063596812,
            "period": 1,
            "unit": "Minutes"
          }
        },
        "metadata_id": "y4hFAB2ZURQ2dzY7BAMxWA",
        "updated_at": 1622063657233,
        "enabled": false,
        "enabled_at": null,
        "description": "Sample transform job",
        "source_index": "sample_index3",
        "data_selection_query": {
          "match_all": {
            "boost": 1.0
          }
        },
        "target_index": "sample_target3",
        "roles": [],
        "page_size": 1,
        "groups": [
          {
            "terms": {
              "source_field": "customer_gender",
              "target_field": "gender"
            }
          },
          {
            "terms": {
              "source_field": "day_of_week",
              "target_field": "day"
            }
          }
        ],
        "aggregations": {
          "quantity": {
            "sum": {
              "field": "total_quantity"
            }
          }
        }
      }
    },
    {
      "_id": "sample9",
      "_seq_no": 98,
      "_primary_term": 1,
      "transform": {
        "transform_id": "sample9",
        "schema_version": 7,
        "schedule": {
          "interval": {
            "start_time": 1622063598065,
            "period": 1,
            "unit": "Minutes"
          }
        },
        "metadata_id": "x8tCIiYMTE3veSbIJkit5A",
        "updated_at": 1622063658388,
        "enabled": false,
        "enabled_at": null,
        "description": "Sample transform job",
        "source_index": "sample_index4",
        "data_selection_query": {
          "match_all": {
            "boost": 1.0
          }
        },
        "target_index": "sample_target4",
        "roles": [],
        "page_size": 1,
        "groups": [
          {
            "terms": {
              "source_field": "customer_gender",
              "target_field": "gender"
            }
          },
          {
            "terms": {
              "source_field": "day_of_week",
              "target_field": "day"
            }
          }
        ],
        "aggregations": {
          "quantity": {
            "sum": {
              "field": "total_quantity"
            }
          }
        }
      }
    }
  ]
}
```

## 启动转换作业
引入 1.0 {：.label .label-purple }

使用 API 创建的转换作业会自动启用，但如果需要启用作业，可以使用启动 API 操作。

### 请求格式

```
POST _plugins/_transform/<transform_id>/_start
```

#### 样品申请

以下请求使用 ID `sample` 启动转换作业：

```json
POST _plugins/_transform/sample/_start
```

#### 示例响应

```json
{
  "acknowledged": true
}
```

## 停止转换作业
引入 1.0 {：.label .label-purple }

停止转换作业。

### 请求格式

```
POST _plugins/_transform/<transform_id>/_stop
```

#### 样品申请

以下请求停止 ID 为以下 ID `sample` 的转换作业：

```json
POST _plugins/_transform/sample/_stop
```

#### 示例响应

```json
{
  "acknowledged": true
}
```

## 获取转换作业的状态
引入 1.0 {：.label .label-purple }

返回转换作业的状态和元数据。

### 请求格式

```
GET _plugins/_transform/<transform_id>/_explain
```

#### 样品申请

以下请求返回 ID `sample` 为以下转换作业的详细信息：

```json
GET _plugins/_transform/sample/_explain
```

#### 示例响应

```json
{
  "sample": {
    "metadata_id": "PzmjweME5xbgkenl9UpsYw",
    "transform_metadata": {
      "continuous_stats": {
        "last_timestamp": 1621883525672,
        "documents_behind": {
          "sample_index": 72
          }
      },
      "transform_id": "sample",
      "last_updated_at": 1621883525873,
      "status": "finished",
      "failure_reason": "null",
      "stats": {
        "pages_processed": 0,
        "documents_processed": 0,
        "documents_indexed": 0,
        "index_time_in_millis": 0,
        "search_time_in_millis": 0
      }
    }
  }
}
```

## 预览转换作业的结果
引入 1.0 {：.label .label-purple }

返回转换后的索引外观的预览。

#### 样品申请

```json
POST _plugins/_transform/_preview

{
  "transform": {
  "enabled": false,
  "schedule": {
    "interval": {
    "period": 1,
    "unit": "Minutes",
    "start_time": 1602100553
    }
  },
  "description": "test transform",
  "source_index": "sample_index",
  "target_index": "sample_target",
  "data_selection_query": {
    "match_all": {}
  },
  "page_size": 10,
  "groups": [
    {
    "terms": {
      "source_field": "customer_gender",
      "target_field": "gender"
    }
    },
    {
    "terms": {
      "source_field": "day_of_week",
      "target_field": "day"
    }
    }
  ],
  "aggregations": {
    "quantity": {
    "sum": {
      "field": "total_quantity"
    }
    }
  }
  }
}
```

#### 示例响应

```json
{
  "documents" : [
  {
    "quantity" : 862.0,
    "gender" : "FEMALE",
    "day" : "Friday"
  },
  {
    "quantity" : 682.0,
    "gender" : "FEMALE",
    "day" : "Monday"
  },
  {
    "quantity" : 772.0,
    "gender" : "FEMALE",
    "day" : "Saturday"
  },
  {
    "quantity" : 669.0,
    "gender" : "FEMALE",
    "day" : "Sunday"
  },
  {
    "quantity" : 887.0,
    "gender" : "FEMALE",
    "day" : "Thursday"
  }
  ]
}
```

## 删除转换作业
引入 1.0 {：.label .label-purple }

删除转换作业。此操作不会删除源索引或目标索引。

### 请求格式

```
DELETE _plugins/_transform/<transform_id>
```

#### 样品申请

以下请求删除 ID `sample` 为：

```json
DELETE _plugins/_transform/sample
```

#### 示例响应

```json
{
  "took": 205,
  "errors": false,
  "items": [
    {
      "delete": {
        "_index": ".opensearch-ism-config",
        "_id": "sample",
        "_version": 4,
        "result": "deleted",
        "forced_refresh": true,
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 6,
        "_primary_term": 1,
        "status": 200
      }
    }
  ]
}
```
