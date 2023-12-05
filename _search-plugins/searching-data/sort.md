---
layout: default
title: 排序结果
parent: 搜索数据
nav_order: 22
redirect_from:
  - /opensearch/search/sort/
---

## 排序结果

排序使您的用户可以以对他们最有意义的方式进行排序。

默认情况下，完整-文本查询按相关得分进行排序结果。
您可以通过设置以下`order` 参数为`asc` 或者`desc`。

例如，通过下降顺序排序结果`line_id` 值，使用以下查询：

```json
GET shakespeare/_search
{
  "query": {
    "term": {
      "play_name": {
        "value": "Henry IV"
      }
    }
  },
  "sort": [
    {
      "line_id": {
        "order": "desc"
      }
    }
  ]
}
```

结果由`line_id` 按顺序下降：

```json
{
  "took" : 24,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3205,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "shakespeare",
        "_id" : "3204",
        "_score" : null,
        "_source" : {
          "type" : "line",
          "line_id" : 3205,
          "play_name" : "Henry IV",
          "speech_number" : 8,
          "line_number" : "",
          "speaker" : "KING HENRY IV",
          "text_entry" : "Exeunt"
        },
        "sort" : [
          3205
        ]
      },
      {
        "_index" : "shakespeare",
        "_id" : "3203",
        "_score" : null,
        "_source" : {
          "type" : "line",
          "line_id" : 3204,
          "play_name" : "Henry IV",
          "speech_number" : 8,
          "line_number" : "5.5.45",
          "speaker" : "KING HENRY IV",
          "text_entry" : "Let us not leave till all our own be won."
        },
        "sort" : [
          3204
        ]
      },
      {
        "_index" : "shakespeare",
        "_id" : "3202",
        "_score" : null,
        "_source" : {
          "type" : "line",
          "line_id" : 3203,
          "play_name" : "Henry IV",
          "speech_number" : 8,
          "line_number" : "5.5.44",
          "speaker" : "KING HENRY IV",
          "text_entry" : "And since this business so fair is done,"
        },
        "sort" : [
          3203
        ]
      },
      {
        "_index" : "shakespeare",
        "_id" : "3201",
        "_score" : null,
        "_source" : {
          "type" : "line",
          "line_id" : 3202,
          "play_name" : "Henry IV",
          "speech_number" : 8,
          "line_number" : "5.5.43",
          "speaker" : "KING HENRY IV",
          "text_entry" : "Meeting the cheque of such another day:"
        },
        "sort" : [
          3202
        ]
      },
      {
        "_index" : "shakespeare",
        "_id" : "3200",
        "_score" : null,
        "_source" : {
          "type" : "line",
          "line_id" : 3201,
          "play_name" : "Henry IV",
          "speech_number" : 8,
          "line_number" : "5.5.42",
          "speaker" : "KING HENRY IV",
          "text_entry" : "Rebellion in this land shall lose his sway,"
        },
        "sort" : [
          3201
        ]
      },
      {
        "_index" : "shakespeare",
        "_id" : "3199",
        "_score" : null,
        "_source" : {
          "type" : "line",
          "line_id" : 3200,
          "play_name" : "Henry IV",
          "speech_number" : 8,
          "line_number" : "5.5.41",
          "speaker" : "KING HENRY IV",
          "text_entry" : "To fight with Glendower and the Earl of March."
        },
        "sort" : [
          3200
        ]
      },
      {
        "_index" : "shakespeare",
        "_id" : "3198",
        "_score" : null,
        "_source" : {
          "type" : "line",
          "line_id" : 3199,
          "play_name" : "Henry IV",
          "speech_number" : 8,
          "line_number" : "5.5.40",
          "speaker" : "KING HENRY IV",
          "text_entry" : "Myself and you, son Harry, will towards Wales,"
        },
        "sort" : [
          3199
        ]
      },
      {
        "_index" : "shakespeare",
        "_id" : "3197",
        "_score" : null,
        "_source" : {
          "type" : "line",
          "line_id" : 3198,
          "play_name" : "Henry IV",
          "speech_number" : 8,
          "line_number" : "5.5.39",
          "speaker" : "KING HENRY IV",
          "text_entry" : "Who, as we hear, are busily in arms:"
        },
        "sort" : [
          3198
        ]
      },
      {
        "_index" : "shakespeare",
        "_id" : "3196",
        "_score" : null,
        "_source" : {
          "type" : "line",
          "line_id" : 3197,
          "play_name" : "Henry IV",
          "speech_number" : 8,
          "line_number" : "5.5.38",
          "speaker" : "KING HENRY IV",
          "text_entry" : "To meet Northumberland and the prelate Scroop,"
        },
        "sort" : [
          3197
        ]
      },
      {
        "_index" : "shakespeare",
        "_id" : "3195",
        "_score" : null,
        "_source" : {
          "type" : "line",
          "line_id" : 3196,
          "play_name" : "Henry IV",
          "speech_number" : 8,
          "line_number" : "5.5.37",
          "speaker" : "KING HENRY IV",
          "text_entry" : "Towards York shall bend you with your dearest speed,"
        },
        "sort" : [
          3196
        ]
      }
    ]
  }
}
```

这`sort` 参数是一个数组，因此您可以按优先级指定多个字段值。

如果您有两个字段，则具有相同的值`line_id`，OpenSearch使用`speech_number`，这是排序的第二个选项：

```json
GET shakespeare/_search
{
  "query": {
    "term": {
      "play_name": {
        "value": "Henry IV"
      }
    }
  },
  "sort": [
    {
      "line_id": {
        "order": "desc"
      }
    },
    {
      "speech_number": {
        "order": "desc"
      }
    }
  ]
}
```

您可以继续按任意数量的字段值进行排序，以按正确的顺序获取结果。它不必是数值值＆mdash;您也可以按日期或时间戳字段进行排序：

```json
"sort": [
    {
      "date": {
        "order": "desc"
      }
    }
  ]
```

分析的文本字段不能用于对文档进行排序，因为倒置索引仅包含单个令牌术语而不是整个字符串。因此，您不能对`play_name`， 例如。

要绕过此限制，您可以使用映射的文本字段的原始版本作为关键字类型。在以下示例中，`play_name.keyword` 未分析，您有完整原始版本的副本以进行排序：

```json
GET shakespeare/_search
{
  "query": {
    "term": {
      "play_name": {
        "value": "Henry IV"
      }
    }
  },
  "sort": [
    {
      "play_name.keyword": {
        "order": "desc"
      }
    }
  ]
}
```

结果由`play_name` 字段按字母顺序排列。

使用`sort` 与[`search_after` 范围]({{site.url}}{{site.baseurl}}/opensearch/search/paginate#the-search_after-parameter) 为了更有效的滚动。
结果始于您在`search_after` 大批。

确保您在`search_after` 数组如`sort` 数组，也以相同的方式订购。
在这种情况下，您要求从以后的文档开始`line_id = 3202` 和`speech_number = 8`：

```json
GET shakespeare/_search
{
  "query": {
    "term": {
      "play_name": {
        "value": "Henry IV"
      }
    }
  },
  "sort": [
    {
      "line_id": {
        "order": "desc"
      }
    },
    {
      "speech_number": {
        "order": "desc"
      }
    }
  ],
  "search_after": [
    "3202",
    "8"
  ]
}
```

## 排序模式

排序模式适用于按数组或多值字段进行排序。它指定为对文档进行排序的数组值。对于包含数字数组的数字字段，您可以按`avg`，`sum`， 或者`median` 模式。要按最小值或最大值进行排序，请使用`min` 或者`max` 用于数字和字符串数据类型的模式。

默认模式为`min` 用于上升顺序和`max` 用于降序排序订单。

以下示例说明了使用排序模式通过数组字段进行排序。

考虑一个持有学生成绩的索引。将两个文档索引到索引：

```json
PUT students/_doc/1
{
   "name": "John Doe",
   "grades": [70, 90]
}

PUT students/_doc/2
{
   "name": "Mary Major",
   "grades": [80, 100]
}
```

使用最高年级的学生对所有学生进行分类`avg` 模式：

```
GET students/_search
{
   "query" : {
      "match_all": {}
   },
   "sort" : [
      {"grades" : {"order" : "desc", "mode" : "avg"}}
   ]
}
```

回答包含由学生排序的学生`grades` 按顺序下降：

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "students",
        "_id" : "2",
        "_score" : null,
        "_source" : {
          "name" : "Mary Major",
          "grades" : [
            80,
            100
          ]
        },
        "sort" : [
          90
        ]
      },
      {
        "_index" : "students",
        "_id" : "1",
        "_score" : null,
        "_source" : {
          "name" : "John Doe",
          "grades" : [
            70,
            90
          ]
        },
        "sort" : [
          80
        ]
      }
    ]
  }
}
```

## 排序嵌套对象

排序时[嵌套]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/nested) 对象，提供`path` 参数指定要排序字段的路径。

例如，在索引中`students`，映射变量`first_sem` 作为`nested`：

```json
PUT students
{
  "mappings" : {
    "properties": {
      "first_sem": { 
        "type" : "nested"
      }
    }
  }
}
```

索引两个带有嵌套字段的文档：

```json
PUT students/_doc/1
{
   "name": "John Doe",
   "first_sem" : {
     "grades": [70, 90]
   }
}

PUT students/_doc/2
{
  "name": "Mary Major",
  "first_sem": {
    "grades": [80, 100]
  }
}
```

按平均成绩排序时，提供通往嵌套场的路径：

```json
GET students/_search
{
 "query" : {
    "match_all": {}
 },
 "sort" : [
    {"first_sem.grades": {
      "order" : "desc", 
      "mode" : "avg",
      "nested": {
        "path": "first_sem"
     }
    }
    }
 ]
}
```

## 处理缺失值

这`missing` 参数指定缺少值的处理。建造-在有效值中是`_last` （列出最后一个具有丢失值的文档）和`_first` （首先列出具有缺失值的文档）。默认值是`_last`。您还可以指定一个自定义值，用于丢失文档作为排序值。

例如，您可以用`average` 字段和另一个文档没有`average` 场地：

```json
PUT students/_doc/1
{
   "name": "John Doe",
   "average": 80
}

PUT students/_doc/2
{
   "name": "Mary Major"
}
```

对文档进行排序，首先订购文档以缺少字段：

```json
GET students/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "average": {
        "order": "desc",
        "missing": "_first"
      }
    }
  ]
}
```

响应列表文档2首先：

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "students",
        "_id" : "2",
        "_score" : null,
        "_source" : {
          "name" : "Mary Major"
        },
        "sort" : [
          9223372036854775807
        ]
      },
      {
        "_index" : "students",
        "_id" : "1",
        "_score" : null,
        "_source" : {
          "name" : "John Doe",
          "average" : 80
        },
        "sort" : [
          80
        ]
      }
    ]
  }
}
```

## 忽略未封闭的字段

如果未映射字段，则默认情况下，该字段对此字段进行分类的搜索请求。为了避免这种情况，您可以使用`unmapped_type` 参数，该参数发信号以忽略字段。例如，如果您设置`unmapped_type` 到`long`，该场被视为被映射为类型`long`。此外，索引中的所有文档都有一个`unmapped_type` 字段被视为在这个领域没有价值，因此不会对其进行分类。

例如，考虑两个索引。索引包含一个的文档`average` 第一个索引中的字段：

```json
PUT students/_doc/1
{
   "name": "John Doe",
   "average": 80
}
```

索引不包含一个文档`average` 第二个索引中的字段：

```json
PUT students_no_map/_doc/2
{
   "name": "Mary Major"
}
```

在索引中搜索所有文档，并通过`average` 场地：

```json
GET students*/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "average": {
        "order": "desc"
      }
    }
  ]
}
```

默认情况下，第二个索引会产生错误，因为`average` 字段没有映射：

```json
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 1,
    "failures" : [
      {
        "shard" : 0,
        "index" : "students_no_map",
        "node" : "cam9NWqVSV-jUIkQ3tRubw",
        "reason" : {
          "type" : "query_shard_exception",
          "reason" : "No mapping found for [average] in order to sort on",
          "index" : "students_no_map",
          "index_uuid" : "JgfRkypKSUSpyU-ZXr9kKA"
        }
      }
    ]
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "students",
        "_id" : "1",
        "_score" : null,
        "_source" : {
          "name" : "John Doe",
          "average" : 80
        },
        "sort" : [
          80
        ]
      }
    ]
  }
}
```

您可以指定`unmapped_type` 参数以便忽略未上限的字段：

```json
GET students*/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "average": {
        "order": "desc",
        "unmapped_type": "long"
      }
    }
  ]
}
```

响应包含两个文档：

```json
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "students",
        "_id" : "1",
        "_score" : null,
        "_source" : {
          "name" : "John Doe",
          "average" : 80
        },
        "sort" : [
          80
        ]
      },
      {
        "_index" : "students_no_map",
        "_id" : "2",
        "_score" : null,
        "_source" : {
          "name" : "Mary Major"
        },
        "sort" : [
          -9223372036854775808
        ]
      }
    ]
  }
}
```

## 跟踪分数

默认情况下，在字段上排序时未计算得分。您可以设置`track_scores` 到`true` 计算和跟踪得分：

```json
GET students/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "average": {
        "order": "desc"
      }
    }
  ],
  "track_scores": true
}
```

## 按地理距离分类

您可以通过`_geo_distance`。支持以下参数。

范围| 描述
:--- | :---
dange_type| 指定计算距离的方法。有效值是`arc` 和`plane`。这`plane` 方法更快，但对于长距离或靠近电线杆的准确性较差。默认为`arc`。
模式| 指定如何使用多个地理点处理字段。默认情况下，当排序顺序上升的最短距离和最长的距离时，文档的排序是最短的。有效值是`min`，`max`，`median`， 和`avg`。
单元| 指定用于计算排序值的单元。默认为米（`m`）。
ignore_unmapped| 指定如何处理未映射的字段。放`ignore_unmapped` 到`true` 忽略未映射的字段。默认为`false` （在遇到未绘制的字段时产生错误）。

这`_geo_distance` 参数不支持`missing_values`。距离总是被认为是`infinity` 当文档不包含用于计算距离的字段时。
{: .note}

例如，索引两个具有地理点的文档：

```json
PUT testindex1/_doc/1
{
  "point": [74.00, 40.71] 
}

PUT testindex1/_doc/2
{
  "point": [73.77, -69.63] 
}
```

搜索所有文档，然后按距提供点的距离进行排序：

```json
GET testindex1/_search
{
  "sort": [
    {
      "_geo_distance": {
        "point": [59, -54],
        "order": "asc",
        "unit": "km",
        "distance_type": "arc",
        "mode": "min",
        "ignore_unmapped": true
      }
    }
  ],
  "query": {
    "match_all": {}
  }
}
```

响应包含分类的文档：

```json
{
  "took" : 864,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "testindex1",
        "_id" : "2",
        "_score" : null,
        "_source" : {
          "point" : [
            73.77,
            -69.63
          ]
        },
        "sort" : [
          1891.2667493895767
        ]
      },
      {
        "_index" : "testindex1",
        "_id" : "1",
        "_score" : null,
        "_source" : {
          "point" : [
            74.0,
            40.71
          ]
        },
        "sort" : [
          10628.402240213345
        ]
      }
    ]
  }
}
```

您可以提供以Geopoint字段类型支持的任何格式的坐标。有关所有格式的描述，请参阅[地理点字段类型文档]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/geo-point)。
{: .note}

将多个地理点传递给`_geo_distance`，使用一个数组：

```json
GET testindex1/_search
{
  "sort": [
    {
      "_geo_distance": {
        "point": [[59, -54], [60, -53]],
        "order": "asc",
        "unit": "km",
        "distance_type": "arc",
        "mode": "min",
        "ignore_unmapped": true
      }
    }
  ],
  "query": {
    "match_all": {}
  }
}
```

对于每个文档，分类距离计算为最小，最大或平均值（按照`mode`）与搜索中提供的所有点与文档中所有点的距离的距离。

## 性能考虑

排序的字段值将加载到内存中进行分类。因此，对于最少的开销，我们建议映射[数字类型]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/numeric) 对于最小的可接受类型，例如`short`，`integer`， 和`float`。[字符串类型]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/string) 不应对分类的字段进行分析或令牌化

