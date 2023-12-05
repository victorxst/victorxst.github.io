---
layout: default
title: API
parent: 警报
nav_order: 15
redirect_from:
  - /monitoring-plugins/alerting/api/
---

# 提醒API

使用警报API以编程方式创建，更新和管理显示器和警报。有关专门支持复合监视器的API，请参见[使用API管理复合监视器]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/composite-monitors/#managing-composite-monitors-with-the-api)。

---

<详细信息关闭的markdown ="block">
  <summary>
    目录
  </summary>
  {: .text-delta }
- TOC
{:toc}
</delect>

---

## 创建一个查询-级别监视器
引入1.0
{: .label .label-purple }

询问-电平监视器运行查询，并检查结果是否应触发警报。询问-级别监视器一次只能触发一个警报。有关查询的更多信息-水平监视器和水桶-关卡监视器，请参阅[创建监视器]({{site.url}}{{site.baseurl}}/monitoring-plugins/alerting/monitors/)。

#### 示例请求

```json
POST _plugins/_alerting/monitors
{
  "type": "monitor",
  "name": "test-monitor",
  "monitor_type": "query_level_monitor",
  "enabled": true,
  "schedule": {
    "period": {
      "interval": 1,
      "unit": "MINUTES"
    }
  },
  "inputs": [{
    "search": {
      "indices": ["movies"],
      "query": {
        "size": 0,
        "aggregations": {},
        "query": {
          "bool": {
            "filter": {
              "range": {
                "@timestamp": {
                  "gte": {% raw %}"{{period_end}}||-1h"{% endraw %},
                  "lte": {% raw %}"{{period_end}}"{% endraw %},
                  "format": "epoch_millis"
                }
              }
            }
          }
        }
      }
    }
  }],
  "triggers": [{
    "name": "test-trigger",
    "severity": "1",
    "condition": {
      "script": {
        "source": "ctx.results[0].hits.total.value > 0",
        "lang": "painless"
      }
    },
    "actions": [{
      "name": "test-action",
      "destination_id": "ld7912sBlQ5JUWWFThoW",
      "message_template": {
        "source": "This is my message body."
      },
      "throttle_enabled": true,
      "throttle": {
        "value": 27,
        "unit": "MINUTES"
      },
      "subject_template": {
        "source": "TheSubject"
      }
    }]
  }]
}
```

如果您为目的地使用自定义Webhook，并且需要将JSON嵌入消息正文中，请确保逃脱您的报价：

```json
{
  "message_template": {
    {% raw %}"source": "{ \"text\": \"Monitor {{ctx.monitor.name}} just entered alert status. Please investigate the issue. - Trigger: {{ctx.trigger.name}} - Severity: {{ctx.trigger.severity}} - Period start: {{ctx.periodStart}} - Period end: {{ctx.periodEnd}}\" }"{% endraw %}
  }
}
```

可选地，要指定后端角色，您可以添加`rbac_roles` 参数和后端角色名称到创建监视器请求的底部。

#### 示例请求

以下请求会创建查询-级别监视器并提供两个后端角色，`role1` 和`role2`。请求底部的部分显示了用此语法指定角色的行：`"rbac_roles": ["role1", "role2"]`。

```json
POST _plugins/_alerting/monitors
{
  "type": "monitor",
  "name": "test-monitor",
  "monitor_type": "query_level_monitor",
  "enabled": true,
  "schedule": {
    "period": {
      "interval": 1,
      "unit": "MINUTES"
    }
  },
  "inputs": [{
    "search": {
      "indices": ["movies"],
      "query": {
        "size": 0,
        "aggregations": {},
        "query": {
          "bool": {
            "filter": {
              "range": {
                "@timestamp": {
                  "gte": "{{period_end}}||-1h",
                  "lte": "{{period_end}}",
                  "format": "epoch_millis"
                }
              }
            }
          }
        }
      }
    }
  }],
  "triggers": [{
    "name": "test-trigger",
    "severity": "1",
    "condition": {
      "script": {
        "source": "ctx.results[0].hits.total.value > 0",
        "lang": "painless"
      }
    },
    "actions": [{
      "name": "test-action",
      "destination_id": "ld7912sBlQ5JUWWFThoW",
      "message_template": {
        "source": "This is my message body."
      },
      "throttle_enabled": true,
      "throttle": {
        "value": 27,
        "unit": "MINUTES"
      },
      "subject_template": {
        "source": "TheSubject"
      }
    }]
  }],
  "rbac_roles": ["role1", "role2"]
}
```

要了解有关使用后端角色限制访问的更多信息，请参见[（高级）限制后端角色访问]({{site.url}}{{site.baseurl}}/monitoring-plugins/alerting/security/#advanced-limit-access-by-backend-role)。

#### 示例响应

```json
{
  "_id": "vd5k2GsBlQ5JUWWFxhsP",
  "_version": 1,
  "_seq_no": 7,
  "_primary_term": 1,
  "monitor": {
    "type": "monitor",
    "schema_version": 1,
    "name": "test-monitor",
    "enabled": true,
    "enabled_time": 1562703611363,
    "schedule": {
      "period": {
        "interval": 1,
        "unit": "MINUTES"
      }
    },
    "inputs": [{
      "search": {
        "indices": [
          "movies"
        ],
        "query": {
          "size": 0,
          "query": {
            "bool": {
              "filter": [{
                "range": {
                  "@timestamp": {
                    "from": {% raw %}"{{period_end}}||-1h"{% endraw %},
                    "to": {% raw %}"{{period_end}}"{% endraw %},
                    "include_lower": true,
                    "include_upper": true,
                    "format": "epoch_millis",
                    "boost": 1
                  }
                }
              }],
              "adjust_pure_negative": true,
              "boost": 1
            }
          },
          "aggregations": {}
        }
      }
    }],
    "triggers": [{
      "id": "ud5k2GsBlQ5JUWWFxRvi",
      "name": "test-trigger",
      "severity": "1",
      "condition": {
        "script": {
          "source": "ctx.results[0].hits.total.value > 0",
          "lang": "painless"
        }
      },
      "actions": [{
        "id": "ut5k2GsBlQ5JUWWFxRvj",
        "name": "test-action",
        "destination_id": "ld7912sBlQ5JUWWFThoW",
        "message_template": {
          "source": "This is my message body.",
          "lang": "mustache"
        },
        "throttle_enabled": false,
        "subject_template": {
          "source": "Subject",
          "lang": "mustache"
        }
      }]
    }],
    "last_update_time": 1562703611363
  }
}
```

如果要指定一个时区，可以通过包括一个[cron表达]({{site.url}}{{site.baseurl}}/monitoring-plugins/alerting/cron/) 带有一个时区名称`schedule` 您的请求部分。

以下示例创建了一个监视器，该显示器在每个月的第1天在太平洋时间下午12:10运行。

#### 示例请求

```json
{
  "type": "monitor",
  "name": "test-monitor",
  "monitor_type": "query_level_monitor",
  "enabled": true,
  "schedule": {
    "cron" : {
        "expression": "10 12 1 * *",
        "timezone": "America/Los_Angeles"
    }
  },
  "inputs": [{
    "search": {
      "indices": ["movies"],
      "query": {
        "size": 0,
        "aggregations": {},
        "query": {
          "bool": {
            "filter": {
              "range": {
                "@timestamp": {
                  "gte": {% raw %}"{{period_end}}||-1h"{% endraw %},
                  "lte": {% raw %}"{{period_end}}"{% endraw %},
                  "format": "epoch_millis"
                }
              }
            }
          }
        }
      }
    }
  }],
  "triggers": [{
    "name": "test-trigger",
    "severity": "1",
    "condition": {
      "script": {
        "source": "ctx.results[0].hits.total.value > 0",
        "lang": "painless"
      }
    },
    "actions": [{
      "name": "test-action",
      "destination_id": "ld7912sBlQ5JUWWFThoW",
      "message_template": {
        "source": "This is a message body."
      },
      "throttle_enabled": true,
      "throttle": {
        "value": 27,
        "unit": "MINUTES"
      },
      "subject_template": {
        "source": "Subject"
      }
    }]
  }]
}
```

有关时区名称的完整列表，请参阅[维基百科](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)。警报插件使用Java[时区](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/TimeZone.html) 班级转换[`ZoneId`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/ZoneId.html) 到有效时区。

---

## Bucket-level monitors

桶-级别监视器将结果分类为按字段分开的桶。然后，监视器通过每个存储桶的结果运行您的脚本，并评估是否触发警报。有关存储桶的更多信息-级别和查询-级别监视器，请参阅[创建显示器]（{{site.url}}} {{site.baseurl}}/Monitoring-插件/警报/监视器/）。

```json
邮政_plugins/_ alerting/监视器
{
  "type"："monitor"，
  "name"："Demo bucket-level monitor"，
  "monitor_type"："bucket_level_monitor"，
  "enabled"： 真的，
  "schedule"：{
    "period"：{
      "interval"：1，
      "unit"："MINUTES"
    }
  }，，
  "inputs"：[[
    {
      "search"：{
        "indices"：[[
          "movies"
        ]，，
        "query"：{
          "size"：0，
          "query"：{
            "bool"：{
              "filter"：[[
                {
                  "range"：{
                    "order_date"：{
                      "from"： {％ 生的 ％}"{{period_end}}||-1h"{％endraw％}，
                      "to"： {％ 生的 ％}"{{period_end}}"{％endraw％}，
                      "include_lower"： 真的，
                      "include_upper"： 真的，
                      "format"："epoch_millis"
                    }
                  }
                }
              这是给出的
            }
          }，，
          "aggregations"：{
            "composite_agg"：{
              "composite"：{
                "sources"：[[
                  {
                    "user"：{
                      "terms"：{
                        "field"："user"
                      }
                    }
                  }
                这是给出的
              }，，
              "aggregations"：{
                "avg_products_base_price"：{
                  "avg"：{
                    "field"："products.base_price"
                  }
                }
              }
            }
          }
        }
      }
    }
  ]，，
  "triggers"：[[
    {
      "bucket_level_trigger"：{
        "name"："test-trigger"，
        "severity"："1"，
        "condition"：{
          "buckets_path"：{
            "_count"："_count"，
            "avg_products_base_price"："avg_products_base_price"
          }，，
          "parent_bucket_path"："composite_agg"，
          "script"：{
            "source"："params._count > 50 || params.avg_products_base_price < 35"，
            "lang"："painless"
          }
        }，，
        "actions"：[[
          {
            "name"："test-action"，
            "destination_id"："E4o5hnsB6KjPKmHtpfCA"，
            "message_template"：{
              "source"： {％ 生的 ％}"""Monitor {{ctx.monitor.name}} just entered alert status. Please investigate the issue.   - Trigger: {{ctx.trigger.name}}   - Severity: {{ctx.trigger.severity}}   - Period start: {{ctx.periodStart}}   - Period end: {{ctx.periodEnd}}    - Deduped Alerts:   {{ctx.dedupedAlerts}}     * {{id}} : {{bucket_keys}}   {{ctx.dedupedAlerts}}    - New Alerts:   {{ctx.newAlerts}}     * {{id}} : {{bucket_keys}}   {{ctx.newAlerts}}    - Completed Alerts:   {{ctx.completedAlerts}}     * {{id}} : {{bucket_keys}}   {{ctx.completedAlerts}}"""{％endraw％}，
              "lang"："mustache"
            }，，
            "throttle_enabled"： 错误的，
            "throttle"：{
              "value"：10，
              "unit"："MINUTES"
            }，，
            "action_execution_policy"：{
              "action_execution_scope"：{
                "per_alert"：{
                  "actionable_alerts"：[[
                    "DEDUPED"，
                    "NEW"
                  这是给出的
                }
              }
            }，，
            "subject_template"：{
              "source"："The Subject"，
              "lang"："mustache"
            }
          }
        这是给出的
      }
    }
  这是给出的
}
```

#### Example response
```json
{
  "_id" ："Dfxr63sBwex6DxEhHV5N"，
  "_version" ：1，
  "_seq_no" ：3，
  "_primary_term" ：1，
  "monitor" ：{
    "type" ："monitor"，
    "schema_version" ：4，
    "name" ："Demo a bucket-level monitor"，
    "monitor_type" ："bucket_level_monitor"，
    "user" ：{
      "name" ：""，
      "backend_roles" ：[]，，
      "roles" ：[]，，
      "custom_attribute_names" ：[]，，
      "user_requested_tenant" ： 无效的
    }，，
    "enabled" ： 真的，
    "enabled_time" ：1631742270785，
    "schedule" ：{
      "period" ：{
        "interval" ：1，
        "unit" ："MINUTES"
      }
    }，，
    "inputs" ：[[
      {
        "search" ：{
          "indices" ：[[
            "opensearch_dashboards_sample_data_flights"
          ]，，
          "query" ：{
            "size" ：0，
            "query" ：{
              "bool" ：{
                "filter" ：[[
                  {
                    "range" ：{
                      "order_date" ：{
                        "from" ： {％ 生的 ％}"{{period_end}}||-1h"{％endraw％}，
                        "to" ： {％ 生的 ％}"{{period_end}}"{％endraw％}，
                        "include_lower" ： 真的，
                        "include_upper" ： 真的，
                        "format" ："epoch_millis"，
                        "boost" ：1.0
                      }
                    }
                  }
                ]，，
                "adjust_pure_negative" ： 真的，
                "boost" ：1.0
              }
            }，，
            "aggregations" ：{
              "composite_agg" ：{
                "composite" ：{
                  "size" ：10，
                  "sources" ：[[
                    {
                      "user" ：{
                        "terms" ：{
                          "field" ："user"，
                          "missing_bucket" ： 错误的，
                          "order" ："asc"
                        }
                      }
                    }
                  这是给出的
                }，，
                "aggregations" ：{
                  "avg_products_base_price" ：{
                    "avg" ：{
                      "field" ："products.base_price"
                    }
                  }
                }
              }
            }
          }
        }
      }
    ]，，
    "triggers" ：[[
      {
        "bucket_level_trigger" ：{
          "id" ："C_xr63sBwex6DxEhHV5B"，
          "name" ："test-trigger"，
          "severity" ："1"，
          "condition" ：{
            "buckets_path" ：{
              "_count" ："_count"，
              "avg_products_base_price" ："avg_products_base_price"
            }，，
            "parent_bucket_path" ："composite_agg"，
            "script" ：{
              "source" ："params._count > 50 || params.avg_products_base_price < 35"，
              "lang" ："painless"
            }，，
            "gap_policy" ："skip"
          }，，
          "actions" ：[[
            {
              "id" ："DPxr63sBwex6DxEhHV5B"，
              "name" ："test-action"，
              "destination_id" ："E4o5hnsB6KjPKmHtpfCA"，
              "message_template" ：{
                "source" ： {％ 生的 ％}"Monitor {{ctx.monitor.name}} just entered alert status. Please investigate the issue.   - Trigger: {{ctx.trigger.name}}   - Severity: {{ctx.trigger.severity}}   - Period start: {{ctx.periodStart}}   - Period end: {{ctx.periodEnd}}    - Deduped Alerts:   {{ctx.dedupedAlerts}}     * {{id}} : {{bucket_keys}}   {{ctx.dedupedAlerts}}    - New Alerts:   {{ctx.newAlerts}}     * {{id}} : {{bucket_keys}}   {{ctx.newAlerts}}    - Completed Alerts:   {{ctx.completedAlerts}}     * {{id}} : {{bucket_keys}}   {{ctx.completedAlerts}}"{％endraw％}，
                "lang" ："mustache"
              }，，
              "throttle_enabled" ： 错误的，
              "subject_template" ：{
                "source" ："The Subject"，
                "lang" ："mustache"
              }，，
              "throttle" ：{
                "value" ：10，
                "unit" ："MINUTES"
              }，，
              "action_execution_policy" ：{
                "action_execution_scope" ：{
                  "per_alert" ：{
                    "actionable_alerts" ：[[
                      "DEDUPED"，
                      "NEW"
                    这是给出的
                  }
                }
              }
            }
          这是给出的
        }
      }
    ]，，
    "last_update_time" ：1631742270785
  }
}
```
## Document-level monitors
引入2.0
{: .label .label-purple }

文档-Level监视器检查索引中的单个文档是否匹配触发条件。如果是这样，监视器会生成警报通知。当您使用文档查询时-级别监视器，返回与触发条件匹配的每个文档的结果。您可以根据查询名称，查询ID或组合多个查询的标签创建触发条件。

要了解有关与文档类似功能的每个文档监视器的更多信息-等级监视器API，请参见[{{stite.url}} {{site.baseurl}}/观察-你的-数据/警报/显示器/）。

### Search the findings index

您可以使用警报搜索API操作搜索发现索引`.opensearch-alerting-finding*` 有关带有GET请求的可用文档发现。默认情况下，没有路径参数的GET请求返回所有可用的发现。

要检索任何可用发现，请在没有任何路径参数的情况下发送Get请求，如下所示：

```json
get/_plugins/_ alerting/findings/_search？
```


要检索元数据以获取单个文档查找条目，您可以通过其发现搜索发现`findingId` 如下：

```json
get/_plugins/_ alerting/findings/_search？findiD = gkqhj8wjit3bxjgfioxc
```

响应返回了个人查找条目的数量`total_findings` 场地。

为了在调查结果搜索中获得更具体的结果，您可以使用下表中定义的任何可选路径参数：

路径参数| 描述| 用法
:--- | :--- ：：---
`findingId` | 查找条目的标识符。| 查找ID在初始查询响应中返回。
`sortString` | 该字段指定了警报插件用于对发现进行排序的字符串。| 默认值是`id`。
`sortOrder` | 对发现列表进行排序的命令，无论是上升还是下降。| 使用`sortOrder=asc` 指示上升或`sortOrder=desc` 用于降序排序订单。
`size` | 响应中返回的最大结果数的可选限制。| 没有最小值或最大值。
`startIndex` | 分页指示器。| 默认为`0`。
`searchString` | 您想要在搜索中返回的发现属性。| 要在特定索引中搜索，请在请求路径中指定索引名称。例如，在`indexABC` 索引，使用`searchString=indexABC'.

### Create a document-level monitor

You can create a document-level monitor with a POST request that provides the monitor details in the request body.
At a minimum, you need to provide the following details: specify the queries or combinations by tag with the `输入` field, a valid trigger condition, and provide the notification message in the `行动` 场地。

下表显示了用于每个触发选项的语法：

触发选项| 定义| 句法
:--- | :--- ：：---
标签| 为将多个查询与应用此标签匹配的文档创建警报。如果您按单个标签对多个查询进行分组，则可以将其设置为触发警报，如果此标签名称返回结果。| `query[tag=<tag-name>]`
按名称查询| 创建符合或返回的文档的警报。| `query[name=<query-name>]`
通过ID查询| 为已确定的查询返回的文档创建警报。| `query[id=<query-id>]`

#### Example request

以下示例显示了如何创建文档-等级监视器：

```json
邮政_plugins/_ alerting/监视器
{
  "type"："monitor"，
  "monitor_type"："doc_level_monitor"，
  "name"："Example document-level monitor"，
  "enabled"： 真的，
  "schedule"：{
    "period"：{
      "interval"：1，
      "unit"："MINUTES"
    }
  }，，
  "inputs"：[[
    {
      "doc_level_input"：{
        "description"："Example document-level monitor for audit logs"，
        "indices"：[[
          "audit-logs"
        ]，，
        "queries"：[[
        {
            "id"："nKQnFYABit3BxjGfiOXC"，
            "name"："sigma-123"，
            "query"："region:\"我们-西方-2 \""，
            "tags"：[[
                "tag1"
            这是给出的
        }，，
        {
            "id"："gKQnABEJit3BxjGfiOXC"，
            "name"："sigma-456"，
            "query"："region:\"我们-东方-1 \""，
            "tags"：[[
                "tag2"
            这是给出的
        }，，
        {
            "id"："h4J2ABEFNW3vxjGfiOXC"，
            "name"："sigma-789"，
            "query"："message:\"这是与IAD区域的单独错误\""，
            "tags"：[[
                "tag3"
            这是给出的
        }
    这是给出的
      }
    }
  ]，，
    "triggers"：[{{"document_level_trigger"：{
      "name"："test-trigger"，
      "severity"："1"，
      "condition"：{
        "script"：{
          "source"："(query[name=sigma-123] || query[tag=tag3]) && query[name=sigma-789]"，
          "lang"："painless"
        }
      }，，
      "actions"：[[
        {
            "name"："test-action"，
            "destination_id"："E4o5hnsB6KjPKmHtpfCA"，
            "message_template"：{
                "source"： {％ 生的 ％}"""Monitor  just entered alert status. Please investigate the issue. Related Finding Ids: {{ctx.alerts.0.finding_ids}}, Related Document Ids: {{ctx.alerts.0.related_doc_ids}}"""{％endraw％}，
                "lang"："mustache"
            }，，
            "action_execution_policy"：{
                "action_execution_scope"：{
                    "per_alert"：{
                        "actionable_alerts"：[]
                    }
                }
            }，，
            "subject_template"：{
                "source"："The Subject"，
                "lang"："mustache"
            }
         }
      这是给出的
  }}]]
}

```

### Limitations

如果您运行文档-级别查询在索引重新索引时，API响应将不会返回重新索引结果。要获取更新，请等到重新索引过程完成，然后重新汇总查询。
{: .tip}

## Update monitor
引入1.0
{: .label .label-purple }

更新监视器时，您可以选择包括`seq_no` 和`primary_term` 作为URL参数。如果这些数字与现有监视器不匹配或不存在监视器，则警报插件会引发错误。OpenSearch会自动增加版本号和序列编号（请参阅示例响应）。

#### Request

```json
放置_plugins/_ alerting/monitors/<Monital_id>
{
  "type"："monitor"，
  "name"："test-monitor"，
  "enabled"： 真的，
  "enabled_time"：1551466220455，
  "schedule"：{
    "period"：{
      "interval"：1，
      "unit"："MINUTES"
    }
  }，，
  "inputs"：[{{
    "search"：{
      "indices"：[[
        "*"
      ]，，
      "query"：{
        "query"：{
          "match_all"：{
            "boost"：1
          }
        }
      }
    }
  }]，
  "triggers"：[{{
    "id"："StaeOmkBC25HCRGmL_y-"，
    "name"："test-trigger"，
    "severity"："1"，
    "condition"：{
      "script"：{
        "source"："return true"，
        "lang"："painless"
      }
    }，，
    "actions"：[{{
      "name"："test-action"，
      "destination_id"："RtaaOmkBC25HCRGm0fxi"，
      "subject_template"：{
        "source"："My Message Subject"，
        "lang"："mustache"
      }，，
      "message_template"：{
        "source"："This is my message body."，
        "lang"："mustache"
      }
    ]]
  }]，
  "last_update_time"：1551466639295
}

put _plugins/_alerting/monitors/<morita_id>？if_seq_no = 3＆if_primary_term = 1
{
  "type"："monitor"，
  "name"："test-monitor"，
  "enabled"： 真的，
  "enabled_time"：1551466220455，
  "schedule"：{
    "period"：{
      "interval"：1，
      "unit"："MINUTES"
    }
  }，，
  "inputs"：[{{
    "search"：{
      "indices"：[[
        "*"
      ]，，
      "query"：{
        "query"：{
          "match_all"：{
            "boost"：1
          }
        }
      }
    }
  }]，
  "triggers"：[{{
    "id"："StaeOmkBC25HCRGmL_y-"，
    "name"："test-trigger"，
    "severity"："1"，
    "condition"：{
      "script"：{
        "source"："return true"，
        "lang"："painless"
      }
    }，，
    "actions"：[{{
      "name"："test-action"，
      "destination_id"："RtaaOmkBC25HCRGm0fxi"，
      "subject_template"：{
        "source"："My Message Subject"，
        "lang"："mustache"
      }，，
      "message_template"：{
        "source"："This is my message body."，
        "lang"："mustache"
      }
    ]]
  }]，
  "last_update_time"：1551466639295
}
```

#### Example response

```json
{
  "_id"："Q9aXOmkBC25HCRGmzfw-"，
  "_version"：4，
  "_seq_no"：4，
  "_primary_term"：1，
  "monitor"：{
    "type"："monitor"，
    "name"："test-monitor"，
    "enabled"： 真的，
    "enabled_time"：1551466220455，
    "schedule"：{
      "period"：{
        "interval"：1，
        "unit"："MINUTES"
      }
    }，，
    "inputs"：[{{
      "search"：{
        "indices"：[[
          "*"
        ]，，
        "query"：{
          "query"：{
            "match_all"：{
              "boost"：1
            }
          }
        }
      }
    }]，
    "triggers"：[{{
      "id"："StaeOmkBC25HCRGmL_y-"，
      "name"："test-trigger"，
      "severity"："1"，
      "condition"：{
        "script"：{
          "source"："return true"，
          "lang"："painless"
        }
      }，，
      "actions"：[{{
        "name"："test-action"，
        "destination_id"："RtaaOmkBC25HCRGm0fxi"，
        "subject_template"：{
          "source"："My Message Subject"，
          "lang"："mustache"
        }，，
        "message_template"：{
          "source"："This is my message body."，
          "lang"："mustache"
        }
      ]]
    }]，
    "last_update_time"：1551466761596
  }
}
```


---

## 获取显示器
引入1.0
{: .label .label-purple }

#### 要求

```
GET _plugins/_alerting/monitors/<monitor_id>
```

#### 示例响应

```json
{
  "_id": "Q9aXOmkBC25HCRGmzfw-",
  "_version": 3,
  "_seq_no": 3,
  "_primary_term": 1,
  "monitor": {
    "type": "monitor",
    "name": "test-monitor",
    "enabled": true,
    "enabled_time": 1551466220455,
    "schedule": {
      "period": {
        "interval": 1,
        "unit": "MINUTES"
      }
    },
    "inputs": [{
      "search": {
        "indices": [
          "*"
        ],
        "query": {
          "query": {
            "match_all": {
              "boost": 1
            }
          }
        }
      }
    }],
    "triggers": [{
      "id": "StaeOmkBC25HCRGmL_y-",
      "name": "test-trigger",
      "severity": "1",
      "condition": {
        "script": {
          "source": "return true",
          "lang": "painless"
        }
      },
      "actions": [{
        "name": "test-action",
        "destination_id": "RtaaOmkBC25HCRGm0fxi",
        "subject_template": {
          "source": "My Message Subject",
          "lang": "mustache"
        },
        "message_template": {
          "source": "This is my message body.",
          "lang": "mustache"
        }
      }]
    }],
    "last_update_time": 1551466639295
  }
}
```


---

## Monitor stats
引入1.0
{: .label .label-purple }

返回有关警报功能的统计信息。使用`_plugins/_alerting/stats` 查找节点ID和指标。然后，您可以使用这些值向下钻探。

#### Request

```json
获取_plugins/_ alerting/stats
获取_plugins/_ alerting/stats/<metric>
获取_plugins/_ alerting/<node-ID>/统计
获取_plugins/_ alerting/<node-id>/stats/<terric>
```

#### Example response

```json
{
  "_nodes"：{
    "total"：9，
    "successful"：9，
    "failed"：0
  }，，
  "cluster_name"："475300751431:alerting65-dont-delete"，
  "plugins.scheduled_jobs.enabled"： 真的，
  "scheduled_job_index_exists"： 真的，
  "scheduled_job_index_status"："green"，
  "nodes_on_schedule"：9，
  "nodes_not_on_schedule"：0，
  "nodes"：{
    "qWcbKbb-TVyyI-Q7VSeOqA"：{
      "name"："qWcbKbb"，
      "schedule_status"："green"，
      "roles"：[[
        "MASTER"
      ]，，
      "job_scheduling_metrics"：{
        "last_full_sweep_time_millis"：207017，
        "full_sweep_on_time"： 真的
      }，，
      "jobs_info"：{}
    }，，
    "Do-DX9ZcS06Y9w1XbSJo1A"：{
      "name"："Do-DX9Z"，
      "schedule_status"："green"，
      "roles"：[[
        "DATA"，
        "INGEST"
      ]，，
      "job_scheduling_metrics"：{
        "last_full_sweep_time_millis"：230516，
        "full_sweep_on_time"： 真的
      }，，
      "jobs_info"：{}
    }，，
    "n5phkBiYQfS5I0FDzcqjZQ"：{
      "name"："n5phkBi"，
      "schedule_status"："green"，
      "roles"：[[
        "MASTER"
      ]，，
      "job_scheduling_metrics"：{
        "last_full_sweep_time_millis"：228406，
        "full_sweep_on_time"： 真的
      }，，
      "jobs_info"：{}
    }，，
    "Tazzo8cQSY-g3vOjgYYLzA"：{
      "name"："Tazzo8c"，
      "schedule_status"："green"，
      "roles"：[[
        "DATA"，
        "INGEST"
      ]，，
      "job_scheduling_metrics"：{
        "last_full_sweep_time_millis"：211722，
        "full_sweep_on_time"： 真的
      }，，
      "jobs_info"：{
        "i-wsFmkB8NzS6aXjQSk0"：{
          "last_execution_time"：1550864912882，
          "running_on_time"： 真的
        }
      }
    }，，
    "Nyf7F8brTOSJuFPXw6CnpA"：{
      "name"："Nyf7F8b"，
      "schedule_status"："green"，
      "roles"：[[
        "DATA"，
        "INGEST"
      ]，，
      "job_scheduling_metrics"：{
        "last_full_sweep_time_millis"：223300，
        "full_sweep_on_time"： 真的
      }，，
      "jobs_info"：{
        "NbpoFmkBeSe-hD59AKgE"：{
          "last_execution_time"：1550864928354，
          "running_on_time"： 真的
        }，，
        "-LlLFmkBeSe-hD59Ydtb"：{
          "last_execution_time"：1550864732727，
          "running_on_time"： 真的
        }，，
        "pBFxFmkBNXkgNmTBaFj1"：{
          "last_execution_time"：1550863325024，
          "running_on_time"： 真的
        }，，
        "hfasEmkBNXkgNmTBrvIW"：{
          "last_execution_time"：1550862000001，
          "running_on_time"： 真的
        }
      }
    }，，
    "oOdJDIBVT5qbbO3d8VLeEw"：{
      "name"："oOdJDIB"，
      "schedule_status"："green"，
      "roles"：[[
        "DATA"，
        "INGEST"
      ]，，
      "job_scheduling_metrics"：{
        "last_full_sweep_time_millis"：227570，
        "full_sweep_on_time"： 真的
      }，，
      "jobs_info"：{
        "4hKRFmkBNXkgNmTBKjYX"：{
          "last_execution_time"：1550864806101，
          "running_on_time"： 真的
        }
      }
    }，，
    "NRDG6JYgR8m0GOZYQ9QGjQ"：{
      "name"："NRDG6JY"，
      "schedule_status"："green"，
      "roles"：[[
        "MASTER"
      ]，，
      "job_scheduling_metrics"：{
        "last_full_sweep_time_millis"：227652，
        "full_sweep_on_time"： 真的
      }，，
      "jobs_info"：{}
    }，，
    "URMrXRz3Tm-CB72hlsl93Q"：{
      "name"："URMrXRz"，
      "schedule_status"："green"，
      "roles"：[[
        "DATA"，
        "INGEST"
      ]，，
      "job_scheduling_metrics"：{
        "last_full_sweep_time_millis"：231048，
        "full_sweep_on_time"： 真的
      }，，
      "jobs_info"：{
        "m7uKFmkBeSe-hD59jplP"：{
          "running_on_time"： 真的
        }
      }
    }，，
    "eXgt1k9oTRCLmx2HBGElUw"：{
      "name"："eXgt1k9"，
      "schedule_status"："green"，
      "roles"：[[
        "DATA"，
        "INGEST"
      ]，，
      "job_scheduling_metrics"：{
        "last_full_sweep_time_millis"：229234，
        "full_sweep_on_time"： 真的
      }，，
      "jobs_info"：{
        "wWkFFmkBc2NG-PeLntxk"：{
          "running_on_time"： 真的
        }，，
        "3usNFmkB8NzS6aXjO1Gs"：{
          "last_execution_time"：1550863959848，
          "running_on_time"： 真的
        }
      }
    }
  }
}
```


---

## 删除监视器
引入1.0
{: .label .label-purple }

#### 要求

```
DELETE _plugins/_alerting/monitors/<monitor_id>
```

#### 示例响应

```json
{
  "_index": ".opensearch-scheduled-jobs",
  "_id": "OYAHOmgBl3cmwnqZl_yH",
  "_version": 2,
  "result": "deleted",
  "forced_refresh": true,
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "_seq_no": 11,
  "_primary_term": 1
}
```


---

## Search monitors
引入1.0
{: .label .label-purple }

#### Request

```json
获取_plugins/_ alerting/monitors/_search
{
  "query"：{
    "match" ：{
      "monitor.name"："my-monitor-name"
    }
  }
}
```

#### Example response

```json
{
  "took"：17，
  "timed_out"： 错误的，
  "_shards"：{
    "total"：5，
    "successful"：5，
    "skipped"：0，
    "failed"：0
  }，，
  "hits"：{
    "total"：1，
    "max_score"：0.6931472，
    "hits"：[{{
      "_index"：".opensearch-scheduled-jobs"，
      "_type"："_doc"，
      "_id"："eGQi7GcBRS7-AJEqfAnr"，
      "_score"：0.6931472，
      "_source"：{
        "type"："monitor"，
        "name"："my-monitor-name"，
        "enabled"： 真的，
        "enabled_time"：1545854942426，
        "schedule"：{
          "period"：{
            "interval"：1，
            "unit"："MINUTES"
          }
        }，，
        "inputs"：[{{
          "search"：{
            "indices"：[[
              "*"
            ]，，
            "query"：{
              "size"：0，
              "query"：{
                "bool"：{
                  "filter"：[{{
                    "range"：{
                      "@timestamp"：{
                        "from"： {％ 生的 ％}"{{period_end}}||-1h"{％endraw％}，
                        "to"： {％ 生的 ％}"{{period_end}}"{％endraw％}，
                        "include_lower"： 真的，
                        "include_upper"： 真的，
                        "format"："epoch_millis"，
                        "boost"：1
                      }
                    }
                  }]，
                  "adjust_pure_negative"： 真的，
                  "boost"：1
                }
              }，，
              "aggregations"：{}
            }
          }
        }]，
        "triggers"：[{{
          "id"："Sooi7GcB53a0ewuj_6MH"，
          "name"："Over"，
          "severity"："1"，
          "condition"：{
            "script"：{
              "source"："_ctx.results[0].hits.total > 400000"，
              "lang"："painless"
            }
          }，，
          "actions"：[]
        }]，
        "last_update_time"：1545854975758
      }
    ]]
  }
}
```


---

## 运行监视器
引入1.0
{: .label .label-purple }

您可以添加可选`?dryrun=true` 参数到URL以显示运行的结果，而无需发送任何消息。


#### 要求

```json
POST _plugins/_alerting/monitors/<monitor_id>/_execute
```

#### 示例响应

```json
{
  "monitor_name": "logs",
  "period_start": 1547161872322,
  "period_end": 1547161932322,
  "error": null,
  "trigger_results": {
    "Sooi7GcB53a0ewuj_6MH": {
      "name": "Over",
      "triggered": true,
      "error": null,
      "action_results": {}
    }
  }
}
```

---

## Get alerts
引入1.0
{: .label .label-purple }

返回所有警报的数组。

#### Path parameters

下表列出了可用路径参数。所有路径参数都是可选的。

| 范围| 数据类型| 描述
| :--- | :--- | :---
| `sortString` | 细绳| 确定如何对结果进行分类。默认为`monitor_name.keyword`。
| `sortOrder` | 细绳| 确定结果的顺序。选项是`asc` 或者`desc`。默认为`asc`。
| `missing` | 细绳| 选修的。
| `size` | 细绳| 确定要返回的请求的大小。默认为`20`。
| `startIndex` | 细绳| 索引开始。用于登机结果。默认为`0`。
| `searchString` | 细绳| 搜索字符串曾经寻找特定的警报。默认为空字符串。
| `severityLevel` | 细绳| 要过滤的严重程度。默认为`ALL`。
| `alertState` | 细绳| 警报状态要过滤。默认为`ALL`。
| `monitorId` | 细绳| 通过显示器ID过滤。

#### Request

```json
获取_plugins/_ alerting/enitors/nervers
```

#### Response

```json
{
  "alerts"：[[
    {
      "id"："eQURa3gBKo1jAh6qUo49"，
      "version"：300，
      "monitor_id"："awUMa3gBKo1jAh6qu47E"，
      "schema_version"：2，
      "monitor_version"：2，
      "monitor_name"："Example_monitor_name"，
      "monitor_user"：{
        "name"："admin"，
        "backend_roles"：[[
          "admin"
        ]，，
        "roles"：[[
          "all_access"，
          "own_index"
        ]，，
        "custom_attribute_names"：[]，，
        "user_requested_tenant"： 无效的
      }，，
      "trigger_id"："bQUQa3gBKo1jAh6qnY6G"，
      "trigger_name"："Example_trigger_name"，
      "state"："ACTIVE"，
      "error_message"： 无效的，
      "alert_history"：[[
        {
          "timestamp"：1617314504873，
          "message"："Example error message"
        }，，
        {
          "timestamp"：1617312543925，
          "message"："Example error message"
        }
      ]，，
      "severity"："1"，
      "action_execution_results"：[[
        {
          "action_id"："bgUQa3gBKo1jAh6qnY6G"，
          "last_execution_time"：1617317979908，
          "throttled_count"：0
        }
      ]，，
      "start_time"：1616704000492，
      "last_notification_time"：1617317979908，
      "end_time"： 无效的，
      "acknowledged_time"： 无效的
    }
  ]，，
  "totalAlerts"：1
}
```

---

## 确认警报
引入1.0
{: .label .label-purple }

[收到警报后](#get-alerts)，您可以在一个呼叫中确认任何数量的主动警报。如果警报已经处于错误，已完成或已确认状态，则将出现在`failed` 大批。


#### 要求

```json
POST _plugins/_alerting/monitors/<monitor-id>/_acknowledge/alerts
{
  "alerts": ["eQURa3gBKo1jAh6qUo49"]
}
```

#### 示例响应

```json
{
  "success": [
  "eQURa3gBKo1jAh6qUo49"
  ],
  "failed": []
}
```

---

## Create destination
引入1.0
{: .label .label-purple }

#### Requests

```json
发布_plugins/_ alerting/destinations
{
  "name"："my-destination"，
  "type"："slack"，
  "slack"：{
    "url"："http://www.example.com"
  }
}

发布_plugins/_ alerting/destinations
{
  "type"："custom_webhook"，
  "name"："my-custom-destination"，
  "custom_webhook"：{
    "path"："incomingwebhooks/123456-123456-XXXXXX"，
    "header_params"：{
      "Content-Type"："application/json"
    }，，
    "scheme"："HTTPS"，
    "port"：443，
    "query_params"：{
      "token"："R2x1UlN4ZHF8MXxxVFJpelJNVDgzdGNwXXXXXXXXX"
    }，，
    "host"："hooks.chime.aws"
  }
}

发布_plugins/_ alerting/destinations
{
  "type"："email"，
  "name"："my-email-destination"，
  "email"：{
    "email_account_id"："YjY7mXMBx015759_IcfW"，
    "recipients"：[[
      {
        "type"："email_group"，
        "email_group_id"："YzY-mXMBx015759_dscs"
      }，，
      {
        "type"："email"，
        "email"："example@email.com"
      }
    这是给出的
  }
}

// email_account_id和email_group_id将是您创建的email_account和email_group的文档ID。
```

#### Example response

```json
{
  "_id"："nO-yFmkB8NzS6aXjJdiI"，
  "_version" ：1，
  "_seq_no" ：3，
  "_primary_term" ：1，
  "destination"：{
    "type"："slack"，
    "name"："my-destination"，
    "last_update_time"：1550863967624，
    "slack"：{
      "url"："http://www.example.com"
    }
  }
}
```


---

## 更新目的地
引入1.0
{: .label .label-purple }

更新目的地时，您可以选择包括`seq_no` 和`primary_term` 作为URL参数。如果这些数字与现有目标不匹配或目的地不存在，则警报插件会引发错误。OpenSearch会自动增加版本号和序列编号（请参阅示例响应）。

#### 要求

```json
PUT _plugins/_alerting/destinations/<destination-id>
{
  "name": "my-updated-destination",
  "type": "slack",
  "slack": {
    "url": "http://www.example.com"
  }
}

PUT _plugins/_alerting/destinations/<destination-id>?if_seq_no=3&if_primary_term=1
{
  "name": "my-updated-destination",
  "type": "slack",
  "slack": {
    "url": "http://www.example.com"
  }
}
```

#### 示例响应

```json
{
  "_id": "pe-1FmkB8NzS6aXjqvVY",
  "_version" : 2,
  "_seq_no" : 4,
  "_primary_term" : 1,
  "destination": {
    "type": "slack",
    "name": "my-updated-destination",
    "last_update_time": 1550864289375,
    "slack": {
      "url": "http://www.example.com"
    }
  }
}
```


---

## Get destination
引入1.0
{: .label .label-purple }

检索一个目的地。

#### Requests

```json
获取_plugins/_ alerting/destinations/<目的地-id>
```

#### Example response

```json
{
  "totalDestinations"：1，
  "destinations"：[{{
      "id"："1a2a3a4a5a6a7a"，
      "type"："slack"，
      "name"："sample-destination"，
      "user"：{
        "name"："psantos"，
        "backend_roles"：[[
          "human-resources"
        ]，，
        "roles"：[[
          "alerting_full_access"，
          "hr-role"
        ]，，
        "custom_attribute_names"：[]
      }，，
      "schema_version"：3，
      "seq_no"：0，
      "primary_term"：6，
      "last_update_time"：1603943261722，
      "slack"：{
        "url"："https://example.com"
      }
    }
  这是给出的
}
```


---

## 获取目的地
引入1.0
{: .label .label-purple }

检索所有目的地。

#### 要求

```json
GET _plugins/_alerting/destinations
```

#### 示例响应

```json
{
  "totalDestinations": 1,
  "destinations": [{
      "id": "1a2a3a4a5a6a7a",
      "type": "slack",
      "name": "sample-destination",
      "user": {
        "name": "psantos",
        "backend_roles": [
          "human-resources"
        ],
        "roles": [
          "alerting_full_access",
          "hr-role"
        ],
        "custom_attribute_names": []
      },
      "schema_version": 3,
      "seq_no": 0,
      "primary_term": 6,
      "last_update_time": 1603943261722,
      "slack": {
        "url": "https://example.com"
      }
    }
  ]
}
```


---

## Delete destination
引入1.0
{: .label .label-purple }

#### Request

```
删除_plugins/_ alerting/destinations/<目的地-id>
```

#### Example response

```json
{
  "_index"：".opendistro-alerting-config"，
  "_type"："_doc"，
  "_id"："Zu-zFmkB8NzS6aXjLeBI"，
  "_version"：2，
  "result"："deleted"，
  "forced_refresh"： 真的，
  "_shards"：{
    "total"：2，
    "successful"：2，
    "failed"：0
  }，，
  "_seq_no"：8，
  "_primary_term"：1
}
```
---

## 创建电子邮件帐户
引入1.0
{: .label .label-purple }

#### 要求
```json
POST _plugins/_alerting/destinations/email_accounts
{
  "name": "example_account",
  "email": "example@email.com",
  "host": "smtp.email.com",
  "port": 465,
  "method": "ssl"
}
```

#### 示例响应
```json
{
  "_id" : "email_account_id",
  "_version" : 1,
  "_seq_no" : 7,
  "_primary_term" : 2,
  "email_account" : {
    "schema_version" : 2,
    "name" : "example_account",
    "email" : "example@email.com",
    "host" : "smtp.email.com",
    "port" : 465,
    "method" : "ssl"
  }
}
```

## 更新电子邮件帐户
引入1.0
{: .label .label-purple }

更新电子邮件帐户时，您可以选择包括`seq_no` 和`primary_term` 作为URL参数。如果这些数字与现有的电子邮件帐户或电子邮件帐户不存在，则警报插件会引发错误。OpenSearch会自动增加版本号和序列编号（请参阅示例响应）。

#### 要求
```json
PUT _plugins/_alerting/destinations/email_accounts/<email_account_id>
{
  "name": "example_account",
  "email": "example@email.com",
  "host": "smtp.email.com",
  "port": 465,
  "method": "ssl"
}

PUT _plugins/_alerting/destinations/email_accounts/<email_account_id>?if_seq_no=18&if_primary_term=2
{
  "name": "example_account",
  "email": "example@email.com",
  "host": "smtp.email.com",
  "port": 465,
  "method": "ssl"
}
```
#### 示例响应
```json
{
  "_id" : "email_account_id",
  "_version" : 3,
  "_seq_no" : 19,
  "_primary_term" : 2,
  "email_account" : {
    "schema_version" : 2,
    "name" : "example_account",
    "email" : "example@email.com",
    "host" : "smtp.email.com",
    "port" : 465,
    "method" : "ssl"
  }
}
```

## 获取电子邮件帐户
引入1.0
{: .label .label-purple }

#### 要求
```json
GET _plugins/_alerting/destinations/email_accounts/<email_account_id>
{
  "name": "example_account",
  "email": "example@email.com",
  "host": "smtp.email.com",
  "port": 465,
  "method": "ssl"
}
```
#### 示例响应
```json
{
  "_id" : "email_account_id",
  "_version" : 2,
  "_seq_no" : 8,
  "_primary_term" : 2,
  "email_account" : {
    "schema_version" : 2,
    "name" : "test_account",
    "email" : "test@email.com",
    "host" : "smtp.test.com",
    "port" : 465,
    "method" : "ssl"
  }
}
```

## 删除电子邮件帐户
引入1.0
{: .label .label-purple }

#### 要求
```
DELETE _plugins/_alerting/destinations/email_accounts/<email_account_id>
```
#### 示例响应

```json
{
  "_index" : ".opendistro-alerting-config",
  "_type" : "_doc",
  "_id" : "email_account_id",
  "_version" : 1,
  "result" : "deleted",
  "forced_refresh" : true,
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 12,
  "_primary_term" : 2
}
```

## 搜索电子邮件帐户
引入1.0
{: .label .label-purple }

#### 要求

```json
POST _plugins/_alerting/destinations/email_accounts/_search
{
  "from": 0,
  "size": 20,
  "sort": { "email_account.name.keyword": "desc" },
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      }
    }
  }
}
```

#### 示例响应

```json
{
  "took" : 8,
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
        "_index" : ".opendistro-alerting-config",
        "_type" : "_doc",
        "_id" : "email_account_id",
        "_seq_no" : 8,
        "_primary_term" : 2,
        "_score" : null,
        "_source" : {
          "schema_version" : 2,
          "name" : "example_account",
          "email" : "example@email.com",
          "host" : "smtp.email.com",
          "port" : 465,
          "method" : "ssl"
        },
        "sort" : [
          "example_account"
        ]
      },
      ...
    ]
  }
}
```

---

## Create email group
引入1.0
{: .label .label-purple }

#### Request

```json
发布_plugins/_ alerting/Destinations/email_groups
{
  "name"："example_email_group"，
  "emails"：[{{
    "email"："example@email.com"
  ]]
}
```

#### Example response

```json
{
  "_id" ："email_group_id"，
  "_version" ：1，
  "_seq_no" ：9，
  "_primary_term" ：2，
  "email_group" ：{
    "schema_version" ：2，
    "name" ："example_email_group"，
    "emails" ：[[
      {
        "email" ："example@email.com"
      }
    这是给出的
  }
}
```

## Update email group
引入1.0
{: .label .label-purple }

更新电子邮件组时，您可以选择包括`seq_no` 和`primary_term` 作为URL参数。如果这些数字不匹配现有的电子邮件组或电子邮件组，则警报插件会引发错误。OpenSearch会自动增加版本号和序列编号（请参阅示例响应）。

#### Request

```json
put _plugins/_alerting/destinations/email_groups/<Email_group_id>
{
  "name"："example_email_group"，
  "emails"：[{{
    "email"："example@email.com"
  ]]
}

put _plugins/_alerting/destinations/email_groups/<Email_group_id>？if_seq_no = 16＆if_primary_term = 2
{
  "name"："example_email_group"，
  "emails"：[{{
    "email"："example@email.com"
  ]]
}
```
#### Example response

```json
{
  "_id" ："email_group_id"，
  "_version" ：4，
  "_seq_no" ：17，
  "_primary_term" ：2，
  "email_group" ：{
    "schema_version" ：2，
    "name" ："example_email_group"，
    "emails" ：[[
      {
        "email" ："example@email.com"
      }
    这是给出的
  }
}
```

## Get email group
引入1.0
{: .label .label-purple }

#### Request
```json
获取_plugins/_alerting/destinations/email_groups/<Email_group_id>
{
  "name"："example_email_group"，
  "emails"：[{{
    "email"："example@email.com"
  ]]
}
```
#### Example response

```json
{
  "_id" ："email_group_id"，
  "_version" ：4，
  "_seq_no" ：17，
  "_primary_term" ：2，
  "email_group" ：{
    "schema_version" ：2，
    "name" ："example_email_group"，
    "emails" ：[[
      {
        "email" ："example@email.com"
      }
    这是给出的
  }
}
```

## Delete email group
引入1.0
{: .label .label-purple }

#### Request
```
delete _plugins/_alerting/destinations/email_groups/<Email_group_id>
```
#### Example response

```json
{
  "_index" ：".opendistro-alerting-config"，
  "_type" ："_doc"，
  "_id" ："email_group_id"，
  "_version" ：1，
  "result" ："deleted"，
  "forced_refresh" ： 真的，
  "_shards" ：{
    "total" ：2，
    "successful" ：2，
    "failed" ：0
  }，，
  "_seq_no" ：11，
  "_primary_term" ：2
}
```

## Search email group
引入1.0
{: .label .label-purple }

#### Request

```json
发布_plugins/_alerting/Destinations/email_groups/_search
{
  "from"：0，
  "size"：20，
  "sort"：{"email_group.name.keyword"："desc" }，，
  "query"：{
    "bool"：{
      "must"：{
        "match_all"：{}
      }
    }
  }
}
```

#### Example response

```json
{
  "took" ：7，
  "timed_out" ： 错误的，
  "_shards" ：{
    "total" ：1，
    "successful" ：1，
    "skipped" ：0，
    "failed" ：0
  }，，
  "hits" ：{
    "total" ：{
      "value" ：5，
      "relation" ："eq"
    }，，
    "max_score" ： 无效的，
    "hits" ：[[
      {
        "_index" ：".opendistro-alerting-config"，
        "_type" ："_doc"，
        "_id" ："email_group_id"，
        "_seq_no" ：10，
        "_primary_term" ：2，
        "_score" ： 无效的，
        "_source" ：{
          "schema_version" ：2，
          "name" ："example_email_group"，
          "emails" ：[[
            {
              "email" ："example@email.com"
            }
          这是给出的
        }，，
        "sort" ：[[
          "example_email_group"
        这是给出的
      }，，
      ...
    这是给出的
  }
}
```
---

