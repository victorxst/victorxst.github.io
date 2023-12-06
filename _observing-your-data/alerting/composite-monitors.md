---
layout: default
title: 复合监视器
nav_order: 25
parent: 监视器
grand_parent: 警报
has_children: false
redirect_from:
 - /observing-your-data/alerting/composite-monitors/
---

# 复合监视器

---

<details closed markdown="block">
  <summary>
    目录
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

---

## 关于复合监视器

基本的[监视类型]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/monitors/#monitor-types) 对于警报插件，旨在定义单个触发类型。例如，每个文档监视器可以根据查询与文档的匹配触发警报，而每个存储桶监视器可以根据针对数据源中汇总值的查询触发警报。复合监视器将多个监视器组合成一个序列，以根据多个标准分析数据源，然后使用其单独的警报生成单个链式警报。这使您可以获取有关数据源的更多详细信息，并且不需要您手动协调单独的监视器的调度。

复合监视器以以下方式删除基本监视器的局限性：

*复合监视器使您能够通过由多种类型的显示器生成的触发器组合来创建复杂的查询。
*他们有能力定义作为单个执行的规则和查询管道。
*他们向用户提供了一个警报，而不是从工作流程中的单个监视器的多个警报。
*他们通过按顺序运行多个监视器和多种类型的监视器来更完整地查看给定数据源，从而创造了更集中的结果并减少结果中的噪声。


## 关键术语

下表中的关键术语描述了复合监视器的基本概念。有关所有类型的监视器常见的其他术语，请参见[关键术语]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/index/#key-terms) 在警报部分。

| 学期| 定义|
| :--- | :--- |
| 复合监视器| 复合监视器是一种支持连续工作流中多个监视器的执行的监视器。它支持配置触发器来创建链式警报。|
| 委托监视器| 委托监视器按照复合监视器的定义在其订单中依次执行。当满足委托监视器的触发条件时，它会生成审核警报。然后，此审核警报成为复合监视器触发器的条件。复合监视器支持每个查询，每个存储桶和每个文档监视器作为代表监视器。|
| 工作流ID| 工作流ID为所有代表监视器的整个工作流提供了标识符。它是复合监视器的显示器ID的代名词。|
| 连锁警报| 当委托监视器生成审计警报时，由复合监视器触发器生成链式警报。链式警报触发条件支持逻辑运算符的使用`AND`，`OR`， 和`NOT` 因此，您可以将多个功能组合为单个表达式。|
| 审核警报| 委托监视器在**审计** 状态。未通知用户每个单独的审核警报，也不需要确认他们。审核警报用于评估复合监视器中的链式警报触发条件。|
| 执行| 在复合监视器的配置中定义的序列中的所有委托监视器的一次运行。|

## 基本工作流程

您可以通过在工作流程中组合单个监视器来创建复合监视器，该工作流程以定义的序列执行每个监视器。当个人审计警报来自代表监视器时，符合复合监视器的触发条件时，复合监视器会生成自己的链式警报。考虑以下事件序列，以了解如何使用两个代表监视的简单复合监视器执行其工作流程。在此示例中，当第一个监视器和第二个监视器都生成警报时，请满足复合监视器的触发条件。

1. 复合监视器启动执行并将其委派给第一个监视器。满足第一个监视器的触发条件，并生成审核警报。
1. 然后，复合监视器将执行委托给第二监视器。还满足了第二个监视器的触发条件，并生成了自己的审核警报。
1. 由于复合监视器的触发条件要求第一和第二监视都会生成审核警报，因此复合监视器随后触发链式警报。
1. 如果在复合监视器的定义中配置了通知，则用户会收到有关链式警报的通知。但是，他们没有收到由两个代表监视器产生的个人审计警报。

在这个简单的示例中，第一个监视器可以是使用三个不同查询来分析数据源的每个文档监视器，而第二个监视器可以是按客户端IP汇总数据的每个存储桶监视器。通过组合每个委托监视器的要求，Composite Monitor聚焦了决定是否生成警报的标准。这可以提高警报的有意义，同时删除没有提供确定性价值的外部警报。


## 使用API管理复合监视器

您可以使用OpenSearch REST API或[OpenSearch仪表板](#creating-composite-monitors-in-opensearch-dashboards)。本节介绍了复合监视器的API功能。

### 创建复合监视器

此API允许您创建一个复合监视器。

```json
POST _plugins/_alerting/workflows
```
{% include copy-curl.html %}}

#### 请求字段

| 场地| 类型| 描述|
| :--- | :--- | :--- |
| `schedule` | 目的| 确定执行频率运行的时间表。|
| `schedule.period.interval` | 数字| 接受一个数值值以设置执行频率。|
| `schedule.period.unit` | 目的| 间隔的度量时间单位：`SECONDS`，`MINUTES`，`HOURS`，`DAYS`。|
| `inputs` | 目的| 接受输入以定义委托监视器，该监视器在执行顺序中指定了委托监视器及其顺序。|
| `inputs.composite_input.sequence.delegates` | 目的| 单个监视器的设置，该设置是复合监视器的基础。|
| `inputs.composite_input.sequence.delegates.order` | 数字| 指定监视器在执行中运行的顺序。|
| `inputs.composite_input.sequence.delegates.monitor_id` | 细绳| 监视器的唯一标识符。|
| `enabled_time` | 数字| 启用监视器的时间。以时代为单位。|
| `enabled` | 布尔| 确定复合监视器是否已启用的设置。设置为`true` 启用复合监视器。默认为`true`。|
| `workflow_type` | 细绳| 设置`composite` 用于复合监视器。|
| `triggers` | 目的| 单个警报触发器的详细信息。|
| `triggers.chained_alert_trigger` | 目的| 每个单独的警报触发器的详细信息。每个监视器的警报触发器将需要设置其配置。|
| `triggers.chained_alert_trigger.id` | 细绳| 警报触发器的唯一标识符。|
| `triggers.chained_alert_trigger.name` | 细绳| 警报触发器的名称。|
| `triggers.chained_alert_trigger.severity` | 数字| 警报的严重性。1 =最高;2 =高;3 =中间;4 =低;5 =最低。|
| `triggers.chained_alert_trigger.condition.script` | 目的| 脚本详细信息，这些详细信息确定了触发警报的条件。|
| `triggers.chained_alert_trigger.condition.script.source` | 细绳| 无痛脚本定义了触发警报的条件。|
| `triggers.chained_alert_trigger.condition.script.lang` | 细绳| 进入`painless` 对于无痛的脚本语言。|
| `actions` | 目的| 提供用于配置警报通知的字段。|

#### 示例请求

```json
POST _plugins/_alerting/workflows
{
	"last_update_time": 1679468231835,
	"owner": "alerting",
	"type": "workflow",
	"schedule": {
		"period": {
			"interval": 1,
			"unit": "MINUTES"
		}
	},
	"inputs": [{
		"composite_input": {
			"sequence": {
				"delegates": [{
						"order": 1,
						"monitor_id": "grsbCIcBvEHfkjWFeCqb"
					},
					{
						"order": 2,
						"monitor_id": "agasbCIcBvEHfkjWFeCqa"
					}
				]
			}
		}
	}],
	"enabled_time": 1679468231835,
	"enabled": true,
	"workflow_type": "composite",
	"name": "scale_up",
	"triggers": [{
			"chained_alert_trigger": {
				"id": "m1ANDm2",
				"name": "jnkjn",
				"severity": "1",
				"condition": {
					"script": {
						"source": "(monitor[id=grsbCIcBvEHfkjWFeCqb] && monitor[id=agasbCIcBvEHfkjWFeCqa])",
						"lang": "painless"
					}
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
		},
		{
			"chained_alert_trigger": {
				"id": "m1ORm2",
				"name": "jnkjn",
				"severity": "1",
				"condition": {
					"script": {
						"source": "(monitor[id=grsbCIcBvEHfkjWFeCqb] || monitor[id=agasbCIcBvEHfkjWFeCqa])",
						"lang": "painless"
					}
				}
			}
		}
	]
}
```
{% include copy-curl.html %}}

#### 使用无痛的脚本语言来定义链接的警报触发条件

复合监视器配置采用无痛的脚本语言来定义生成链式警报的条件。为复合监视器的每个执行应用条件。您在`triggers.chained_alert_triggers.condition.script.source` 请求字段。使用无痛语法，您可以将逻辑应用于与基本布尔操作员的监视器之间的链接，或者，或者，不及优先：

*和=`&&`
*或=`||`
*不=`!`
*先例=`()`

请参阅以下示例，以了解监视器定义中的每个示例。

***示例1**
   
   `monitor[id=1] && monitor[id=2]`
   
   当两个监视器时，代表监视器的以下条件将触发复合监视#1和监视#2生成警报。

***示例2**
   
   `monitor[id=1] || !monitor[id=2]`

   以下条件将触发复合监视器以产生链式警报。#1生成警报或监视#2不会产生警报。

***示例3**
   
   `monitor[id=1] && (monitor[id=2] || monitor[id=3])`

   以下条件将触发复合监视器以在监视器时产生链式警报#1生成警报并监视#2或监视#3生成警报。
   
无痛脚本中的监视器ID顺序不能定义监视器的执行顺序。监视执行序列在`inputs.composite_input.sequence.delegates.order` 在请求中字段。
{:.note}


### 获取复合监视器

此API在指定的监视器上检索信息。

```json
GET _plugins/_alerting/workflows/<workflow_id>
```
{% include copy-curl.html %}}

#### 路径参数

| 场地| 类型| 描述|
| :--- | :--- | :--- |
| `workflow_id` | 细绳| 复合监视器的[工作流ID](#key-terms)。|


### 更新复合监视器

此API更新了复合监视器的详细信息。看[创建复合监视器](#create-composite-monitor) 有关请求字段的描述。

#### 示例请求

```json
PUT _plugins/_alerting/workflows/<workflow_id>
{
    "owner": "security_analytics",
    "type": "workflow",
    "schedule": {
        "period": {
        "interval": 1,
        "unit": "MINUTES"
        }
    },
    "inputs": [
        {
            "composite_input": {
                "sequence": {
                    "delegates": [
                        {
                            "order": 1,
                            "monitor_id": "grsbCIcBvEHfkjWFeCqb"
                        },
                        {
                            "order": 2,
                            "monitor_id": "agasbCIcBvEHfkjWFeCqa"
                        }
                    ]
                }
            }
        }
    ],
    "enabled_time": 1679468231835,
    "enabled": true,
    "workflow_type": "composite",
    "name": "NTxdwApKbv"
}
```
{% include copy-curl.html %}}


### 删除复合监视器

```json
DELETE _plugins/_alerting/workflows/<workflow_id>
```
{% include copy-curl.html %}}


### 执行复合监视器

此API开始为复合监视器的工作流执行：

```json
POST /_plugins/_alerting/workflows/<workflow_id>/_execute
```
{% include copy-curl.html %}}

#### 示例响应

```json
{
    "execution_id": "I0GXeIgBYKBG2nHoiHCL_2023-06-01T20:18:48.511884_a9c1d055-9b70-49c2-b32a-716cff1f562e",
    "workflow_name": "scale_up",
    "workflow_id": "I0GXeIgBYKBG2nHoiHCL",
    "trigger_results": {
        "m1ANDm2": {
            "name": "jnkjn",
            "triggered": true,
            "action_results": {},
            "error": null
        },
        "m1ORm2": {
            "name": "jnkjn",
            "triggered": true,
            "action_results": {},
            "error": null
        }
    },
    "monitor_run_results": [{
            "monitor_name": "test triggers",
            "period_start": 1685650668501,
            "period_end": 1685650728501,
            "error": null,
            "input_results": {
                "results": [{
                    "bhjh": [
                        "OkGceIgBYKBG2nHoyHAn|test1",
                        "O0GceIgBYKBG2nHozHCW|test1"
                    ],
                    "nkjkj": [
                        "OkGceIgBYKBG2nHoyHAn|test1",
                        "O0GceIgBYKBG2nHozHCW|test1"
                    ],
                    "jknkjn": [
                        "OkGceIgBYKBG2nHoyHAn|test1",
                        "O0GceIgBYKBG2nHozHCW|test1"
                    ]
                }],
                "error": null
            },
            "trigger_results": {
                "NC3Dd4cBCDCIfBYtViLI": {
                    "name": "njkkj",
                    "triggeredDocs": [
                        "OkGceIgBYKBG2nHoyHAn|test1",
                        "O0GceIgBYKBG2nHozHCW|test1"
                    ],
                    "action_results": {},
                    "error": null
                }
            }
        },
        {
            "monitor_name": "test triggers 2",
            "period_start": 1685650668501,
            "period_end": 1685650728501,
            "error": null,
            "input_results": {
                "results": [{
                    "bhjh": [
                        "PEGceIgBYKBG2nHo1HCw|test",
                        "PUGceIgBYKBG2nHo3HA8|test"
                    ],
                    "nkjkj": [
                        "PEGceIgBYKBG2nHo1HCw|test",
                        "PUGceIgBYKBG2nHo3HA8|test"
                    ],
                    "jknkjn": [
                        "PEGceIgBYKBG2nHo1HCw|test",
                        "PUGceIgBYKBG2nHo3HA8|test"
                    ]
                }],
                "error": null
            },
            "trigger_results": {
                "NC3Dd4cBCDCIfBYtViLI": {
                    "name": "njkkj",
                    "triggeredDocs": [
                        "PEGceIgBYKBG2nHo1HCw|test",
                        "PUGceIgBYKBG2nHo3HA8|test"
                    ],
                    "action_results": {},
                    "error": null
                }
            }
        }
    ],
    "execution_start_time": "2023-06-01T20:18:48.511874Z",
    "execution_end_time": "2023-06-01T20:18:53.682405Z",
    "error": null
}
```


### 获得链接警报

该API返回复合监视器工作流中生成的一系列链接警报：

```json
GET /_plugins/_alerting/workflows/alerts?workflowIds=<workflow_ids>&getAssociatedAlerts=true
```

#### 查询参数

| 场地| 类型| 必需的| 描述|
| :--- | :--- | :--- | :--- |
| `workflowIds` | 大批| 不| 使用此参数时，它将返回指定工作流创建的警报。|
| `getAssociatedAlerts` | 布尔| 不| 什么时候`true`，响应返回审计警报，该警报使用用于创建链式警报的复合监视器。默认为`false`。|


#### 示例响应

```json
{
    "alerts": [
        {
            "id": "PbQoZokBfd2ci_FqMGi6",
            "version": 1,
            "monitor_id": "",
            "workflow_id": "G7QoZokBfd2ci_FqD2iZ",
            "workflow_name": "scale_up",
            "associated_alert_ids": [
                "4e8256c5-529a-484c-bf7b-d3980c03e9a4",
                "513a8cb3-44bc-4eee-8aac-131be10b399e"
            ],
            "schema_version": -1,
            "monitor_version": -1,
            "monitor_name": "",
            "execution_id": "G7QoZokBfd2ci_FqD2iZ_2023-07-17T23:20:55.244970_edd977d2-c02b-4cbe-8a79-2aa7991c4191",
            "trigger_id": "m1ANDm2",
            "trigger_name": "jnkjn",
            "finding_ids": [],
            "related_doc_ids": [],
            "state": "ACTIVE",
            "error_message": null,
            "alert_history": [],
            "severity": "1",
            "action_execution_results": [],
            "start_time": 1689636057269,
            "last_notification_time": 1689636057270,
            "end_time": null,
            "acknowledged_time": null
        },
        {
            "id": "PrQoZokBfd2ci_FqMGj8",
            "version": 1,
            "monitor_id": "",
            "workflow_id": "G7QoZokBfd2ci_FqD2iZ",
            "workflow_name": "scale_up",
            "associated_alert_ids": [
                "4e8256c5-529a-484c-bf7b-d3980c03e9a4",
                "513a8cb3-44bc-4eee-8aac-131be10b399e"
            ],
            "schema_version": -1,
            "monitor_version": -1,
            "monitor_name": "",
            "execution_id": "G7QoZokBfd2ci_FqD2iZ_2023-07-17T23:20:55.244970_edd977d2-c02b-4cbe-8a79-2aa7991c4191",
            "trigger_id": "m1ORm2",
            "trigger_name": "jnkjn",
            "finding_ids": [],
            "related_doc_ids": [],
            "state": "ACTIVE",
            "error_message": null,
            "alert_history": [],
            "severity": "1",
            "action_execution_results": [],
            "start_time": 1689636057340,
            "last_notification_time": 1689636057340,
            "end_time": null,
            "acknowledged_time": null
        }
    ],
    "associatedAlerts": [
        {
            "id": "4e8256c5-529a-484c-bf7b-d3980c03e9a4",
            "version": -1,
            "monitor_id": "DrQoZokBfd2ci_FqCWh8",
            "workflow_id": "G7QoZokBfd2ci_FqD2iZ",
            "workflow_name": "",
            "associated_alert_ids": [],
            "schema_version": 5,
            "monitor_version": 1,
            "monitor_name": "test triggers",
            "execution_id": "G7QoZokBfd2ci_FqD2iZ_2023-07-17T23:20:55.244970_edd977d2-c02b-4cbe-8a79-2aa7991c4191",
            "trigger_id": "NC3Dd4cBCDCIfBYtViLI",
            "trigger_name": "njkkj",
            "finding_ids": [
                "277afca7-d5aa-46ed-8023-5449ece65d36"
            ],
            "related_doc_ids": [
                "H7QoZokBfd2ci_FqFmii|test1"
            ],
            "state": "AUDIT",
            "error_message": null,
            "alert_history": [],
            "severity": "1",
            "action_execution_results": [],
            "start_time": 1689636056410,
            "last_notification_time": 1689636056410,
            "end_time": null,
            "acknowledged_time": null
        },
        {
            "id": "513a8cb3-44bc-4eee-8aac-131be10b399e",
            "version": -1,
            "monitor_id": "EbQoZokBfd2ci_FqCmiR",
            "workflow_id": "G7QoZokBfd2ci_FqD2iZ",
            "workflow_name": "",
            "associated_alert_ids": [],
            "schema_version": 5,
            "monitor_version": 1,
            "monitor_name": "test triggers 2",
            "execution_id": "G7QoZokBfd2ci_FqD2iZ_2023-07-17T23:20:55.244970_edd977d2-c02b-4cbe-8a79-2aa7991c4191",
            "trigger_id": "NC3Dd4cBCDCIfBYtViLI",
            "trigger_name": "njkkj",
            "finding_ids": [
                "6d185585-a077-4dde-8e43-b4c01b9f3102"
            ],
            "related_doc_ids": [
                "ILQoZokBfd2ci_FqGmhb|test"
            ],
            "state": "AUDIT",
            "error_message": null,
            "alert_history": [],
            "severity": "1",
            "action_execution_results": [],
            "start_time": 1689636056943,
            "last_notification_time": 1689636056943,
            "end_time": null,
            "acknowledged_time": null
        }
    ],
    "totalAlerts": 2
}
```

#### 请求字段

| 场地| 类型| 描述|
| :--- | :--- | :--- |
| `alerts` | 大批| 复合监视器生成的链式警报列表。|
| `associatedAlerts` | 大批| 代表监视器生成的审核警报列表。|


### 承认链接的警报

[收到警报后](#get-chained-alerts)，您可以在一个呼叫中确认多个主动警报。如果警报已经处于错误，已完成或已确认状态，则将显示在失败的数组中。

```json
POST _plugins/_alerting/workflows/<workflow_id>/_acknowledge/alerts
{
    "alerts": ["eQURa3gBKo1jAh6qUo49"]
}
```
{% include copy-curl.html %}}

#### 请求字段

| 场地| 类型| 描述|
| :--- | :--- | :--- |
| `alerts` | 大批| ID警报列表。结果包括系统确认的警报以及系统未识别的警报。|

#### 示例响应

```json
{
    "success": [
    "eQURa3gBKo1jAh6qUo49"
    ],
    "failed": []
}
```

## 在OpenSearch仪表板中创建复合监视器

首先导航到**创建监视器** OpenSearch仪表板中的页面：**警报>显示器** 并选择**创建监视器**。给显示器一个名称，然后选择**复合监视器** 作为监视器类型。创建复合监视器工作流和触发条件的步骤因您是否使用**视觉编辑器** 或者**提取查询编辑器**。第一个提供了用于定义复合监视器的基本UI选择器，而第二个则允许您使用脚本构建工作流和触发条件。确定使用哪种方法后，请参阅相应的部分。

### 视觉编辑器

要使用视觉编辑器来定义工作流程和触发条件，请选择**视觉编辑器** 广播按钮**监视定义方法** 部分。这在下面的图像中显示。

<img src="{{site.url}}{{site.baseurl}}/images/alerting/vis-editor.png" alt="Selecting the Visual editor" width="50%">

要在视觉编辑器中完成创建复合监视器，请按照以下步骤：

1. 在里面**频率** 下拉列表，选择一个**间隔**，**日常的**，**每周**，**每月**， 或者**自定义cron表达式**：
  ***间隔**  - 允许您根据指定的分钟，小时数或天数重复运行时间表。
  ***日常的**  - 指定一天中的时间和时区。
  ***每周**  - 指定一周中的一天，一天的时间和时区。
  ***每月**  - 指定一个月的一天，一天的时间和时区。
  ***自定义cron表达式**  - 为时间表创建自定义CRON表达式。使用**克朗表达** 链接以创建这些表达方式，或查看[cron表达参考]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/cron/)。

1. 在里面**代表监视器** 部分，通过在下拉列表中选择要在工作流中包含的单个监视器。在里面**视觉编辑器**，选择监视器的顺序确定其在工作流程中的顺序。
  
   选择**添加另一个显示器** 添加另一个下拉列表。至少需要两个代表监视器，总共允许最多10个。请记住，复合监测每个查询的支持，每个存储桶和每个文档监视器作为代表监视器。
   
   在每个下拉列表旁边，您可以选择“视图监视器”图标（{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/view-monitor-icon.png" class ="inline-icon" alt ="view monitor icon"/> {:/}）打开监视器的详细信息窗口并查看有关它的信息。
   
1. 为复合监视器定义触发器或触发器。在里面**触发器** 部分，选择**添加触发器**。添加触发名称，然后定义触发条件。
    * 使用**选择委托监视器** 标签打开流行音乐-上图中显示的向上窗口。
    
    <img src ="{{site.url}}{{site.baseurl}}/images/alerting/trigger1.png" alt ="This pop-up window shows options for selecting a delegate monitor and trigger condition operator" width="50%">
    
    * 使用**选择委托监视器** 下拉列表以从上一步中定义的那些选择委托监视器。对于第一个委托监视器，如果愿意，您可以选择不作为操作员选择。在现场填充监视器之后，您可以使用垃圾桶图标（{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/alerting/trash-can-icon.png" class ="inline-icon" alt ="trash can icon"/> {:/}）在列表的右侧，以在需要时删除监视器。
    *选择加号（{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/alerting/plus-sign-icon.png" class ="inline-icon" alt ="plus sign"/> {:/}）在第一个监视器的右侧选择第二个委托监视器。选择第二个显示器后，选择一个操作员`AND`，`OR`，`AND NOT`， 或者`OR NOT` 在两个监视器之间应用条件。应用操作员后，您可以选择操作员打开POP-再次向上窗口并更改选择。
    *选择警报的严重性级别。选项包括**1（最高）**，**2（高）**，**3（媒介）**，**4（低）**， 和**5（最低）**。
    * 在里面**通知** 部分，从下拉列表中选择一个通知频道。如果没有渠道，请选择**管理频道** 在下拉列表的右侧标记以设置通知频道。有关通知的更多信息，请参阅[通知]({{site.url}}{{site.baseurl}}/observing-your-data/notifications/index/) 文档。您也可以选择**添加通知** 为警报触发器指定其他通知。
      
      通知对于所有监视器类型都是可选的。
      {:.note}

    *要定义额外的触发器，请选择**添加另一个触发器**。您最多可以有10个触发器。选择**删除扳机** 在屏幕的右侧以卸下扳机。
    
1. 完成监视器工作流并定义触发器后，选择**创造** 在较低-屏幕的右角。创建了复合监视器，并打开了监视器的详细信息窗口。

### 提取查询编辑器

要使用提取查询编辑器来定义工作流程和触发器，请选择**提取查询编辑器** 广播按钮**监视定义方法** 部分。这在下面的图像中显示。

<img src="{{site.url}}{{site.baseurl}}/images/alerting/extract-q-editor.png" alt="Selecting the Extraction query editor" width="50%">

提取查询编辑器遵循与Visual Editor相同的一般步骤，但是它允许您使用API查询中的提取物构建复合监视器工作流和警报触发器。这为您提供了创建视觉编辑器不支持的更高级配置的能力。以下各节提供了这两个字段中每个字段的内容示例。复合监视器创建的所有其他步骤都与视觉编辑器相同。

***定义工作流程**
  
  在里面**定义工作流程** 字段，输入一个定义委托监视器及其在工作流中的顺序的序列。下面的示例显示了工作流中包含的代表监视器以及其顺序中的顺序：

  ```json
  {
      "sequence": {
          "delegates": [
              {
                  "order": 1,
                  "monitor_id": "0TgBZokB2ZtsLaRvXz70"
              },
              {
                  "order": 2,
                  "monitor_id": "8jgBZokB2ZtsLaRv6z4N"
              }
          ]
      }
  }
  ```
  
  All delegate monitors included in the workflow require a `Monitor_ID` and a value for `命令`。
  
***触发条件**
  
  在里面**触发条件** 字段，输入监视器和将用于定义它们之间条件的操作员。该领域要求以无痛的脚本语言进行触发条件的格式。要查看如何为触发条件形成这些脚本，请参见[使用无痛的脚本来定义链接的警报触发条件](#using-painless-scripting-language-to-define-chained-alert-trigger-conditions)。

  下面的示例显示了需要第一个监视器或第二个监视器生成审核警报的触发条件，然后复合监视器才能生成链式警报：

  ```painless
  (monitor[id=8d36S4kB0DWOHH7wpkET] || monitor[id=4t36S4kB0DWOHH7wL0Hk])
  ```

### 查看监视器详细信息

创建复合监视器后，它将显示在监视器列表中**监视器** 标签。这**类型** 列指示监视器的类型，包括复合监视器类型。这**与复合监视器的关联** 列提供了数量，其中有多少复合监视基本监视器用作委托监视器。在**监视名称** 列打开其详细信息窗口。

对于复合监视器，**警报** 详细信息窗口的部分包括**动作** 列，其中包括视图详细信息图标（{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/view-monitor-icon.png" class ="inline-icon" alt ="view monitor icon"/> {:/}）。下图显示了**动作** 列作为右侧的最后一列。

<img src="{{site.url}}{{site.baseurl}}/images/alerting/comp-details-alerts.png" alt="Alerts section of the monitor details window" width="75%">

选择此图标以打开**警报详细信息** 窗户。此窗口向您显示了生成链式警报的执行的一部分，并包括生成审核警报的代表监视器。选择**X** 在上部-窗口的右角关闭**警报详细信息**。

返回到**警报** 监视器详细信息窗口的部分，您可以选择该复选框**警报开始时间** 突出显示警报。警报突出显示后，您可以选择**承认** 在上部-本节的正确部分。警报已得到确认，并在**状态** 列从主动变为确认。

