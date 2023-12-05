---
layout: default
title: 快照管理API
parent: 快照
nav_order: 30
has_children: false
grand_parent: 可用性和恢复
redirect_from: 
  - /opensearch/snapshots/sm-api/
---

# 快照管理API

使用快照管理（SM）API自动化[拍摄快照]({{site.url}}{{site.baseurl}}/opensearch/snapshots/snapshot-restore#take-snapshots)。

---

#### Table of contents
- TOC
{:toc}


---

## 创建或更新策略
引入2.1
{: .label .label-purple }

创建或更新SM策略。

#### 要求

创造：

```json
POST _plugins/_sm/policies/<policy_name> 
```

更新：

```json
PUT _plugins/_sm/policies/<policy_name>?if_seq_no=0&if_primary_term=1
```

您必须提供`seq_no` 和`primary_term` 更新请求的参数。

### 例子

```json
POST _plugins/_sm/policies/daily-policy
{
  "description": "Daily snapshot policy",
  "creation": {
    "schedule": {
      "cron": {
        "expression": "0 8 * * *",
        "timezone": "UTC"
      }
    },
    "time_limit": "1h"
  },
  "deletion": {
    "schedule": {
      "cron": {
        "expression": "0 1 * * *",
        "timezone": "America/Los_Angeles"
      }
    },
    "condition": {
      "max_age": "7d",
      "max_count": 21,
      "min_count": 7
    },
    "time_limit": "1h"
  },
  "snapshot_config": {
    "date_format": "yyyy-MM-dd-HH:mm",
    "timezone": "America/Los_Angeles",
    "indices": "*",
    "repository": "s3-repo",
    "ignore_unavailable": "true",
    "include_global_state": "false",
    "partial": "true",
    "metadata": {
      "any_key": "any_value"
    }
  },
  "notification": {
    "channel": {
      "id": "NC3OpoEBzEoHMX183R3f"
    },
    "conditions": {
      "creation": true,
      "deletion": false,
      "failure": false,
      "time_limit_exceeded": false
    }
  }
}
```

### 回复

```json
{
  "_id" : "daily-policy-sm-policy",
  "_version" : 5,
  "_seq_no" : 54983,
  "_primary_term" : 21,
  "sm_policy" : {
    "name" : "daily-policy",
    "description" : "Daily snapshot policy",
    "schema_version" : 15,
    "creation" : {
      "schedule" : {
        "cron" : {
          "expression" : "0 8 * * *",
          "timezone" : "UTC"
        }
      },
      "time_limit" : "1h"
    },
    "deletion" : {
      "schedule" : {
        "cron" : {
          "expression" : "0 1 * * *",
          "timezone" : "America/Los_Angeles"
        }
      },
      "condition" : {
        "max_age" : "7d",
        "min_count" : 7,
        "max_count" : 21
      },
      "time_limit" : "1h"
    },
    "snapshot_config" : {
      "indices" : "*",
      "metadata" : {
        "any_key" : "any_value"
      },
      "ignore_unavailable" : "true",
      "timezone" : "America/Los_Angeles",
      "include_global_state" : "false",
      "date_format" : "yyyy-MM-dd-HH:mm",
      "repository" : "s3-repo",
      "partial" : "true"
    },
    "schedule" : {
      "interval" : {
        "start_time" : 1656425122909,
        "period" : 1,
        "unit" : "Minutes"
      }
    },
    "enabled" : true,
    "last_updated_time" : 1656425122909,
    "enabled_time" : 1656425122909,
    "notification" : {
      "channel" : {
        "id" : "NC3OpoEBzEoHMX183R3f"
      },
      "conditions" : {
        "creation" : true,
        "deletion" : false,
        "failure" : false,
        "time_limit_exceeded" : false
      }
    }
  }
}
```

### 参数

您可以指定以下参数来创建/更新SM策略。

范围| 类型| 描述
:--- | :--- |:--- |:--- |
`description` | 细绳| SM政策的描述。选修的。
`enabled` | 布尔| 应该在创建时启用此SM策略吗？选修的。
`snapshot_config` | 目的| 快照创建的配置选项。必需的。
`snapshot_config.date_format` | 细绳| 快照名称具有格式`<policy_name>-<date>-<random number>`。`date_format` 指定快照名称中日期的格式。支持OpenSearch支持的所有日期格式。选修的。默认为"yyyy-MM-dd'T'HH:mm:ss"。
`snapshot_config.date_format_timezone` | 细绳| 快照名称具有格式`<policy_name>-<date>-<random number>`。`date_format_timezone` 指定快照名称中日期的时区。选修的。默认值为UTC。
`snapshot_config.indices` | 细绳| 快照中的索引名称。多个索引名称由`,`。支持通配符（`*`）。选修的。默认为`*` （所有索引）。
`snapshot_config.repository` | 细绳| 存储快照的存储库。必需的。
`snapshot_config.ignore_unavailable` | 布尔| 您想忽略不可用的索引吗？选修的。默认为`false`。
`snapshot_config.include_global_state` | 布尔| 您要包括群集状态吗？选修的。默认为`true` 因为[安全插件注意事项]({{site.url}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/snapshots/snapshot-restore#security-considerations)。
`snapshot_config.partial` | 布尔| 您想允许部分快照吗？选修的。默认为`false`。
`snapshot_config.metadata` | 目的| 元数据以键/值对的形式。选修的。
`creation` | 目的| 快照创建的配置。必需的。
`creation.schedule` | 细绳| Cron时间表用于创建快照。必需的。
`creation.time_limit` | 细绳| 设定最大的时间等待快照创建完成。如果time_limit的长度超过拍摄快照的计划时间间隔，则直到time_limit lelapses都没有计划快照。例如，如果将Time_limit设置为35分钟，并且从午夜开始每30分钟拍摄每30分钟，则拍摄00:00和01:00的快照，但跳过了00:30的快照。选修的。
`deletion` | 目的| 快照删除的配置。选修的。默认值是保留所有快照。
`deletion.schedule` | 细绳| CRON时间表用于删除快照。选修的。默认是使用`creation.schedule`，这是必需的。
`deletion.time_limit` | 细绳| 设定最大时间等待快照删除完成。选修的。
`deletion.delete_condition` | 目的| 快照删除的条件。选修的。
`deletion.delete_condition.max_count` | 整数| 要保留的最大快照数。选修的。
`deletion.delete_condition.max_age` | 细绳| 保留快照的最长时间。选修的。
`deletion.delete_condition.min_count` | 整数| 要保留的最小快照数量。选修的。默认是一个。
`notification` | 目的| 定义SM事件的通知。选修的。
`notification.channel` | 目的| 定义通知渠道。你必须[创建并配置通知频道]({{site.url}}{{site.baseurl}}/notifications-plugin/api) 在设置SM通知之前。必需的。
`notification.channel.id` | 细绳| 通知通道的通道ID。要获取所有创建的频道的通道ID，请使用`GET _plugins/_notifications/configs`。必需的。
`notification.conditions` | 目的| 您想通知的SM事件。将您感兴趣的人设置为`true`。
`notification.conditions.creation` | 布尔| 您是否想要有关快照创建的通知？选修的。默认为`true`。
`notification.conditions.deletion` | 布尔| 您是否需要有关快照删除的通知？选修的。默认为`false`。
`notification.conditions.failure` | 布尔| 您是否需要有关创建或删除故障的通知？选修的。默认为`false`。
`notification.conditions.time_limit_exceeded` | 布尔| 当快照操作比time_limit花费更长的时间时，您是否需要通知？选修的。默认为`false`。

## 获取政策
引入2.1
{: .label .label-purple }

获得SM政策。

#### 要求

获取所有SM政策：

```json
GET _plugins/_sm/policies
```
您可以使用[请求参数]({{site.url}}{{site.baseurl}}/query-dsl/full-text/query-string/) 并指定分页，要按顺序排序的字段，然后排序：

```json
GET _plugins/_sm/policies?from=0&size=20&sortField=sm_policy.name&sortOrder=desc&queryString=*
```

获取特定的SM政策：

```
GET _plugins/_sm/policies/<policy_name>
```

### 例子

```json
GET _plugins/_sm/policies/daily-policy
```

### 回复

```json
{
  "_id" : "daily-policy-sm-policy",
  "_version" : 6,
  "_seq_no" : 44696,
  "_primary_term" : 19,
  "sm_policy" : {
    "name" : "daily-policy",
    "description" : "Daily snapshot policy",
    "schema_version" : 15,
    "creation" : {
      "schedule" : {
        "cron" : {
          "expression" : "0 8 * * *",
          "timezone" : "UTC"
        }
      },
      "time_limit" : "1h"
    },
    "deletion" : {
      "schedule" : {
        "cron" : {
          "expression" : "0 1 * * *",
          "timezone" : "America/Los_Angeles"
        }
      },
      "condition" : {
        "max_age" : "7d",
        "min_count" : 7,
        "max_count" : 21
      },
      "time_limit" : "1h"
    },
    "snapshot_config" : {
      "metadata" : {
        "any_key" : "any_value"
      },
      "ignore_unavailable" : "true",
      "include_global_state" : "false",
      "date_format" : "yyyy-MM-dd-HH:mm",
      "repository" : "s3-repo",
      "partial" : "true"
    },
    "schedule" : {
      "interval" : {
        "start_time" : 1656341042874,
        "period" : 1,
        "unit" : "Minutes"
      }
    },
    "enabled" : true,
    "last_updated_time" : 1656341042874,
    "enabled_time" : 1656341042874
  }
}
```

## 解释
引入2.1
{: .label .label-purple }

为指定的所有策略提供启用/禁用状态和元数据。多个策略名称与`,`。您还可以用通配符模式指定所需的政策。

<img src="{{site.url}}{{site.baseurl}}/images/sm-state-machine.png" alt="SM State Machine" width="150" style="float: left; margin-right: 15px;"/>

SM使用状态机进行快照创建和删除。左侧的图像显示了创建工作流的一个执行期，从create_start状态到creation_fined态。删除工作流与创建工作流相同的模式。

创建工作流程始于Creation_start状态，并不断检查Creation Cron计划中的条件是否满足。满足条件后，创建工作流将切换到creation_condition_met状态，并继续到创建状态。创建状态异步调用创建快照API，然后等待快照创建以creation_fined状态结束。快照创建结束后，创建工作流将返回到创建_start状态，并且周期继续。这`current_state` 现场`metadata.creation` 和`metadata.deletion` 返回状态机的当前状态。

#### 要求

```json
GET _plugins/_sm/policies/<policy_names>/_explain
```

### 例子

```json
GET _plugins/_sm/policies/daily*/_explain
```

### 回复

```json
{
  "policies" : [
    {
      "name" : "daily-policy",
      "creation" : {
        "current_state" : "CREATION_START",
        "trigger" : {
          "time" : 1656403200000
        }
      },
      "deletion" : {
        "current_state" : "DELETION_START",
        "trigger" : {
          "time" : 1656403200000
        }
      },
      "policy_seq_no" : 44696,
      "policy_primary_term" : 19,
      "enabled" : true
    }
  ]
}
```

下表列出了响应中每个策略的所有字段。

场地| 描述
：--- |：--- 
`name` | SM政策的名称。
`creation` | 有关最新创建操作的信息。请参阅下面的子场。
`deletion` | 有关最新删除操作的信息。请参阅下面的子场。
`policy_seq_no` <br>`policy_primary_term` | SM策略的版本。
`enabled` | 策略运行吗？

下表列出了`creation` 和`deletion` 每个策略的对象。

场地| 描述
：--- |：--- 
`current_state` | 如上所述，运行快照创建/删除的状态机的当前状态。
`trigger.time` | 自时代以来，下一个创建/删除执行时间以毫秒为单位。
`latest_execution` | 描述最新的创建/删除执行。
`latest_execution.status` | 最新创建/删除的执行状态。可能的值是：<br>`IN_PROGRESS`：快照创建/删除已经开始。<br>`SUCCESS`：快照创建/删除已成功完成。<br>`RETRYING`：创建/删除尝试失败了。它将进行三次。<br>`FAILED`：三次重试后创建/删除尝试失败。结束当前执行期，然后转到下一个执行期。<br>`TIME_LIMIT_EXCEEDED`：创建/删除时间超过了策略中设置的Time_limit。结束当前执行期，然后转到下一个执行期。
`latest_execution.start_time` | 自时代以来，最新执行的最新执行时间开始。
`latest_execution.end_time` | 自时代以来，最新执行的最新执行时间的结束时间。
`latest_execution.info.message` | 用户-友好的消息描述了最新执行的状态。
`latest_execution.info.cause` | 如果最新的执行失败，则包含故障原因。
`retry.count` | 剩余的执行重试尝试。


## 开始政策
引入2.1
{: .label .label-purple }

通过设置其策略`enabled` 标记为`true`。

#### 要求

```json
POST  _plugins/_sm/policies/<policy_name>/_start
```

### 例子

```json
POST  _plugins/_sm/policies/daily-policy/_start
```

### 回复

```json
{
  "acknowledged" : true
}
```

## 停止政策
引入2.1
{: .label .label-purple }

设置`enabled` 标记为`false` 用于SM政策。在您之前，该政策不会运行[开始](#start-a-policy) 它。

#### 要求

```json
POST  _plugins/_sm/policies/<policy_name>/_stop
```

### 例子

```json
POST  _plugins/_sm/policies/daily-policy/_stop
```

### 回复

```json
{
  "acknowledged" : true
}
```

## 删除政策
引入2.1
{: .label .label-purple }

删除指定的SM策略。

#### 要求

```json
DELETE  _plugins/_sm/policies/<policy_name>
```

### 例子

```json
DELETE _plugins/_sm/policies/daily-policy
```

### 回复

```json
{
  "_index" : ".opendistro-ism-config",
  "_id" : "daily-policy-sm-policy",
  "_version" : 8,
  "result" : "deleted",
  "forced_refresh" : true,
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 45366,
  "_primary_term" : 20
}
```
