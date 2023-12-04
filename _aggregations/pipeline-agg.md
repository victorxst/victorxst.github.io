---
layout: default
title: 管道聚合
nav_order: 5
has_children: false
redirect_from:
  - /opensearch/pipeline-agg/
  - /query-dsl/aggregations/pipeline-agg/
---

# 管道聚合

使用管道聚合，您可以通过将一个聚合的结果作为输入来链接聚合，以获得更细微的输出。

您可以使用管道聚合来计算复杂的统计和数学措施，例如衍生物，移动平均值，累积总和等。

## 管道聚合语法

管道聚合使用`buckets_path` 访问其他聚合结果的属性。
这`buckets_path` 属性具有特定的语法：

```
buckets_path = <AGG_NAME>[<AGG_SEPARATOR>,<AGG_NAME>]*[<METRIC_SEPARATOR>, <METRIC>];
```

在哪里：

- `AGG_NAME` 是聚合的名称。
- `AGG_SEPARATOR` 分开聚合。它表示为`>`。
- `METRIC_SEPARATOR` 将聚合与其指标分开。它表示为`.`。
- `METRIC` 是度量的名称，以防万一-价值度量聚合。

例如，`my_sum.sum` 选择`sum` 聚集的指标称为`my_sum`。`popular_tags>my_sum.sum` 巢`my_sum.sum` 进入`popular_tags` 聚合。

您还可以指定以下其他参数：

- `gap_policy`： 真实的-世界数据可以包含空白值或零值。您可以指定与此类数据处理此类数据的策略`gap_policy` 财产。您可以设置`gap_policy` 财产为`skip` 跳过丢失的数据并从下一个可用值继续`insert_zeros` 用零替换缺失值并继续运行。
- `format`：输出值的格式类型。例如，`yyyy-MM-dd` 对于日期值。

## 快速示例

总结所有由`sum_total_memory` 聚合：

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "number_of_bytes": {
      "histogram": {
        "field": "bytes",
        "interval": 10000
      },
      "aggs": {
        "sum_total_memory": {
          "sum": {
            "field": "phpmemory"
          }
        }
      }
    },
    "sum_copies": {
      "sum_bucket": {
        "buckets_path": "number_of_bytes>sum_total_memory"
      }
    }
  }
}
```

#### 示例响应

```json
...
"aggregations" : {
  "number_of_bytes" : {
    "buckets" : [
      {
        "key" : 0.0,
        "doc_count" : 13372,
        "sum_total_memory" : {
          "value" : 9.12664E7
        }
      },
      {
        "key" : 10000.0,
        "doc_count" : 702,
        "sum_total_memory" : {
          "value" : 0.0
        }
      }
    ]
  },
  "sum_copies" : {
    "value" : 9.12664E7
  }
 }
}
```

## 管道聚合的类型

管道聚合有两种类型：

### 兄弟姐妹聚集

兄弟姐妹聚集取出嵌套聚合的输出，并在与嵌套水桶相同的水平上产生新的存储桶或新聚合。

兄弟姐妹的聚集必须是多型-存储键聚集（具有一定字段的多个分组值），并且度量必须是数字值。

`min_bucket`，`max_bucket`，`sum_bucket`， 和`avg_bucket` 是常见的兄弟姐妹聚集。

### 父集合

父集合将外部聚合的输出输出，并在与现有存储桶的相同水平上产生新的存储桶或新聚合。

家长聚合必须具有`min_doc_count` 设置为0（默认`histogram` 聚合）和指定的度量必须是数字值。如果`min_doc_count` 大于`0`，省略了一些水桶，这可能会导致结果不正确。

`derivatives` 和`cumulative_sum` 是常见的母体聚合。

## avg_bucket，sum_bucket，min_bucket，max_bucket

这`avg_bucket`，`sum_bucket`，`min_bucket`， 和`max_bucket` 聚合是计算先前聚合的每个桶中度量的平均值，总和，最小值和最大值的同级聚合。

以下示例使用一个创建日期直方图-月间隔。这`sum` 子-聚合计算每个月的所有字节的总和。最后，`avg_bucket` 聚合使用此总和来计算每月的平均字节数：

```json
POST opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "visits_per_month": {
      "date_histogram": {
        "field": "@timestamp",
        "interval": "month"
      },
      "aggs": {
        "sum_of_bytes": {
          "sum": {
            "field": "bytes"
          }
        }
      }
    },
    "avg_monthly_bytes": {
      "avg_bucket": {
        "buckets_path": "visits_per_month>sum_of_bytes"
      }
    }
  }
}
```

#### 示例响应

```json
...
"aggregations" : {
  "visits_per_month" : {
    "buckets" : [
      {
        "key_as_string" : "2020-10-01T00:00:00.000Z",
        "key" : 1601510400000,
        "doc_count" : 1635,
        "sum_of_bytes" : {
          "value" : 9400200.0
        }
      },
      {
        "key_as_string" : "2020-11-01T00:00:00.000Z",
        "key" : 1604188800000,
        "doc_count" : 6844,
        "sum_of_bytes" : {
          "value" : 3.8880434E7
        }
      },
      {
        "key_as_string" : "2020-12-01T00:00:00.000Z",
        "key" : 1606780800000,
        "doc_count" : 5595,
        "sum_of_bytes" : {
          "value" : 3.1445055E7
        }
      }
    ]
  },
  "avg_monthly_bytes" : {
    "value" : 2.6575229666666668E7
  }
 }
}
```

以类似的方式，您可以计算`sum_bucket`，`min_bucket`， 和`max_bucket` 每月字节的值。

## stats_bucket，extended_stats_bucket

这`stats_bucket` 聚合是一个兄弟姐妹聚合，返回各种统计数据（`count`，`min`，`max`，`avg`， 和`sum`）用于先前聚合的存储桶。

以下示例返回了由`sum_of_bytes` 聚集嵌套到`visits_per_month` 聚合：

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "visits_per_month": {
      "date_histogram": {
        "field": "@timestamp",
        "interval": "month"
      },
      "aggs": {
        "sum_of_bytes": {
          "sum": {
            "field": "bytes"
          }
        }
      }
    },
    "stats_monthly_bytes": {
      "stats_bucket": {
        "buckets_path": "visits_per_month>sum_of_bytes"
      }
    }
  }
}
```

#### 示例响应

```json
...
"stats_monthly_bytes" : {
  "count" : 3,
  "min" : 9400200.0,
  "max" : 3.8880434E7,
  "avg" : 2.6575229666666668E7,
  "sum" : 7.9725689E7
  }
 }
}
```

这`extended_stats` 聚合是一个扩展版本的`stats` 聚合。除了包括基本统计数据外，`extended_stats` 还提供诸如`sum_of_squares`，`variance`， 和`std_deviation`。

#### 示例响应

```json
"stats_monthly_visits" : {
  "count" : 3,
  "min" : 9400200.0,
  "max" : 3.8880434E7,
  "avg" : 2.6575229666666668E7,
  "sum" : 7.9725689E7,
  "sum_of_squares" : 2.588843392021381E15,
  "variance" : 1.5670496550438025E14,
  "variance_population" : 1.5670496550438025E14,
  "variance_sampling" : 2.3505744825657038E14,
  "std_deviation" : 1.251818539183616E7,
  "std_deviation_population" : 1.251818539183616E7,
  "std_deviation_sampling" : 1.5331583357780447E7,
  "std_deviation_bounds" : {
    "upper" : 5.161160045033899E7,
    "lower" : 1538858.8829943463,
    "upper_population" : 5.161160045033899E7,
    "lower_population" : 1538858.8829943463,
    "upper_sampling" : 5.723839638222756E7,
    "lower_sampling" : -4087937.0488942266
   }
  }
 }
}
```

## buck_script，bucket_selector

这`bucket_script` 聚合是父派的，该脚本执行脚本以执行每个-先前聚集的水桶计算。确保指标是数字类型，并且返回的值也是数字。

使用`script` 参数添加您的脚本。该脚本可以在文件，文件中或索引中是内联。要启用内联脚本，请将以下行添加到您的`opensearch.yml` 文件中的文件`config` 文件夹：

```yaml
script.inline: on
```

这`buckets_path` 属性由多个条目组成。每个条目都是钥匙和值。关键是您可以在脚本中使用的值的名称。

基本语法是：

```json
{
  "bucket_script": {
    "buckets_path": {
    "my_var1": "the_sum",
    "my_var2": "the_value_count"
  },
 "script": "params.my_var1 / params.my_var2"
  }
}
```

以下示例使用`sum` 日期直方图生成的存储桶上的聚合。从最终的存储桶值中，在ZIP扩展的背景下以10,000个字节的间隔计算RAM的百分比：

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "histogram": {
        "field": "bytes",
        "interval": "10000"
      },
      "aggs": {
        "total_ram": {
          "sum": {
            "field": "machine.ram"
          }
        },
        "ext-type": {
          "filter": {
            "term": {
              "extension.keyword": "zip"
            }
          },
          "aggs": {
            "total_ram": {
              "sum": {
                "field": "machine.ram"
              }
            }
          }
        },
        "ram-percentage": {
          "bucket_script": {
            "buckets_path": {
              "machineRam": "ext-type>total_ram",
              "totalRam": "total_ram"
            },
            "script": "params.machineRam / params.totalRam"
          }
        }
      }
    }
  }
}
```

#### 示例响应

```json
"aggregations" : {
  "sales_per_month" : {
    "buckets" : [
      {
        "key" : 0.0,
        "doc_count" : 13372,
        "os-type" : {
          "doc_count" : 1558,
          "total_ram" : {
            "value" : 2.0090783268864E13
          }
        },
        "total_ram" : {
          "value" : 1.7214228922368E14
        },
        "ram-percentage" : {
          "value" : 0.11671032934131736
        }
      },
      {
        "key" : 10000.0,
        "doc_count" : 702,
        "os-type" : {
          "doc_count" : 116,
          "total_ram" : {
            "value" : 1.622423896064E12
          }
        },
        "total_ram" : {
          "value" : 9.015136354304E12
        },
        "ram-percentage" : {
          "value" : 0.17996665078608862
        }
      }
    ]
  }
 }
}
```

计算RAM百分比并在每个存储桶的末尾附加。

这`bucket_selector` 聚合是一个脚本-基于聚合，选择由A返回的存储桶`histogram` （或者`date_histogram`）聚合。在您不希望根据您提供的条件中某些输出中的某些存储桶的情况下使用它。

这`bucket_selector` 汇总执行脚本以确定一个水桶是否停留在父级中-铲斗聚集。

基本语法是：

```json
{
  "bucket_selector": {
    "buckets_path": {
    "my_var1": "the_sum",
    "my_var2": "the_value_count"
  },
  "script": "params.my_var1 / params.my_var2"
  }
}
```

以下示例计算字节的总和，然后评估此总和是否大于20,000。如果为true，则将存储桶保留在存储桶列表中。否则，它将从最终输出中删除。

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "bytes_per_month": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "month"
      },
      "aggs": {
        "total_bytes": {
          "sum": {
            "field": "bytes"
          }
        },
        "bytes_bucket_filter": {
          "bucket_selector": {
            "buckets_path": {
              "totalBytes": "total_bytes"
            },
            "script": "params.totalBytes > 20000"
          }
        }
      }
    }
  }
}
```

#### 示例响应

```json
"aggregations" : {
  "bytes_per_month" : {
    "buckets" : [
      {
        "key_as_string" : "2020-10-01T00:00:00.000Z",
        "key" : 1601510400000,
        "doc_count" : 1635,
        "total_bytes" : {
          "value" : 9400200.0
        }
      },
      {
        "key_as_string" : "2020-11-01T00:00:00.000Z",
        "key" : 1604188800000,
        "doc_count" : 6844,
        "total_bytes" : {
          "value" : 3.8880434E7
        }
      },
      {
        "key_as_string" : "2020-12-01T00:00:00.000Z",
        "key" : 1606780800000,
        "doc_count" : 5595,
        "total_bytes" : {
          "value" : 3.1445055E7
        }
      }
    ]
  }
 }
}
```

## bucket_sort

这`bucket_sort` 聚合是父母聚集，与先前聚集的桶分类。

您可以将多个排序字段与相应的排序顺序一起指定。此外，您可以根据其钥匙，计数或其子来对每个存储桶进行排序-聚合。您也可以通过设置来截断水桶`from` 和`size` 参数。

句法

```json
{
  "bucket_sort": {
    "sort": [
    {"sort_field_1": {"order": "asc"}},
    {"sort_field_2": {"order": "desc"}},
    "sort_field_3"
    ],
 "from":1,
 "size":3
 }
}
```

以下示例分类了`date_histogram` 基于计算的聚合`total_sum` 值。我们按降序对存储库进行排序，以便首先返回具有最高字节数的存储桶。

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "month"
      },
      "aggs": {
        "total_bytes": {
          "sum": {
            "field": "bytes"
          }
        },
        "bytes_bucket_sort": {
          "bucket_sort": {
            "sort": [
              { "total_bytes": { "order": "desc" } }
            ],
            "size": 3                                
          }
        }
      }
    }
  }
}
```

#### 示例响应

```json
"aggregations" : {
  "sales_per_month" : {
    "buckets" : [
      {
        "key_as_string" : "2020-11-01T00:00:00.000Z",
        "key" : 1604188800000,
        "doc_count" : 6844,
        "total_bytes" : {
          "value" : 3.8880434E7
        }
      },
      {
        "key_as_string" : "2020-12-01T00:00:00.000Z",
        "key" : 1606780800000,
        "doc_count" : 5595,
        "total_bytes" : {
          "value" : 3.1445055E7
        }
      },
      {
        "key_as_string" : "2020-10-01T00:00:00.000Z",
        "key" : 1601510400000,
        "doc_count" : 1635,
        "total_bytes" : {
          "value" : 9400200.0
        }
      }
    ]
  }
 }
}
```

您也可以使用此聚合来截断所得的存储桶而无需排序。为此，只需使用`from` 和/或`size` 没有参数`sort`。

## 累积_sum

这`cumulative_sum` 聚集是一个母体聚集，可以计算先前聚集的每个桶的累积总和。

累积总和是给定序列的部分总和。例如，序列的累积总和`{a,b,c,…}` 是`a`，`a+b`，`a+b+c`， 等等。您可以使用累积总和来可视化磁场随时间的变化率。

以下示例每月计算字节的累积数：

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "month"
      },
      "aggs": {
        "no-of-bytes": {
          "sum": {
            "field": "bytes"
          }
        },
        "cumulative_bytes": {
          "cumulative_sum": {
            "buckets_path": "no-of-bytes"
          }
        }
      }
    }
  }
}
```

#### 示例响应

```json
...
"aggregations" : {
  "sales_per_month" : {
    "buckets" : [
      {
        "key_as_string" : "2020-10-01T00:00:00.000Z",
        "key" : 1601510400000,
        "doc_count" : 1635,
        "no-of-bytes" : {
          "value" : 9400200.0
        },
        "cumulative_bytes" : {
          "value" : 9400200.0
        }
      },
      {
        "key_as_string" : "2020-11-01T00:00:00.000Z",
        "key" : 1604188800000,
        "doc_count" : 6844,
        "no-of-bytes" : {
          "value" : 3.8880434E7
        },
        "cumulative_bytes" : {
          "value" : 4.8280634E7
        }
      },
      {
        "key_as_string" : "2020-12-01T00:00:00.000Z",
        "key" : 1606780800000,
        "doc_count" : 5595,
        "no-of-bytes" : {
          "value" : 3.1445055E7
        },
        "cumulative_bytes" : {
          "value" : 7.9725689E7
        }
      }
    ]
  }
 }
}
```

## 衍生物

这`derivative` 聚合是一个母体聚合，该聚合计算先前聚集的每个桶的一阶和第二阶导数。

在数学中，功能的衍生物衡量其对变化的敏感性。换句话说，衍生物评估了某些变量的某些功能的变化率。要了解有关衍生物的更多信息，请参阅[维基百科](https://en.wikipedia.org/wiki/Derivative)。

您可以使用导数来计算数字值的变化速率与其先前的时间段相比。

一阶导数表明度量是否正在增加或减少，以及它的增加或减少。

以下示例计算每月字节总和的一阶导数。一阶导数是本月和上个月的字节数之间的差异：

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "month"
      },
      "aggs": {
        "number_of_bytes": {
          "sum": {
            "field": "bytes"
          }
        },
        "bytes_deriv": {
          "derivative": {
            "buckets_path": "number_of_bytes"
          }
        }
      }
    }
  }
}
```

#### 示例响应

```json
...
"aggregations" : {
  "sales_per_month" : {
    "buckets" : [
      {
        "key_as_string" : "2020-10-01T00:00:00.000Z",
        "key" : 1601510400000,
        "doc_count" : 1635,
        "number_of_bytes" : {
          "value" : 9400200.0
        }
      },
      {
        "key_as_string" : "2020-11-01T00:00:00.000Z",
        "key" : 1604188800000,
        "doc_count" : 6844,
        "number_of_bytes" : {
          "value" : 3.8880434E7
        },
        "bytes_deriv" : {
          "value" : 2.9480234E7
        }
      },
      {
        "key_as_string" : "2020-12-01T00:00:00.000Z",
        "key" : 1606780800000,
        "doc_count" : 5595,
        "number_of_bytes" : {
          "value" : 3.1445055E7
        },
        "bytes_deriv" : {
          "value" : -7435379.0
        }
      }
    ]
  }
}
```

第二阶导数是衍生物的双衍生物或衍生物。
它表明数量的变化率本身是如何变化的。这是相邻桶的一阶导数之间的区别。

为了计算第二阶导数，将一个衍生物聚集链到另一个衍生物：

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "month"
      },
      "aggs": {
        "number_of_bytes": {
          "sum": {
            "field": "bytes"
          }
        },
        "bytes_deriv": {
          "derivative": {
            "buckets_path": "number_of_bytes"
          }
        },
        "bytes_2nd_deriv": {
          "derivative": {
            "buckets_path": "bytes_deriv"
          }
        }
      }
    }
  }
}
```

#### 示例响应

```json
...
"aggregations" : {
  "sales_per_month" : {
    "buckets" : [
      {
        "key_as_string" : "2020-10-01T00:00:00.000Z",
        "key" : 1601510400000,
        "doc_count" : 1635,
        "number_of_bytes" : {
          "value" : 9400200.0
        }
      },
      {
        "key_as_string" : "2020-11-01T00:00:00.000Z",
        "key" : 1604188800000,
        "doc_count" : 6844,
        "number_of_bytes" : {
          "value" : 3.8880434E7
        },
        "bytes_deriv" : {
          "value" : 2.9480234E7
        }
      },
      {
        "key_as_string" : "2020-12-01T00:00:00.000Z",
        "key" : 1606780800000,
        "doc_count" : 5595,
        "number_of_bytes" : {
          "value" : 3.1445055E7
        },
        "bytes_deriv" : {
          "value" : -7435379.0
        },
        "bytes_2nd_deriv" : {
          "value" : -3.6915613E7
        }
      }
    ]
  }
}
```

第一个水桶没有一阶导数，因为派生需要至少两个点进行比较。第一和第二个存储桶没有第二阶导数，因为第二阶派生至少需要从一阶导数中获得两个数据点。

一阶导数"2020-11-01" 铲斗为2.9480234e7，"2020-12-01" 水桶是-7435379.因此，“ 2020-12-01英寸的存储桶是-3.6915613E7（-7435379-2.9480234E7）。

从理论上讲，您可以继续链接派生的聚合以计算第三，第四，甚至更高的-订单衍生物。但是，这对大多数数据集提供了几乎没有价值。

## Move_avg

A`moving_avg` 聚合是一个母体聚集，可计算移动平均值。

这`moving_avg` 聚合发现数据集的不同窗口（子集）的一系列平均值。窗口的大小表示每次迭代窗口涵盖的数据点的数量（由`window` 属性并默认设置为5）。在每次迭代中，该算法计算出适合窗口的所有数据点的平均值，然后通过排除上一个窗口的第一个成员以及在下一个窗口中包括第一个成员来向前滑动。

例如，给定数据`[1, 5, 8, 23, 34, 28, 7, 23, 20, 19]`，您可以计算一个简单的移动平均值，窗口的大小为5，如下所示：

```
(1 + 5 + 8 + 23 + 34) / 5 = 14.2
(5 + 8 + 23 + 34+ 28) / 5 = 19.6
(8 + 23 + 34 + 28 + 7) / 5 = 20
so on...
```

有关更多信息，请参阅[维基百科](https://en.wikipedia.org/wiki/Moving_average)。

您可以使用`moving_avg` 聚集以使要么平滑-术语波动或突出显示更长-您的时间趋势或周期-系列数据。

指定一个小窗口大小（例如，`window`：10）密切遵循数据以使小小-比例波动。
或者，指定较大的窗口大小（例如，`window`：100）落后于实际数据的大量落后于实际数据，以使所有更高-频率波动或随机噪声，使较低的频率趋势更加明显。

以下示例筑巢`moving_avg` 聚集成一个`date_histogram` 聚合：

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "my_date_histogram": {                                
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "month"
      },
      "aggs": {
        "sum_of_bytes": {
          "sum": { "field": "bytes" }                 
        },
        "moving_avg_of_sum_of_bytes": {
          "moving_avg": { "buckets_path": "sum_of_bytes" }
        }
      }
    }
  }
}
```

#### 示例响应

```json  
...
"aggregations" : {
  "my_date_histogram" : {
    "buckets" : [
      {
        "key_as_string" : "2020-10-01T00:00:00.000Z",
        "key" : 1601510400000,
        "doc_count" : 1635,
        "sum_of_bytes" : {
          "value" : 9400200.0
        }
      },
      {
        "key_as_string" : "2020-11-01T00:00:00.000Z",
        "key" : 1604188800000,
        "doc_count" : 6844,
        "sum_of_bytes" : {
          "value" : 3.8880434E7
        },
        "moving_avg_of_sum_of_bytes" : {
          "value" : 9400200.0
        }
      },
      {
        "key_as_string" : "2020-12-01T00:00:00.000Z",
        "key" : 1606780800000,
        "doc_count" : 5595,
        "sum_of_bytes" : {
          "value" : 3.1445055E7
        },
        "moving_avg_of_sum_of_bytes" : {
          "value" : 2.4140317E7
        }
      }
    ]
  }
}
```

您也可以使用`moving_avg` 聚集以预测未来的存储桶。
要预测存储桶，请添加`predict` 属性并将其设置为您想要看到的预测数。

以下示例将五个预测添加到上述查询：

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "my_date_histogram": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "month"
      },
      "aggs": {
        "sum_of_bytes": {
          "sum": {
            "field": "bytes"
          }
        },
        "moving_avg_of_sum_of_bytes": {
          "moving_avg": {
            "buckets_path": "sum_of_bytes",
            "predict": 5
          }
        }
      }
    }
  }
}
```

#### 示例响应

```json
"aggregations" : {
  "my_date_histogram" : {
    "buckets" : [
      {
        "key_as_string" : "2020-10-01T00:00:00.000Z",
        "key" : 1601510400000,
        "doc_count" : 1635,
        "sum_of_bytes" : {
          "value" : 9400200.0
        }
      },
      {
        "key_as_string" : "2020-11-01T00:00:00.000Z",
        "key" : 1604188800000,
        "doc_count" : 6844,
        "sum_of_bytes" : {
          "value" : 3.8880434E7
        },
        "moving_avg_of_sum_of_bytes" : {
          "value" : 9400200.0
        }
      },
      {
        "key_as_string" : "2020-12-01T00:00:00.000Z",
        "key" : 1606780800000,
        "doc_count" : 5595,
        "sum_of_bytes" : {
          "value" : 3.1445055E7
        },
        "moving_avg_of_sum_of_bytes" : {
          "value" : 2.4140317E7
        }
      },
      {
        "key_as_string" : "2021-01-01T00:00:00.000Z",
        "key" : 1609459200000,
        "doc_count" : 0,
        "moving_avg_of_sum_of_bytes" : {
          "value" : 2.6575229666666668E7
        }
      },
      {
        "key_as_string" : "2021-02-01T00:00:00.000Z",
        "key" : 1612137600000,
        "doc_count" : 0,
        "moving_avg_of_sum_of_bytes" : {
          "value" : 2.6575229666666668E7
        }
      },
      {
        "key_as_string" : "2021-03-01T00:00:00.000Z",
        "key" : 1614556800000,
        "doc_count" : 0,
        "moving_avg_of_sum_of_bytes" : {
          "value" : 2.6575229666666668E7
        }
      },
      {
        "key_as_string" : "2021-04-01T00:00:00.000Z",
        "key" : 1617235200000,
        "doc_count" : 0,
        "moving_avg_of_sum_of_bytes" : {
          "value" : 2.6575229666666668E7
        }
      },
      {
        "key_as_string" : "2021-05-01T00:00:00.000Z",
        "key" : 1619827200000,
        "doc_count" : 0,
        "moving_avg_of_sum_of_bytes" : {
          "value" : 2.6575229666666668E7
        }
      }
    ]
  }
}
```

这`moving_avg` 聚合支持五个模型 - `simple`，`linear`，`exponentially weighted`，`holt-linear`， 和`holt-winters`。这些模型在窗口的值加权方面有所不同。随着数据点变为"older" （即，窗户滑离它们），它们的加权可能会有所不同。您可以通过设置`model` 财产。这`model` 属性拥有模型的名称，`settings` 对象，您可以用来提供模型属性。有关这些模型的更多信息，请参阅[维基百科](https://en.wikipedia.org/wiki/Moving_average)。

A`simple` 模型首先计算窗口中所有数据点的总和，然后将该总和除以窗口的大小。换句话说，一个`simple` 模型计算数据集中每个窗口的简单算术平均值。

以下示例使用一个简单的模型，窗口大小为30：

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "my_date_histogram": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "month"
      },
      "aggs": {
        "sum_of_bytes": {
          "sum": {
            "field": "bytes"
          }
        },
        "moving_avg_of_sum_of_bytes": {
          "moving_avg": {
            "buckets_path": "sum_of_bytes",
            "window": 30,
            "model": "simple"
          }
        }
      }
    }
  }
}
```


#### 示例响应

```json
...
"aggregations" : {
  "my_date_histogram" : {
    "buckets" : [
      {
        "key_as_string" : "2020-10-01T00:00:00.000Z",
        "key" : 1601510400000,
        "doc_count" : 1635,
        "sum_of_bytes" : {
          "value" : 9400200.0
        }
      },
      {
        "key_as_string" : "2020-11-01T00:00:00.000Z",
        "key" : 1604188800000,
        "doc_count" : 6844,
        "sum_of_bytes" : {
          "value" : 3.8880434E7
        },
        "moving_avg_of_sum_of_bytes" : {
          "value" : 9400200.0
        }
      },
      {
        "key_as_string" : "2020-12-01T00:00:00.000Z",
        "key" : 1606780800000,
        "doc_count" : 5595,
        "sum_of_bytes" : {
          "value" : 3.1445055E7
        },
        "moving_avg_of_sum_of_bytes" : {
          "value" : 2.4140317E7
        }
      }
    ]
  }
}
```

以下示例使用`holt` 模型。您可以设置重要性衰减的速度`alpha` 和`beta` 环境。默认值的默认值`alpha` 是0.3和`beta` 是0.1。您可以指定0之间的任何浮点值-1包含。

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "my_date_histogram": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "month"
      },
      "aggs": {
        "sum_of_bytes": {
          "sum": {
            "field": "bytes"
          }
        },
        "moving_avg_of_sum_of_bytes": {
          "moving_avg": {
            "buckets_path": "sum_of_bytes",
            "model": "holt",
            "settings": {
              "alpha": 0.6,
              "beta": 0.4
            }
          }
        }
      }
    }
  }
}
```

#### 示例响应

```json
...
"aggregations" : {
  "my_date_histogram" : {
    "buckets" : [
      {
        "key_as_string" : "2020-10-01T00:00:00.000Z",
        "key" : 1601510400000,
        "doc_count" : 1635,
        "sum_of_bytes" : {
          "value" : 9400200.0
        }
      },
      {
        "key_as_string" : "2020-11-01T00:00:00.000Z",
        "key" : 1604188800000,
        "doc_count" : 6844,
        "sum_of_bytes" : {
          "value" : 3.8880434E7
        },
        "moving_avg_of_sum_of_bytes" : {
          "value" : 9400200.0
        }
      },
      {
        "key_as_string" : "2020-12-01T00:00:00.000Z",
        "key" : 1606780800000,
        "doc_count" : 5595,
        "sum_of_bytes" : {
          "value" : 3.1445055E7
        },
        "moving_avg_of_sum_of_bytes" : {
          "value" : 2.70883404E7
        }
      }
    ]
  }
}
```



## serial_diff

这`serial_diff` 聚合是一个母体管道聚合，它计算以前聚合的存储桶的时间滞后之间的一系列值差异。

您可以使用`serial_diff` 集合查找时间段之间的数据更改，而不是找到整个值。

与`lag` 参数（正，非-零整数值），您可以从当前的桶中判断哪个以前的存储桶要减去。如果您不指定`lag` 参数，OpenSearch将其设置为1。

可以说，一个城市的人口随着时间的流逝而增长。如果您在一天的时间内使用串行差异聚合，则可以看到每日增长。例如，您可以计算总价每周平均变化的一系列差异。

```json
GET opensearch_dashboards_sample_data_logs/_search
{
   "size": 0,
   "aggs": {
      "my_date_histogram": {                  
         "date_histogram": {
            "field": "@timestamp",
            "calendar_interval": "month"
         },
         "aggs": {
            "the_sum": {
               "sum": {
                  "field": "bytes"     
               }
            },
            "thirtieth_difference": {
               "serial_diff": {                
                  "buckets_path": "the_sum",
                  "lag" : 30
               }
            }
         }
      }
   }
}
```

#### 示例响应

```json
...
"aggregations" : {
  "my_date_histogram" : {
    "buckets" : [
      {
        "key_as_string" : "2020-10-01T00:00:00.000Z",
        "key" : 1601510400000,
        "doc_count" : 1635,
        "the_sum" : {
          "value" : 9400200.0
        }
      },
      {
        "key_as_string" : "2020-11-01T00:00:00.000Z",
        "key" : 1604188800000,
        "doc_count" : 6844,
        "the_sum" : {
          "value" : 3.8880434E7
        }
      },
      {
        "key_as_string" : "2020-12-01T00:00:00.000Z",
        "key" : 1606780800000,
        "doc_count" : 5595,
        "the_sum" : {
          "value" : 3.1445055E7
        }
      }
    ]
  }
}
```

