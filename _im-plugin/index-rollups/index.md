---
layout: default
title: 索引汇总
nav_order: 35
has_children: true
redirect_from: 
  - /im-plugin/index-rollups/
---

# 索引汇总

时序数据会增加存储成本，使群集运行状况紧张，并随着时间的推移减慢聚合速度。索引汇总允许你通过将旧数据汇总到汇总索引中来定期降低数据粒度。

你可以选择你感兴趣的字段，然后使用索引汇总来创建新索引，其中仅将这些字段聚合到较粗的时间存储桶中。在相同的查询性能下，你可以存储数月或数年的历史数据，而成本只是其中的一小部分。

例如，假设你每 5 秒收集一次 CPU 消耗数据，并将其存储在热节点上。你可以汇总或压缩此数据，而不是将旧数据移动到只读热节点，只需每天的平均 CPU 消耗量或每周减少 10% 的间隔。

你可以通过三种方式使用索引汇总：

1. 将索引汇总 API 用于按需索引汇总作业，该作业对未主动引入的索引（如滚动索引）进行操作。例如，你可以执行索引汇总操作，将每隔 5 分钟收集的数据减少到每周平均值，以便进行趋势分析。
2. 使用 OpenSearch 控制面板 UI 创建按定义的计划运行的索引汇总作业。你还可以将其设置为在主动引入索引时汇总索引。例如，你可以将 Logstash 索引从 5 秒间隔持续汇总到 1 小时间隔。
3. 将索引汇总作业指定为 ISM 操作，以便完成索引管理。这允许你在特定事件（例如滚动更新、索引期限达到某个点、索引变为只读等）后汇总索引。你还可以按顺序运行滚动更新和索引汇总作业，其中滚动更新首先将当前索引移动到暖节点，然后索引汇总作业使用热节点上的最小化数据创建新索引。

## 创建索引汇总作业

要开始使用，请选择**索引管理** In OpenSearch Dashboards。选择并选择**汇总作业** **创建汇总作业**。

### 步骤 1：设置索引

1. 在该**职位名称和描述**部分中，指定索引汇总作业的唯一名称和可选描述。
2. 在该**指标**部分中，选择源索引和目标索引。源索引是要汇总的索引。源索引保持原样，索引汇总作业将创建一个称为目标索引的新索引。目标索引是保存索引汇总结果的位置。对于目标索引，可以键入新索引的名称，也可以选择现有索引。
5. 选择**下一个**

创建索引汇总作业后，无法更改索引选择。

### 第 2 步：定义聚合和指标

选择包含要汇总的聚合（术语和直方图）和指标（平均值、总和、最大值、最小值和值计数）的属性。请确保不要添加大量高度精细的属性，因为不会节省太多空间。

例如，考虑城市和这些城市内的人口统计数据数据集。你可以基于城市进行聚合，并将城市内的人口统计数据指定为指标。选择属性的顺序至关重要。城市后跟人口统计与人口后跟城市不同。

1. 在**时间聚合**该部分中，选择时间戳字段。在间隔类型之间**固定****日历**进行选择，并指定间隔和时区。索引汇总作业使用此信息为时间戳字段创建日期直方图。
2. （可选）为每个字段添加其他聚合。你可以为所有字段类型选择术语聚合，而仅为数值字段选择直方图聚合。
3. （可选）为每个字段添加其他指标。你可以在、、、**最小值****和****麦克斯**、**平均值**或**值计数**之间**都**进行选择。
4. 选择**下一个**。

### 第 3 步：指定计划

指定一个计划，以便在引入索引时汇总索引。默认情况下，索引汇总作业处于启用状态。

1. 指定数据是否连续。
3. 对于汇总执行频率，选择并指定**汇总间隔**和时间单位，或者**通过 cron 表达式定义**添加 cron 表达式以选择**按固定间隔定义**间隔。要了解如何定义 cron 表达式，请参见[Alerting]({{site.url}}{{site.baseurl}}/monitoring-plugins/alerting/cron/)。
4. 指定每个执行进程的页数。数字越大，执行速度越快，内存成本越高。
5. （可选）向汇总执行添加延迟。这是作业等待数据引入以适应任何处理时间的时间量。例如，如果将此值设置为 10 分钟，则在下午 2 点执行以汇总下午 1 点到下午 2 点的数据的索引汇总将从下午 2：10 开始。
6. 选择**下一个**。

### 第 4 步：查看和创建

查看你的配置，然后选择**创造**。

### 步骤 5：搜索目标索引

你可以使用标准 `_search` API 搜索目标索引。确保查询与目标索引的约束匹配。例如，如果你未在字段上设置术语聚合，则不会收到术语聚合的结果。如果未设置最大聚合数，则不会收到最大聚合数的结果。

你无法访问目标索引中数据的内部结构，因为插件会在后台自动重写查询以适应目标索引。这是为了确保你可以对源索引和目标索引使用相同的查询。

要查询目标索引，请设置为 `size` 0：

```json
GET target_index/_search
{
  "size": 0,
  "query": {
    "match_all": {}
  },
  "aggs": {
    "avg_cpu": {
      "avg": {
        "field": "cpu_usage"
      }
    }
  }
}
```

假设你每小时收集下午 1 点到晚上 9 点的汇总数据，以及以分钟为间隔收集晚上 7 点到晚上 11 点的实时数据。如果在同一查询中对这些数据执行聚合，则在晚上 7 点到晚上 9 点，你会看到汇总数据和实时数据重叠，因为它们在聚合中被计数两次。

## 示例演练

本演练使用 OpenSearch 控制面板示例电子商务数据。要添加该示例数据，请登录 OpenSearch 控制面板，选择**家**和**试试我们的示例数据**。对于**电子商务订单示例**，选择**添加数据**。

然后运行搜索：

```json
GET opensearch_dashboards_sample_data_ecommerce/_search
```

#### 响应示例

```json
{
  "took": 23,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 4675,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "opensearch_dashboards_sample_data_ecommerce",
        "_type": "_doc",
        "_id": "jlMlwXcBQVLeQPrkC_kQ",
        "_score": 1,
        "_source": {
          "category": [
            "Women's Clothing",
            "Women's Accessories"
          ],
          "currency": "EUR",
          "customer_first_name": "Selena",
          "customer_full_name": "Selena Mullins",
          "customer_gender": "FEMALE",
          "customer_id": 42,
          "customer_last_name": "Mullins",
          "customer_phone": "",
          "day_of_week": "Saturday",
          "day_of_week_i": 5,
          "email": "selena@mullins-family.zzz",
          "manufacturer": [
            "Tigress Enterprises"
          ],
          "order_date": "2021-02-27T03:56:10+00:00",
          "order_id": 581553,
          "products": [
            {
              "base_price": 24.99,
              "discount_percentage": 0,
              "quantity": 1,
              "manufacturer": "Tigress Enterprises",
              "tax_amount": 0,
              "product_id": 19240,
              "category": "Women's Clothing",
              "sku": "ZO0064500645",
              "taxless_price": 24.99,
              "unit_discount_amount": 0,
              "min_price": 12.99,
              "_id": "sold_product_581553_19240",
              "discount_amount": 0,
              "created_on": "2016-12-24T03:56:10+00:00",
              "product_name": "Blouse - port royal",
              "price": 24.99,
              "taxful_price": 24.99,
              "base_unit_price": 24.99
            },
            {
              "base_price": 10.99,
              "discount_percentage": 0,
              "quantity": 1,
              "manufacturer": "Tigress Enterprises",
              "tax_amount": 0,
              "product_id": 17221,
              "category": "Women's Accessories",
              "sku": "ZO0085200852",
              "taxless_price": 10.99,
              "unit_discount_amount": 0,
              "min_price": 5.06,
              "_id": "sold_product_581553_17221",
              "discount_amount": 0,
              "created_on": "2016-12-24T03:56:10+00:00",
              "product_name": "Snood - rose",
              "price": 10.99,
              "taxful_price": 10.99,
              "base_unit_price": 10.99
            }
          ],
          "sku": [
            "ZO0064500645",
            "ZO0085200852"
          ],
          "taxful_total_price": 35.98,
          "taxless_total_price": 35.98,
          "total_quantity": 2,
          "total_unique_products": 2,
          "type": "order",
          "user": "selena",
          "geoip": {
            "country_iso_code": "MA",
            "location": {
              "lon": -8,
              "lat": 31.6
            },
            "region_name": "Marrakech-Tensift-Al Haouz",
            "continent_name": "Africa",
            "city_name": "Marrakesh"
          },
          "event": {
            "dataset": "sample_ecommerce"
          }
        }
      }
    ]
  }
}
...
```

创建索引汇总作业。此示例选取 `order_date`、、 `customer_gender` `geoip.city_name` `geoip.region_name` 和 `day_of_week` 字段，并将它们汇总到目标索引中 `example_rollup`：

```json
PUT _plugins/_rollup/jobs/example
{
  "rollup": {
    "enabled": true,
    "schedule": {
      "interval": {
        "period": 1,
        "unit": "Minutes",
        "start_time": 1602100553
      }
    },
    "last_updated_time": 1602100553,
    "description": "An example policy that rolls up the sample ecommerce data",
    "source_index": "opensearch_dashboards_sample_data_ecommerce",
    "target_index": "example_rollup",
    "page_size": 1000,
    "delay": 0,
    "continuous": false,
    "dimensions": [
      {
        "date_histogram": {
          "source_field": "order_date",
          "fixed_interval": "60m",
          "timezone": "America/Los_Angeles"
        }
      },
      {
        "terms": {
          "source_field": "customer_gender"
        }
      },
      {
        "terms": {
          "source_field": "geoip.city_name"
        }
      },
      {
        "terms": {
          "source_field": "geoip.region_name"
        }
      },
      {
        "terms": {
          "source_field": "day_of_week"
        }
      }
    ],
    "metrics": [
      {
        "source_field": "taxless_total_price",
        "metrics": [
          {
            "avg": {}
          },
          {
            "sum": {}
          },
          {
            "max": {}
          },
          {
            "min": {}
          },
          {
            "value_count": {}
          }
        ]
      },
      {
        "source_field": "total_quantity",
        "metrics": [
          {
            "avg": {}
          },
          {
            "max": {}
          }
        ]
      }
    ]
  }
}
```

你可以在索引中 `example_rollup` 查询汇总作业中设置的字段的术语聚合。你将得到与原始 `opensearch_dashboards_sample_data_ecommerce` 源索引相同的响应：

```json
POST example_rollup/_search
{
  "size": 0,
  "query": {
    "bool": {
      "must": {"term": { "geoip.region_name": "California" } }
    }
  },
  "aggregations": {
    "daily_numbers": {
      "terms": {
        "field": "day_of_week"
      },
      "aggs": {
        "per_city": {
          "terms": {
            "field": "geoip.city_name"
          },
          "aggregations": {
            "average quantity": {
               "avg": {
                  "field": "total_quantity"
                }
              }
            }
          },
          "total_revenue": {
            "sum": {
              "field": "taxless_total_price"
          }
        }
      }
    }
  }
}
```

#### 响应示例

```json
{
  "took" : 14,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 281,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "daily_numbers" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "Friday",
          "doc_count" : 59,
          "total_revenue" : {
            "value" : 4858.84375
          },
          "per_city" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "Los Angeles",
                "doc_count" : 59,
                "average quantity" : {
                  "value" : 2.305084745762712
                }
              }
            ]
          }
        },
        {
          "key" : "Saturday",
          "doc_count" : 46,
          "total_revenue" : {
            "value" : 3547.203125
          },
          "per_city" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "Los Angeles",
                "doc_count" : 46,
                "average quantity" : {
                  "value" : 2.260869565217391
                }
              }
            ]
          }
        },
        {
          "key" : "Tuesday",
          "doc_count" : 45,
          "total_revenue" : {
            "value" : 3983.28125
          },
          "per_city" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "Los Angeles",
                "doc_count" : 45,
                "average quantity" : {
                  "value" : 2.2888888888888888
                }
              }
            ]
          }
        },
        {
          "key" : "Sunday",
          "doc_count" : 44,
          "total_revenue" : {
            "value" : 3308.1640625
          },
          "per_city" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "Los Angeles",
                "doc_count" : 44,
                "average quantity" : {
                  "value" : 2.090909090909091
                }
              }
            ]
          }
        },
        {
          "key" : "Thursday",
          "doc_count" : 40,
          "total_revenue" : {
            "value" : 2876.125
          },
          "per_city" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "Los Angeles",
                "doc_count" : 40,
                "average quantity" : {
                  "value" : 2.3
                }
              }
            ]
          }
        },
        {
          "key" : "Monday",
          "doc_count" : 38,
          "total_revenue" : {
            "value" : 2673.453125
          },
          "per_city" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "Los Angeles",
                "doc_count" : 38,
                "average quantity" : {
                  "value" : 2.1578947368421053
                }
              }
            ]
          }
        },
        {
          "key" : "Wednesday",
          "doc_count" : 38,
          "total_revenue" : {
            "value" : 3202.453125
          },
          "per_city" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "Los Angeles",
                "doc_count" : 38,
                "average quantity" : {
                  "value" : 2.236842105263158
                }
              }
            ]
          }
        }
      ]
    }
  }
}
```

## doc_count 字段

 `doc_count` 存储桶聚合中的字段包含每个存储桶中收集的文档数。计算存储桶 `doc_count` 时，文档数将按每个汇总文档中预先聚合的文档数递增。从汇总搜索返回的文档数表示源索引中匹配文档的 `doc_count` 总数。无论你搜索源索引还是汇总目标索引，每个存储桶的文档计数都是相同的。

## 查询字符串查询

若要利用查询 DSL 中更短且更易于编写的字符串，可以使用[查询字符串]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/full-text/query-string/)来简化汇总索引中的搜索查询。若要使用查询字符串，请将以下字段添加到汇总搜索请求中：

```json
"query": {
      "query_string": {
          "query": "field_name:field_value"
      }
  }
```

以下示例使用带有 `*` 通配符运算符的查询字符串在名为 `my_server_logs_rollup`：

```json
GET my_server_logs_rollup/_search
{
  "size": 0,
  "query": {
      "query_string": {
          "query": "email* OR inventory",
          "default_field": "service_name"
      }
  },  
  
  "aggs": {
    "service_name": {
      "terms": {
        "field": "service_name"
      },
      "aggs": {
        "region": {
          "terms": {
            "field": "region"
          },
          "aggs": {
            "average quantity": {
               "avg": {
                  "field": "cpu_usage"
                }
              }
            }
          }
        }
      }
    }
}
```

有关查询字符串查询参数的详细信息，请参阅[查询字符串查询]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/full-text/query-string/#parameters)。

## 动态目标索引

<style>
.nobr { white-space: nowrap }
</style>

在 ISM rollup 中，该 `target_index` 字段可能包含在每次汇总索引时编译的模板。例如，如果将 `target_index` 字段指定为 `{% raw %}rollup_ndx-{{ctx.source_index}}{% endraw %}` <span style="white-space: nowrap">，</span>则源索引将汇总到目标索引 `log-000001` `rollup_ndx-log-000001` 中。这允许你将数据汇总到多个基于时间的索引中，并为每个源索引创建一个汇总作业。

 `source_index` {% raw %}{% endraw %} `{{ctx.source_index}}` 中的参数不能包含通配符。{:.note}

## 搜索多个汇总索引

将数据汇总到多个目标索引中时，可以对所有汇总索引运行一次搜索。要搜索具有相同汇总的多个目标索引，请将索引名称指定为逗号分隔列表或通配符模式。例如，使用 `target_index` as <span style="white-space: nowrap"></span> `{% raw %}rollup_ndx-{{ctx.source_index}}{% endraw %}` 和以 开头 `rollup_ndx-log*` 的 `log` 源索引指定模式。或者，要搜索汇总的 log-000001 和 log-000002 索引，请指定列表 `rollup_ndx-log-000001,rollup_ndx-log-000002`。

不能使用同一查询搜索汇总索引和非汇总索引的混合。{:.note}

## 例

以下示例演示了 `doc_count` 字段、动态索引名称以及使用同一汇总搜索多个汇总索引。

**步骤 1：**为 ISM 添加索引模板，以管理别名为： `log`

```json
PUT _index_template/ism_rollover
{
  "index_patterns": ["log*"],
  "template": {
   "settings": {
    "plugins.index_state_management.rollover_alias": "log"
   }
 }
}
```

**步骤 2：**设置 ISM 滚动更新策略，以便在将一个文档上载到该索引后滚动更新名称以开头 `log*` 的任何索引，然后汇总单个后备索引。目标索引名称是从源索引名称动态生成的，方法是在源索引名称前面加上字符串 `rollup_ndx-`。

```json
PUT _plugins/_ism/policies/rollover_policy 
{ 
  "policy": { 
    "description": "Example rollover policy.", 
    "default_state": "rollover", 
    "states": [ 
      { 
        "name": "rollover", 
        "actions": [ 
          { 
            "rollover": { 
              "min_doc_count": 1 
            } 
          } 
        ], 
        "transitions": [ 
          { 
            "state_name": "rp" 
          } 
        ] 
      }, 
      { 
        "name": "rp", 
        "actions": [
          { 
            "rollup": { 
              "ism_rollup": { 
                "target_index": {% raw %}"rollup_ndx-{{ctx.source_index}}"{% endraw %}, 
                "description": "Example rollup job", 
                "page_size": 200, 
                "dimensions": [ 
                  { 
                    "date_histogram": { 
                      "source_field": "ts", 
                      "fixed_interval": "60m", 
                      "timezone": "America/Los_Angeles" 
                    } 
                  }, 
                  { 
                    "terms": { 
                      "source_field": "message.keyword" 
                    } 
                  } 
                ], 
                "metrics": [ 
                  { 
                    "source_field": "msg_size", 
                    "metrics": [ 
                      { 
                        "sum": {} 
                      } 
                    ]
                  } 
                ]
              } 
            } 
          } 
        ], 
        "transitions": [] 
      } 
    ], 
    "ism_template": { 
      "index_patterns": ["log*"], 
      "priority": 100 
    } 
  } 
}
```

**步骤 3：**创建一个名为 `log-000001` 该 `log` 索引的索引并为其设置别名。

```json
PUT log-000001
{
  "aliases": {
    "log": {
      "is_write_index": true
    }
  }
}
```

**步骤 4：**将四个文档索引到上面创建的索引中。其中两个文档显示消息“成功”，两个文档显示消息“错误”。

```json
POST log/_doc?refresh=true 
{ 
  "ts" : "2022-08-26T09:28:48-04:00", 
  "message": "Success", 
  "msg_size": 10 
}
```

```json
POST log/_doc?refresh=true 
{ 
  "ts" : "2022-08-26T10:06:25-04:00", 
  "message": "Error", 
  "msg_size": 20 
}
```

```json
POST log/_doc?refresh=true 
{ 
  "ts" : "2022-08-26T10:23:54-04:00", 
  "message": "Error", 
  "msg_size": 30 
}
```

```json
POST log/_doc?refresh=true 
{ 
  "ts" : "2022-08-26T10:53:41-04:00", 
  "message": "Success", 
  "msg_size": 40 
}
```

为第一个文档编制索引后，将执行鼠标悬停操作。此操作将创建附加到它的索引 `log-000002` `rollover_policy`。然后执行汇总操作，这将创建汇总索引 `rollup_ndx-log-000001`。

要监视滚动更新和汇总索引创建的状态，可以使用 ISM 解释 API：{： `GET _plugins/_ism/explain`.tip}

**步骤 5：**搜索汇总索引。

```json
GET rollup_ndx-log-*/_search
{
  "size": 0,
  "query": {
    "match_all": {}
  },
  "aggregations": {
    "message_numbers": {
      "terms": {
        "field": "message.keyword"
      },
      "aggs": {
        "per_message": {
          "terms": {
            "field": "message.keyword"
          },
          "aggregations": {
            "sum_message": {
              "sum": {
                "field": "msg_size"
              }
            }
          }
        }
      }
    }
  }
}
```

响应包含两个存储桶，“Error”和“Success”，每个存储桶的文档计数为 2：

```json
{
  "took" : 30,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "message_numbers" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "Success",
          "doc_count" : 2,
          "per_message" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "Success",
                "doc_count" : 2,
                "sum_message" : {
                  "value" : 50.0
                }
              }
            ]
          }
        },
        {
          "key" : "Error",
          "doc_count" : 2,
          "per_message" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "Error",
                "doc_count" : 2,
                "sum_message" : {
                  "value" : 50.0
                }
              }
            ]
          }
        }
      ]
    }
  }
}
```

## 索引编解码器注意事项

有关索引编解码器注意事项，请参见[索引编解码器]({{site.url}}{{site.baseurl}}/im-plugin/index-codecs/#index-rollups-and-transforms)。