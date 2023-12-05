---
layout: default
title: 检测器API
parent: API工具
nav_order: 35
---

# 检测器API

以下API可用于与检测器相关的许多任务，从创建检测器到更新和搜索检测器。许多API调用使用请求中的检测器ID，可以使用[搜索检测器API](#search-detector-api)。

---
## Create Detector API

创建一个新的检测器。

```json
发布_plugins/_security_analytics/detector
```

### Request fields

您可以在创建检测器时指定以下字段。

场地| 类型| 描述
:--- | :--- |：--- |
`enabled` | 布尔| 将检测器设置为活动（true）或无效（false）。默认为`true` 创建新的检测器时。必需的。
`name` | 细绳| 检测器的名称。名称应仅由上下小写字母组成，数字0-9，连字符，空间和下划线。使用5至50个字符。必需的。
`detector_type` | 细绳| 定义检测器的日志类型。选项是`linux`，`network` ，`windows`，`ad_ldap`，`apache_access`，`cloudtrail`，`dns`， 和`s3`。必需的。
`schedule` | 目的| 确定检测器运行频率的时间表。有关在API中指定固定间隔的信息，请参见[cron表达式参考]（{{site.url}}} {{site.baseurl}}/Monitoring-插件/警报/cron/）。
`schedule.period` | 目的| 时间表频率的详细信息。
`schedule.period.interval` | 整数| 检测器运行的间隔。
`schedule.period.unit` | 细绳| 间隔的时间单位。
`inputs` | 目的| 检测器输入。
`inputs.detector_input` | 大批| 包含用于创建检测器的索引和定义的数组。当前，检测器仅允许一个日志数据源。
`inputs.detector_input.description` | 细绳| 检测器的描述。选修的。
`inputs.detector_input.custom_rules` | 大批| 定制规则的检测器输入。至少必须为检测器指定一个规则。可选如果前-指定包装规则。
`inputs.detector_input.custom_rules.id` | 细绳| 用户生成的自定义规则的有效规则ID。有效的规则是在全球或普遍唯一的标识符（UUID）上格式化的，请参见[普遍唯一的标识符]（https://en.wikipedia.org/wiki/wiki/universally_unique_istifier）。
`inputs.detector_input.indices` | 大批| 用于检测器的日志数据源，可以是索引名称或索引模式。目前，只有一个条目由计划在将来的版本中支持多个索引的计划。必需的。
`inputs.detector_input.pre_packaged_rules` | 大批| 探测器输入-包装规则（而不是自定义规则）。至少必须为检测器指定一个规则。如果指定自定义规则，可选。
`inputs.detector_input.pre_packaged_rules.id` | 细绳| PRE的规则ID-包装规则。请参阅[搜索pre-包装规则]（{{site.url}}} {{site.baseurl}}/security-分析/API-工具/规则-api/#搜索-pre-包装-规则）有关如何使用API搜索规则并在结果中获得规则ID的信息。
`triggers` | 大批| 触发警报设置。
`triggers.ids` | 大批| 成为触发条件一部分的规则ID列表。
`triggers.tags` | 大批| 标签在安全规则中指定。然后可以选择标签并将其应用于警报触发器，以聚焦警报的触发条件。请参阅Sigma [规则创建指南]（https://github.com/sigmahq/sigma/wiki/rule）中的Sigma规则中如何使用标签的示例。-创建-指导#标签）。
`triggers.id` | 细绳| 触发器的唯一ID。
`triggers.sev_levels` | 大批| Sigma规则严重程度：`informational`;`low`;`medium`;`high`;`criticial`。请参阅[level]（https://github.com/sigmahq/sigma/wiki/rule-创建-指导#级别）在《 Sigma规则创建指南》中。
`triggers.name` | 细绳| 触发器的名称。名称应仅由上下小写字母组成，数字0-9，连字符，空间和下划线。使用5至50个字符。必需的。
`triggers.severity` | 整数| 触发器表示为整数的严重程度：1 =最高；2 =高;3 =中间;4 =低;5 =最低。触发严重性是警报定义的一部分。
`triggers.actions` | 目的| 当满足触发条件时，操作发送通知。可选，作为通知消息，作为警报的一部分。
`triggers.actions.id` | 细绳| 动作的独特ID。用户生成。
`triggers.actions.destination_id` | 细绳| 通知目的地的唯一ID。用户生成。
`triggers.actions.subject_template` | 目的| 包含通知消息主题字段的信息。选修的。
`triggers.actions.subject_template.source` | 细绳| 通知消息的主题。
`triggers.actions.subject_template.lang` | 细绳| 用于定义主题的脚本语言。必须是胡子。有关模板的更多信息，请参见[小胡子手册]（https://mustache.github.io/mustache.5.html）。
`triggers.actions.name` | 细绳| 触发警报的名称。名称应仅由上下小写字母组成，数字0-9，连字符，空间和下划线。使用5至50个字符。
`triggers.actions.message_template` | 细绳| 包含通知消息正文的信息。选修的。
`triggers.actions.message_template.source` | 细绳| 通知消息的主体。
`triggers.actions.message_template.lang` | 细绳| 用于定义消息的脚本语言。必须是`Mustache`。
`triggers.actions.throttle_enabled` | 布尔| 启用节流通知。选修的。默认为`false`。
`triggers.actions.throttle` | 目的| 节流限制您在给定时间内收到的通知数量。
`triggers.actions.throttle.unit` | 细绳| 节流的时间单位。
`triggers.actions.throttle.value` | 整数| 时间单位的值。

### Example request

```json
发布_plugins/_security_analytics/detector
{
  "enabled"： 真的，
  "schedule"：{
    "period"：{
      "interval"：1，
      "unit"："MINUTES"
    }
  }，，
  "detector_type"："WINDOWS"，
  "type"："detector"，
  "inputs"：[[
    {
      "detector_input"：{
        "description"："windows detector for security analytics"，
        "custom_rules"：[[
          {
            "id"："bc2RB4QBrbtylUb_1Pbm"
          }
        ]，，
        "indices"：[[
          "windows"
        ]，，
        "pre_packaged_rules"：[[
          {
            "id"："06724a9a-52fc-11ed-bdc3-0242ac120002"
          }
        这是给出的
      }
    }
  ]，，
  "triggers"：[[
    {
      "ids"：[[
        "06724a9a-52fc-11ed-bdc3-0242ac120002"
      ]，，
      "types"：[]，，
      "tags"：[[
        "attack.defense_evasion"
      ]，，
      "severity"："1"，
      "actions"：[{{
          "id"："hVTLkZYzlA"，
          "destination_id"："6r8ZBoQBKW_6dKriacQb"，
          "subject_template"：{
            "source"："Trigger: {{ctx.trigger.name}}"，
            "lang"："mustache"
          }，，
          "name"："hello_world"，
          "throttle_enabled"： 错误的，
          "message_template"：{
            "source"："Detector {{ctx.detector.name}} just entered alert status. Please investigate the issue." +
            "- Trigger: {{ctx.trigger.name}}" +
            "- Severity: {{ctx.trigger.severity}}"，
            "lang"："mustache"
          }，，
          "throttle"：{
            "unit"："MINUTES"，
            "value"：108
          }
        }
      ]，，
      "id"："8qhrBoQBYK1JzUUDzH-N"，
      "sev_levels"：[]，，
      "name"："test-trigger"
    }
  ]，，
  "name"："nbReFCjlfn"
}
```
{% include copy-curl.html %}

### Example response

```json
{
    "_id"："dc2VB4QBrbtylUb_Hfa3"，
    "_version"：1，
    "detector"：{
        "name"："nbReFCjlfn"，
        "detector_type"："windows"，
        "enabled"： 真的，
        "schedule"：{
            "period"：{
                "interval"：1，
                "unit"："MINUTES"
            }
        }，，
        "inputs"：[[
            {
                "detector_input"：{
                    "description"："windows detector for security analytics"，
                    "indices"：[[
                        "windows"
                    ]，，
                    "custom_rules"：[[
                        {
                            "id"："bc2RB4QBrbtylUb_1Pbm"
                        }
                    ]，，
                    "pre_packaged_rules"：[[
                        {
                            "id"："06724a9a-52fc-11ed-bdc3-0242ac120002"
                        }
                    这是给出的
                }
            }
        ]，，
        "triggers"：[[
            {
                "id"："8qhrBoQBYK1JzUUDzH-N"，
                "name"："test-trigger"，
                "severity"："1"，
                "types"：[]，，
                "ids"：[[
                    "06724a9a-52fc-11ed-bdc3-0242ac120002"
                ]，，
                "sev_levels"：[]，，
                "tags"：[[
                    "attack.defense_evasion"
                ]，，
                "actions"：[[
                    {
                        "id"："hVTLkZYzlA"，
                        "name"："hello_world"，
                        "destination_id"："6r8ZBoQBKW_6dKriacQb"，
                        "message_template"：{
                            "source"："Trigger: {{ctx.trigger.name}}"，
                            "lang"："mustache"
                        }，，
                        "throttle_enabled"： 错误的，
                        "subject_template"：{
                            "source"："Detector {{ctx.detector.name}} just entered alert status. Please investigate the issue." +
                    "- Trigger: {{ctx.trigger.name}}" +
                    "- Severity: {{ctx.trigger.severity}}"，
                            "lang"："mustache"
                        }，，
                        "throttle"：{
                            "value"：108，
                            "unit"："MINUTES"
                        }
                    }
                这是给出的
            }
        ]，，
        "last_update_time"："2022-10-24T01:22:03.738379671Z"，
        "enabled_time"："2022-10-24T01:22:03.738376103Z"
    }
}
```

---
## 更新检测器API

更新检测器API可用于更新检测器定义。它要求检测器ID指定检测器。

```json
PUT /_plugins/_security_analytics/detectors/<detector_Id>
```

### 请求字段

更新检测器时，您可以指定以下字段。

场地| 类型| 描述
:--- | :--- |：--- |
`detector_type` | 细绳| 定义检测器的日志类型。选项是`linux`，`network` ，`windows`，`ad_ldap`，`apache_access`，`cloudtrail`，`dns`， 和`s3`。
`name` | 细绳| 检测器的名称。名称应仅由上下小写字母组成，数字0-9，连字符，空间和下划线。使用5至50个字符。必需的。
`enabled` | 布尔| 将检测器设置为活动（true）或无效（false）。
`schedule.period.interval` | 整数| 检测器运行的间隔。
`schedule.period.unit` | 细绳| 间隔的时间单位。
`inputs.input.description` | 细绳| 检测器的描述。
`inputs.input.indices` | 大批| 用于检测器的日志数据源。此时仅允许一个来源。
`inputs.input.rules.id` | 大批| 检测器定义的安全规则列表。
`triggers.sev_levels` | 大批| Sigma规则严重程度：`informational`;`low`;`medium`;`high`;`criticial`。看[等级](https://github.com/SigmaHQ/sigma/wiki/Rule-Creation-Guide#level) 在Sigma规则创建指南中。
`triggers.tags` | 大批| 标签在安全规则中指定。然后可以选择标签并将其应用于警报触发器，以聚焦警报的触发条件。请参阅一个示例，说明在Sigma的Sigma规则中如何使用标签[规则创建指南](https://github.com/SigmaHQ/sigma/wiki/Rule-Creation-Guide#tags)。
`triggers.actions` | 目的| 当满足触发条件时，操作发送通知。请参阅触发操作[创建检测器API]({{site.url}}{{site.baseurl}}/security-analytics/api-tools/detector-api/#create-detector-api)。


### 示例请求

```json
PUT /_plugins/_security_analytics/detectors/J1RX1IMByX0LvTiGTddR
{
  "type": "detector",
  "detector_type": "windows",
  "name": "windows_detector",
  "enabled": true,
  "createdBy": "chip",
  "schedule": {
    "period": {
      "interval": 1,
      "unit": "MINUTES"
    }
  },
  "inputs": [
    {
      "input": {
        "description": "windows detector for security analytics",
        "indices": [
          "windows"
        ],
        "custom_rules": [],
        "pre_packaged_rules": [
          {
            "id": "73a883d0-0348-4be4-a8d8-51031c2564f8"
          },
          {
            "id": "1a4bd6e3-4c6e-405d-a9a3-53a116e341d4"
          }
        ]
      }
    }
  ],
  "triggers": [
    {
      "sev_levels": [],
      "tags": [],
      "actions": [],
      "types": [
        "windows"
      ],
      "name": "test-trigger",
      "id": "fyAy1IMBK2A1DZyOuW_b"
    }
  ]
}
```
{% include copy-curl.html %}

### 示例响应

```json
{
    "_id": "J1RX1IMByX0LvTiGTddR",
    "_version": 1,
    "detector": {
        "name": "windows_detector",
        "detector_type": "windows",
        "enabled": true,
        "schedule": {
            "period": {
                "interval": 1,
                "unit": "MINUTES"
            }
        },
        "inputs": [
            {
                "detector_input": {
                    "description": "windows detector for security analytics",
                    "indices": [
                        "windows"
                    ],
                    "rules": [
                        {
                            "id": "LFRY1IMByX0LvTiGZtfh"
                        }
                    ]
                }
            }
        ],
        "triggers": [],
        "last_update_time": "2022-10-14T02:36:32.909581688Z",
        "enabled_time": "2022-10-14T02:33:34.197Z"
    }
}
```

#### 响应字段

场地| 类型| 描述
:--- | :--- |：--- |
`_version` | 细绳| 更新的版本号。
`detector.last_update_time` | 细绳| 最后更新的日期和时间。
`detector.enabled_time` | 细绳| 最后启用检测器的日期和时间。

---
## Delete Detector API

该API使用检测器ID来指定和删除检测器。

### Path and HTTP methods

```json
delete/_plugins/_security_analytics/detectors/ijaxz4qbrmvplm4jyxx_
```

### Example request

```json
delete/_plugins/_security_analytics/detectors/<检测器ID>
```
{% include copy-curl.html %}

### Example response

```json
{
  "_id" ："IJAXz4QBrmVplM4JYxx_"，
  "_version" ：1
}
```

---
## 获取检测器API

GET探测器API检索检测器的详细信息。使用呼叫中的检测器ID获取检测器详细信息。

### 路径和HTTP方法

```json
GET /_plugins/_security_analytics/detectors/x-dwFIYBT6_n8WeuQjo4
```

### 示例请求

```json
GET /_plugins/_security_analytics/detectors/<detector Id>
```
{% include copy-curl.html %}

### 示例响应

```json
{
  "_id" : "x-dwFIYBT6_n8WeuQjo4",
  "_version" : 1,
  "detector" : {
    "name" : "DetectorTest1",
    "detector_type" : "windows",
    "enabled" : true,
    "schedule" : {
      "period" : {
        "interval" : 1,
        "unit" : "MINUTES"
      }
    },
    "inputs" : [
      {
        "detector_input" : {
          "description" : "Test and delete",
          "indices" : [
            "windows1"
          ],
          "custom_rules" : [ ],
          "pre_packaged_rules" : [
            {
              "id" : "847def9e-924d-4e90-b7c4-5f581395a2b4"
            }
          ]
        }
      }
    ],
    "last_update_time" : "2023-02-02T23:22:26.454Z",
    "enabled_time" : "2023-02-02T23:22:26.454Z"
  }
}
```

---
## Search Detector API

搜索探测器API通过检测器ID，检测器名称或检测器类型搜索检测器匹配。

### Request fields

场地| 类型| 描述
:--- | :--- |：--- |
`_id` | 细绳| 更新的版本号。
`detector.name` | 细绳| 检测器的名称。
`detector_type` | 细绳| 检测器的日志类型。选项是`linux`，`network` ，`windows`，`ad_ldap`，`apache_access`，`cloudtrail`，`dns`， 和`s3`。

### Example request

**检测器ID**
```json
post/_plugins/_security_analytics/detectors/_search
{
    "query"：{
        "match"：{
            "_id"："MFRg1IMByX0LvTiGHtcN"
        }
    }
}
```
{% include copy-curl.html %}

**检测器名称**
```json
post/_plugins/_security_analytics/detectors/_search
{
  "size"：30，
  "query"：{
    "nested"：{
      "path"："detector"，
      "query"：{
        "bool"：{
          "must"：[[
            {"match"：{"detector.name"："DetectorTest1"}}}
          这是给出的
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

### Example response

```json
{
  "took" ：0，
  "timed_out" ： 错误的，
  "_shards" ：{
    "total" ：1，
    "successful" ：1，
    "skipped" ：0，
    "failed" ：0
  }，，
  "hits" ：{
    "total" ：{
      "value" ：1，
      "relation" ："eq"
    }，，
    "max_score" ：3.671739，
    "hits" ：[[
      {
        "_index" ：".opensearch-sap-detectors-config"，
        "_id" ："x-dwFIYBT6_n8WeuQjo4"，
        "_version" ：1，
        "_seq_no" ：76，
        "_primary_term" ：17，
        "_score" ：3.671739，
        "_source" ：{
          "type" ："detector"，
          "name" ："DetectorTest1"，
          "detector_type" ："windows"，
          "enabled" ： 真的，
          "enabled_time" ：1675380146454，
          "schedule" ：{
            "period" ：{
              "interval" ：1，
              "unit" ："MINUTES"
            }
          }，，
          "inputs" ：[[
            {
              "detector_input" ：{
                "description" ："Test and delete"，
                "indices" ：[[
                  "windows1"
                ]，，
                "custom_rules" ：[]，，
                "pre_packaged_rules" ：[[
                  {
                    "id" ："847def9e-924d-4e90-b7c4-5f581395a2b4"
                  }
                这是给出的
              }
            }
          ]，，
          "triggers" ：[[
            {
              "id" ："w-dwFIYBT6_n8WeuQToW"，
              "name" ："trigger 1"，
              "severity" ："1"，
              "types" ：[[
                "windows"
              ]，，
              "ids" ：[[
                "847def9e-924d-4e90-b7c4-5f581395a2b4"
              ]，，
              "sev_levels" ：[[
                "critical"
              ]，，
              "tags" ：[[
                "attack.t1003.002"
              ]，，
              "actions" ：[[
                {
                  "id" ：""，
                  "name" ："Triggered alert condition:  - Severity: 1 (Highest) - Threat detector: DetectorTest1"，
                  "destination_id" ：""，
                  "message_template" ：{
                    "source" ："""Triggered alert condition: 
Severity: 1 (Highest)
Threat detector: DetectorTest1
Description: Test and delete
Detector data sources:
  windows1"""，
                    "lang" ："mustache"
                  }，，
                  "throttle_enabled" ： 错误的，
                  "subject_template" ：{
                    "source" ："Triggered alert condition:  - Severity: 1 (Highest) - Threat detector: DetectorTest1"，
                    "lang" ："mustache"
                  }，，
                  "throttle" ：{
                    "value" ：10，
                    "unit" ："MINUTES"
                  }
                }
              这是给出的
            }
          ]，，
          "last_update_time" ：1675380146454，
          "monitor_id" ：[[
            "xOdwFIYBT6_n8WeuQToa"
          ]，，
          "bucket_monitor_id_rule_id" ：{
            "-1" ："xOdwFIYBT6_n8WeuQToa"
          }，，
          "rule_topic_index" ：".opensearch-sap-windows-detectors-queries"，
          "alert_index" ：".opensearch-sap-windows-alerts"，
          "alert_history_index" ：".opensearch-sap-windows-alerts-history"，
          "alert_history_index_pattern" ："<.opensearch-sap-windows-alerts-history-{now/d}-1>"，
          "findings_index" ：".opensearch-sap-windows-findings"，
          "findings_index_pattern" ："<.opensearch-sap-windows-findings-{now/d}-1>"
        }
      }
    这是给出的
  }
}
```


