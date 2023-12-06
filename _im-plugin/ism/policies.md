---
layout: default
title: 策略
nav_order: 1
parent: 索引状态管理
has_children: false
---

# 政策

策略是定义以下内容的 JSON 文档：

- *国家*索引可以处于哪个位置，包括新索引的默认状态。例如，你可以将状态命名为“热”、“暖”、“删除”等。有关详细信息，请参阅[States](#states)。
- 你希望插件在索引进入状态（例如执行滚动更新）时采取的任何*行动*步骤。有关详细信息，请参阅[Actions](#actions)。
- 索引进入新状态（称为*转换*）所必须满足的条件。例如，如果索引已超过 8 周，则可能需要将其移至“删除”状态。有关详细信息，请参阅[转换](#transitions)。

换言之，策略定义*国家*了索引可以处于的状态、处于状态时要执行的条件以及状态之间必须满足的*行动**过渡*条件。

在设计策略方面，你可以完全灵活地进行操作。你可以创建任何状态，转换到任何其他状态，并在每种状态中指定任意数量的操作。

下表列出了策略的相关字段。

字段 | 描述 | 类型 | 必需 | 只读
:--- | :--- |:--- |:--- |
 `policy_id` | 策略的名称。| `string` | 是 | 是的
 `description` | 用户可读的策略描述。| `string` | 是 | 不
 `ism_template` | 指定 ISM 模板以自动将策略应用于新创建的索引。| `nested list of objects` | 否 | 不
 `ism_template.index_patterns` | 指定与新创建的索引名称匹配的模式。| `list of strings` | 否 | 不
 `ism_template.priority` | 指定当多个策略与新创建的索引名称匹配时要消除歧义的优先级。| `number` | 否 | 不
 `last_updated_time`  | 上次更新策略的时间。| `timestamp` | 是 | 是的
 `error_notification` | 错误通知的目标和消息模板。目标可以是 Amazon Chime、Slack 或 Webhook URL。| `object` | 否 | 不
 `default_state` | 使用此策略的每个索引的默认起始状态。| `string` | 是 | 不
 `states` | 你在策略中定义的状态。| `nested list of objects` | 是 | 不

---

#### 目录
1. 目录
{:toc}


---

## 国家

状态是对托管索引当前所处状态的描述。托管索引一次只能处于一种状态。每个状态都有在进入状态时按顺序执行的关联操作，以及在所有操作完成后检查的转换。

下表列出了可以为状态定义的参数。

字段 | 描述 | 类型 | 必填
:--- | :--- |:--- |:--- |
 `name` | 状态的名称。| `string` | 是的
 `actions` | 进入状态后要执行的操作。有关详细信息，请参见[Actions](#actions). | | `nested list of objects` 是的
 `transitions` | 接下来的状态以及过渡到这些状态所需的条件。如果不存在转换，则策略假定它已完成，现在可以停止管理索引。有关详细信息，请参见[转换](#transitions). | | `nested list of objects` 是的

---

## 行动

操作是策略在进入特定状态时按顺序执行的步骤。

ISM 按照操作的定义顺序执行操作。例如，如果定义操作 [A，B，C，D]，ISM 将执行操作 A，然后根据群集设置 `plugins.index_state_management.job_interval` 进入休眠期。休眠期结束后，ISM 将继续执行其余操作。但是，如果 ISM 无法成功执行操作 A，则操作将结束，并且不会执行操作 B、C 和 D。

或者，你可以定义操作的超时期限，如果超过该期限，则强制使操作失败。例如，如果超时设置为 `1d`，并且 ISM 在一天内未完成操作，则即使在重试后，操作也会失败。

下表列出了可以为操作定义的参数。

参数 | 描述 | 类型 | 必需 | 违约
:--- | :--- |:--- |:--- |
 `timeout` | 操作的超时期限。接受分钟、小时和天的时间单位。| `time unit` | 否 |-
 `retry` | 操作的重试配置。| `object` | 否 | 特定于操作

该 `retry` 操作具有以下参数：

参数 | 描述 | 类型 | 必需 | 违约
:--- | :--- |:--- |:--- |
 `count` | 重试次数。| `number` | 是 |-
 `backoff` | 重试时要使用的回退策略类型。有效值为 Exponential、Constant 和 Linear。| `string` | 否 | 指数
 `delay` | 两次重试之间等待的时间。接受分钟、小时和天的时间单位。| `time unit` | 否 |1 分钟

以下示例操作的超时期限为 1 小时。该策略使用指数回退策略重试此操作 3 次，每次重试之间有 10 分钟的延迟：

```json
"actions": {
  "timeout": "1h",
  "retry": {
    "count": 3,
    "backoff": "exponential",
    "delay": "10m"
  }
}
```

有关可用单位类型的列表，请参见[支持的单位]({{site.url}}{{site.baseurl}}/opensearch/units/)。

## ISM 支持的操作

ISM 支持以下操作：

- [force_merge](#force_merge)
- [read_only](#read_only)
- [read_write](#read_write)
- [replica_count](#replica_count)
- [shrink](#shrink)
- [close](#close)
- [open](#open)
- [delete](#delete)
- [rollover](#rollover)
- [通知](#notification)
- [snapshot](#snapshot)
- [index_priority](#index_priority)
- [allocation](#allocation)
- [rollup](#rollup)

### force_merge

通过合并单个分片的分段来减少 Lucene 段的数量。此操作尝试在开始合并过程之前将索引设置为状态 `read-only`。

参数 | 描述 | 类型 | 必填
:--- | :--- |:--- |:--- |
 `max_num_segments` | 要将分片减少到的分段数。| `number` | 是的
wait_for_completion | 布尔值 | 设置为 `false` 时，请求会立即返回，而不是在操作完成后返回。若要监视操作状态，请使用[Tasks API]({{site.url}}{{site.baseurl}}/api-reference/tasks/)请求返回的任务 ID。缺省值为 `true`。
task_execution_timeout | 时间 | 显式任务执行超时。仅当 wait_for_completion 设置为 `false` 时才有用。默认值为 `1h`. | 不

```json
{
  "force_merge": {
    "max_num_segments": 1
  }
}
```

### read_only

将托管索引设置为只读。

```json
{
  "read_only": {}
}
```

将托管索引的索引设置 `index.blocks.write` 设置为 `true`。此块不会阻止索引刷新。*** 注意：**

### read_write

将托管索引设置为可写。

```json
{
  "read_write": {}
}
```

### replica_count

设置要分配给索引的副本数。

参数 | 描述 | 类型 | 必填
:--- | :--- |:--- |:--- |
 `number_of_replicas` | 定义要分配给索引的副本数。| `number` | 是的

```json
{
  "replica_count": {
    "number_of_replicas": 2
  }
}
```

有关设置复制副本的信息，请参见[主分片和副本分片]({{site.url}}{{site.baseurl}}/opensearch#primary-and-replica-shards)。

### 收缩

允许你减少索引中的主分片数量。通过此操作，你可以指定：

- 目标索引应包含的主分片数。
- 目标索引中主分片的最大分片大小。
- 指定百分比以缩小目标索引中的主分片数量。

```json
"shrink": {
    "num_new_shards": 1,
    "target_index_name_template": {
        "source": "{{ctx.index}}_shrunken"
    },
    "aliases": [
      {
        "my-alias": {}
      }
    ],
    "force_unsafe": false
}
```

参数 | 描述 | 类型 | 示例 | 必填
:--- | :--- |:--- |:--- |
 `num_new_shards` | 缩减索引中主分片的最大数量。| 整数 | `5` | 可以，但是它不能与 `max_shard_size` 或 `percentage_of_source_shards`
 `max_shard_size` | 目标索引的分片的最大大小（以字节为单位）。| 关键字 | `5gb` | 可以，但是它不能与 `num_new_shards` 或 `percentage_of_source_shards`
 `percentage_of_source_shards` | 要收缩的原始主分片数的百分比。此参数表示在缩减主分片数量时要使用的最小百分比。必须介于 0.0 和 1.0 之间，不包括。| 百分比 | `0.5` | 可以，但是它不能与 `max_shard_size` 或 `num_new_shards`
 `target_index_name_template` | 缩小索引的名称。接受字符串和 Mustache 变量 `{{ctx.index}}` `{{ctx.indexUuid}}` 和. | 或 Mustache 模板 | | `string` `{"source": "{{ctx.index}}_shrunken"}` 不
 `aliases` | 要添加到新索引的别名。| 对象 | `myalias` | 否，但必须是别名对象的数组
 `force_unsafe` | 如果为 true，则即使没有副本，也执行收缩操作。| 布尔值 | `false` | 不

如果要添加到 `aliases` 操作中，该参数必须包含[别名对象]({{site.url}}{{site.baseurl}}/api-reference/alias/).例如

```json
"aliases": [
  {
    "my-alias": {}
  },
  {
    "my-second-alias": {
      "is_write_index": false,
      "filter": {
        "multi_match": {
          "query": "QUEEN",
          "fields": ["speaker", "text_entry"]
        }
      },
      "index_routing" : "1",
      "search_routing" : "1"
    }
  },
]
```

### 关闭

关闭托管索引。

```json
{
  "close": {}
}
```

关闭的索引保留在磁盘上，但不占用 CPU 或内存。你无法读取、写入或搜索已关闭的索引。

如果你需要将数据保留的时间长于主动搜索数据所需的时间，并且数据节点上有足够的磁盘空间，那么关闭索引是一个不错的选择。如果需要再次搜索数据，重新打开已关闭的索引比从快照还原索引更简单。

### 打开

打开托管索引。

```json
{
  "open": {}
}
```

### 删除

删除托管索引。

```json
{
  "delete": {}
}
```

### 过渡

当托管索引满足其中一个滚动更新条件时，将别名滚动到新索引。

ISM 根据不断_不_检查操作**策略的每次执行****设置间隔**条件。如果值**已达到**或_已超过_配置的限制**执行检查时**.例如，如果 `min_size` 配置为 100GiB 值，则 ISM 可能会检查索引的 99 GiB，而不执行滚动更新。但是，如果索引在下次检查时已超过限制（例如，105GiB），则执行该操作。

如果需要跳过滚动更新操作，可以将索引设置 `index.plugins.index_state_management.rollover_skip` 设置为 `true`。例如，如果收到错误消息“缺少别名或不是写入索引...”，则可以将 `index.plugins.index_state_management.rollover_skip` 参数设置为 `true` 并重试以跳过滚动更新操作。

索引格式必须与以下模式匹配： `^.*-\d+$`。例如， `(logs-000001)`.设置为 `index.plugins.index_state_management.rollover_alias` 要滚动更新的别名。

参数 | 描述 | 类型 | 示例 | 必填
:--- | :--- |:--- |:--- |
 `min_size` | 滚动更新索引所需的主分片存储总量（不包括副本数）的最小大小。例如，如果你设置为 `min_size` 100 GiB，并且你的索引有 5 个主分片和 5 个副本分片，每个分片 20 GiB，则所有主分片的总大小为 100 GiB，因此会发生滚动更新。请参阅**重要**上面的注释。| | `20gb` 或 `5mb` | `string` 不
 `min_primary_shard_size` | 滚动更新索引所需的最小存储大小**单个主分片**。例如，如果设置为 `min_primary_shard_size` 30 GiB，并且**其中之一**索引中的主分片的大小大于条件，则会发生滚动更新。请参阅**重要**上面的注释。| | `20gb` 或 `5mb` | `string` 不
 `min_doc_count` | 滚动更新索引所需的最小文档数。请参阅**重要**上面的注释。| | `2000000` | `number` 不
 `min_index_age` | 展期索引所需的最低年龄。索引年龄是其创建与现在之间的时间。支持的单位为 `d`（天）、（小时）、 `m`（分钟）、（秒）、 `ms` `h` `s`（毫秒）和 `micros`（微秒）。请参阅**重要**上面的注释。| | `5d` 或 `7h` | `string` 不
 `copy_alias` | 控制是否将所有别名从当前索引复制到新创建的索引。默认值为 `false`. | `boolean` | `true` 或 `false` | 不

```json
{
  "rollover": {
    "min_size": "50gb"
  }
}
```

```json
{
  "rollover": {
    "min_primary_shard_size": "30gb"
  }
}
```

```json
{
  "rollover": {
    "min_doc_count": 100000000
  }
}
```

```json
{
  "rollover": {
    "min_index_age": "30d"
  }
}
```

### 通知

向你发送通知。

参数 | 描述 | 类型 | 必填
:--- | :--- |:--- |:--- |
 `destination` | 目标 URL。| `Slack, Amazon Chime, or webhook URL` | 是的
 `message_template` | 消息的文本。你可以使用向消息[小胡子模板](https://mustache.github.io/mustache.5.html)添加变量。| | `object` 是的

目标系统**必须**返回响应，否则通知操作将引发错误。

#### 示例 1：铃声通知

```json
{
  "notification": {
    "destination": {
      "chime": {
        "url": "<url>"
      }
    },
    "message_template": {
      "source": "the index is {% raw %}{{ctx.index}}{% endraw %}"
    }
  }
}
```

#### 示例 2：自定义 Webhook 通知

```json
{
  "notification": {
    "destination": {
      "custom_webhook": {
        "url": "https://<your_webhook>"
      }
    },
    "message_template": {
      "source": "the index is {% raw %}{{ctx.index}}{% endraw %}"
    }
  }
}
```

#### 示例 3：松弛通知

```json
{
  "notification": {
    "destination": {
      "slack": {
        "url": "https://hooks.slack.com/services/xxx/xxxxxx"
      }
    },
    "message_template": {
      "source": "the index is {% raw %}{{ctx.index}}{% endraw %}"
    }
  }
}
```

你可以在 `ctx` 消息中使用变量来表示基于过去执行的策略的多个策略参数。例如，如果你的策略具有滚动更新操作，则可以在消息中用于 `{% raw %}{{ctx.action.name}}{% endraw %}` 表示滚动更新的名称。

以下 `ctx` 变量选项可用于每个策略：

#### 保证变量

参数 | 描述 | 类型
:--- | :--- |:--- |:--- |
 `index` | 索引的名称。| `string`
 `index_uuid` | 索引的 uuid。| `string`
 `policy_id` | 策略的名称。| `string`

### 快照

备份集群的索引和状态。有关快照的详细信息，请参阅[拍摄和恢复快照]({{site.url}}{{site.baseurl}}/opensearch/snapshots/snapshot-restore)。

该 `snapshot` 操作具有以下参数：

参数 | 描述 | 类型 | 必需 | 违约
:--- | :--- |:--- |:--- |
 `repository` | 你通过本机快照 API 操作注册的存储库名称。| `string` | 是 |-
 `snapshot` | 快照的名称。接受字符串和 Mustache 变量 `{{ctx.index}}` 和 `{{ctx.indexUuid}}`.如果 Mustache 变量无效，则快照名称默认为索引的名称。| `string` 或小胡子模板 | 是 |-

```json
{
  "snapshot": {
    "repository": "my_backup",
    "snapshot": "{{ctx.indexUuid}}"
  }
}
```

### index_priority

在特定状态下设置索引的优先级。如果可能，将按其优先级顺序恢复未分配的索引分片。首先恢复具有较高优先级值的索引，然后恢复具有较低优先级值的索引。

该 `index_priority` 操作具有以下参数：

参数 | 描述 | 类型 | 必需 | 违约
:--- | :--- |:--- |:--- |:---
 `priority` | 索引进入状态后的优先级。| `number` | 是 |1

```json
"actions": [
  {
    "index_priority": {
      "priority": 50
    }
  }
]
```

### 分配

将索引分配给具有特定属性集[like this]({{site.url}}{{site.baseurl}}/opensearch/cluster/#advanced-step-7-set-up-a-hot-warm-architecture)的节点。例如，设置为 `require` `warm` 仅将数据移动到“暖”节点。

该 `allocation` 操作具有以下参数：

参数 | 描述 | 类型 | 必填
:--- | :--- |:--- |:---
 `require` | 将索引分配给具有指定属性的节点。| `string` | 是的
 `include` | 将索引分配给具有任何指定属性的节点。| `string` | 是的
 `exclude` | 不要将索引分配给具有任何指定属性的节点。| `string` | 是的
 `wait_for` | 等待策略执行，然后再将索引分配给具有指定属性的节点。| `string` | 是的

```json
"actions": [
  {
    "allocation": {
      "require": { "temp": "warm" }
    }
  }
]
```

### 汇总

[索引汇总]({{site.url}}{{site.baseurl}}/im-plugin/index-rollups/index/)允许你通过将旧数据汇总到汇总索引中来定期降低数据粒度。

汇总作业可以是连续的，也可以是非连续的。使用 ISM 策略创建的汇总作业只能是非连续的。
{: .note}

#### Path 和 HTTP 方法

````bash
PUT _plugins/_rollup/jobs/<rollup_id>
GET _plugins/_rollup/jobs/<rollup_id>
DELETE _plugins/_rollup/jobs/<rollup_id>
POST _plugins/_rollup/jobs/<rollup_id>/_start
POST _plugins/_rollup/jobs/<rollup_id>/_stop
GET _plugins/_rollup/jobs/<rollup_id>/_explain
````

#### Sample ISM rollup policy

````json
{
    "policy": {
        "description": "Sample rollup" ,
        "default_state": "rollup",
        "states": [
            {
                "name": "rollup",
                "actions": [
                    {
                        "rollup": {
                            "ism_rollup": {
                                "description": "Creating rollup through ISM",
                                "target_index": "target",
                                "page_size": 1000,
                                "dimensions": [
                                    {
                                        "date_histogram": {
                                            "fixed_interval": "60m",
                                            "source_field": "order_date",
                                            "target_field": "order_date",
                                            "timezone": "America/Los_Angeles"
                                        }
                                    },
                                    {
                                        "terms": {
                                            "source_field": "customer_gender",
                                            "target_field": "customer_gender"
                                        }
                                    },
                                    {
                                        "terms": {
                                            "source_field": "day_of_week",
                                            "target_field": "day_of_week"
                                        }
                                    }
                                ],
                                "metrics": [
                                    {
                                        "source_field": "taxless_total_price",
                                        "metrics": [
                                            {
                                                "sum": {}
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
                    }
                ],
                "transitions": []
            }
        ]
    }
}
````

#### Request fields

创建ISM策略时需要请求字段。您可以引用[index rolups api]({{site.url}}) {{site.baseurl}}/im im-插件/索引-卷/卷-api/#创造-或者-更新-一个-指数-卷起-作业）页面用于请求字段选项。

#### Adding a rollup policy in Dashboards

要在仪表板中添加汇总策略，请按照以下步骤操作。

- 选择顶部的菜单按钮-仪表板用户界面的左侧。
- 在仪表板菜单中，选择`Index Management`。
- 在下一个屏幕上选择`Rollup jobs`。
- 选择`Create rollup` 按钮。
- 按照步骤`Create rollup job` 向导。
- 在`Name` 盒子。
- 您可以引用[index rolups api]({{site.url}}) {{site.baseurl}}/im im-插件/索引-卷/卷-api/#创造-或者-更新-一个-指数-卷起-作业）页面以配置汇总策略。
- 最后，选择`Create` 底部的按钮-仪表板用户界面的右侧。

---

## 过渡

过渡定义了一个国家需要满足的条件。在当前状态中的所有操作完成后，策略开始检查过渡条件。

ISM按定义的顺序评估过渡。例如，如果定义过渡：[a，b，c，d]，则通过此过渡列表进行迭代，直到找到评估到的过渡`true`，然后停止并将下一个状态设置为该过渡中定义的状态。ISM在下次执行中驳回了其余的过渡，并以新状态开始。

如果您没有在过渡中指定任何条件并将其留为空，则假定它相当于始终为true。这意味着政策在检查时刻将索引转换为此状态。

该表列出了您可以为过渡定义的参数。

范围| 描述| 类型| 必需的
:--- | :--- |:--- |:--- |
`state_name` |  如果满足条件，则过渡到状态的名称。| `string` | 是的
`conditions` |  列出过渡的条件。| `list` | 是的

这`conditions` 对象具有以下参数：

范围| 描述| 类型| 必需的
:--- | :--- |:--- |:--- |
`min_index_age` | 过渡所需的指数的最低年龄。| `string` | 不
`min_rollover_age` | 过渡到下一个州所需的最低年龄。| `string` | 不
`min_doc_count` | 过渡所需的索引的最低文件计数。| `number` | 不
`min_size` | 过渡所需的总主碎片存储（不计数复制品）的最小尺寸。例如，如果您设置`min_size` 到100 GIB，您的索引有5个主碎片和5个复制碎片，每个Gib均为20 Gib，所有主要碎片的总尺寸为100 GIB，因此您的索引已过渡到下一个状态。| `string` | 不
`cron` | 这`cron` 如果没有其他过渡首先发生，则会触发过渡。| `object` | 不
`cron.cron.expression` | 这`cron` 触发过渡的表达。| `string` | 是的
`cron.cron.timezone` | 触发过渡的时区。| `string` | 是的

以下示例将索引过渡到`cold` 经过30天后的状态：

```json
"transitions": [
  {
    "state_name": "cold",
    "conditions": {
      "min_index_age": "30d"
    }
  }
]
```

ISM 根据设置的时间间隔检查每次执行策略时的条件。

此示例使用条件在 `cron` 太平洋时间每周六 5：00 转换索引：

```json
"transitions": [
  {
    "state_name": "cold",
    "conditions": {
      "cron": {
        "cron": {
          "expression": "* 17 * * SAT",
          "timezone": "America/Los_Angeles"
        }
      }
    }
  }
]
```

请注意，此条件不会在下午 5：00 执行;作业仍会根据 `job_interval` 设置执行。由于开始时间的这种差异以及在检查转换条件之前完成操作所需的时间，我们建议不要使用过于狭窄的 cron 表达式。例如，不要使用 `15 17 * * SAT`（周六下午 5：15）。

此示例使用的一个小时窗口通常就足够了，但你可以将其增加到 2--3 小时，以避免错过该窗口并等待一周才能进行转换。或者，你可以使用更广泛的表达式，例如 `* * * * SAT,SUN` 在周末的任何时间进行转换。

有关编写 cron 表达式的信息，请参见[Cron 表达式参考]({{site.url}}{{site.baseurl}}/monitoring-plugins/alerting/cron/)。

---

## 错误通知

如果托管索引失败，该 `error_notification` 操作会向你发送通知。它通知单个目标或[通知通道]({{site.url}}{{site.baseurl}}/notifications-plugin/index)自定义消息。

在策略级别设置错误通知：

```json
{
  "policy": {
    "description": "hot warm delete workflow",
    "default_state": "hot",
    "schema_version": 1,
    "error_notification": { },
    "states": [ ]
  }
}
```

参数 | 描述 | 类型 | 必填
:--- | :--- |:--- |:--- |
 `destination` | 目标 URL。| `Slack, Amazon Chime, or webhook URL` | 如果未指定，则 `channel` 为”
 `channel` | 通知通道的 ID | `string` | 如果未指定，则 `destination` 为”
 `message_template` | 消息的文本。你可以使用向消息[小胡子模板](https://mustache.github.io/mustache.5.html)添加变量。| | `object` 是的

目标系统**必须**返回响应，否则操作 `error_notification` 将引发错误。

#### 示例 1：铃声通知

```json
{
  "error_notification": {
    "destination": {
      "chime": {
        "url": "<url>"
      }
    },
    "message_template": {
      "source": "The index {% raw %}{{ctx.index}}{% endraw %} failed during policy execution."
    }
  }
}
```

#### 示例 2：自定义 Webhook 通知

```json
{
  "error_notification": {
    "destination": {
      "custom_webhook": {
        "url": "https://<your_webhook>"
      }
    },
    "message_template": {
      "source": "The index {% raw %}{{ctx.index}}{% endraw %} failed during policy execution."
    }
  }
}
```

#### 示例 3：松弛通知

```json
{
  "error_notification": {
    "destination": {
      "slack": {
        "url": "https://hooks.slack.com/services/xxx/xxxxxx"
      }
    },
    "message_template": {
      "source": "The index {% raw %}{{ctx.index}}{% endraw %} failed during policy execution."
    }
  }
}
```

#### 示例 4：使用通知通道

```json
{
  "error_notification": {
    "channel": {
      "id": "some-channel-config-id"
    },
    "message_template": {
      "source": "The index {% raw %}{{ctx.index}}{% endraw %} failed during policy execution."
    }
  }
}
```

你可以对 `ctx` 变量使用与[通知](#notification)操作相同的选项。

## 带有 ISM 模板的自动滚动更新策略示例

以下示例模板策略适用于滚动更新用例。

如果要跳过索引的滚动更新，请在该索引的设置中设置为 `index.plugins.index_state_management.rollover_skip` `true`。

1. 使用 `ism_template` 字段创建策略：

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

   你需要指定该 `index_patterns` 字段。如果未指定 `priority` 的值，则默认为 0。

2. 使用以下 `rollover_alias` 命令 `log` 设置模板：

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

3. 使用 `log` 别名创建索引：

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

4. 为文档编制索引以触发滚动更新条件：

   ```json
   POST log/_doc
   {
     "message": "dummy"
   }
   ```

5. 验证策略是否附加到 `log-000001` 索引：

   ```json
   GET _plugins/_ism/explain/log-000001?pretty
   ```

## 别名操作的 ISM 模板策略示例

以下示例策略适用于别名操作用例。

在以下示例中，第一个作业将触发滚动更新操作，并将创建一个新索引。接下来，将另一个文档添加到两个索引中。然后，新作业将导致第二个索引指向日志别名，并且由于别名操作，将删除较旧的索引。

首先，创建 ISM 策略：

```json
PUT /_plugins/_ism/policies/rollover_policy?pretty
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
        "transitions": [{
            "state_name": "alias",
            "conditions": {
              "min_doc_count": "2"
            }
          }]
      },
      {
        "name": "alias",
        "actions": [
          {
            "alias": {
              "actions": [
                {
                  "remove": {
                      "alias": "log"
                  }
                }
              ]
            }
          }
        ]
      }
    ],
    "ism_template": {
      "index_patterns": ["log*"],
      "priority": 100
    }
  }
}
```

接下来，创建一个索引模板，以在其上启用策略：

```json
PUT /_index_template/ism_rollover?
{
  "index_patterns": ["log*"],
  "template": {
   "settings": {
    "plugins.index_state_management.rollover_alias": "log"
   }
 }
}
```
{% include copy-curl.html %}

接下来，更改群集设置以每分钟触发一次作业：

```json
PUT /_cluster/settings?pretty=true
{
  "persistent" : {
    "plugins.index_state_management.job_interval" : 1
  }
}
```
{% include copy-curl.html %}

接下来，创建一个新索引：

```json
PUT /log-000001
{
  "aliases": {
    "log": {
      "is_write_index": true
    }
  }
}
```
{% include copy-curl.html %}

最后，将文档添加到索引以触发作业：

```json
POST /log-000001/_doc
{
  "message": "dummy"
}
```
{% include copy-curl.html %}

你可以使用别名和索引 API 验证以下步骤：

```json
GET /_cat/indices?pretty
```
{% include copy-curl.html %}

```json
GET /_cat/aliases?pretty
```
{% include copy-curl.html %}

注意： `index` 别名操作策略不允许使用和 `remove_index` 参数。 `add` 只允许和 `remove` alias 操作参数。
{: .warning}

## 示例策略

以下示例策略实现 `hot`、 `warm` 和 `delete` 工作流。你可以将此策略用作模板，根据资源的活动级别确定索引的优先级。

在这种情况下，索引最初处于状态 `hot`。一天后，它会变为一种 `warm` 状态，即副本数增加到 5 以提高读取性能。

30 天后，策略将此索引移动到状态 `delete`。该服务会向 Chime 聊天室发送索引正在删除的通知，然后将其永久删除。

```json
{
  "policy": {
    "description": "hot warm delete workflow",
    "default_state": "hot",
    "schema_version": 1,
    "states": [
      {
        "name": "hot",
        "actions": [
          {
            "rollover": {
              "min_index_age": "1d",
              "min_primary_shard_size": "30gb"
            }
          }
        ],
        "transitions": [
          {
            "state_name": "warm"
          }
        ]
      },
      {
        "name": "warm",
        "actions": [
          {
            "replica_count": {
              "number_of_replicas": 5
            }
          }
        ],
        "transitions": [
          {
            "state_name": "delete",
            "conditions": {
              "min_index_age": "30d"
            }
          }
        ]
      },
      {
        "name": "delete",
        "actions": [
          {
            "notification": {
              "destination": {
                "chime": {
                  "url": "<URL>"
                }
              },
              "message_template": {
                "source": "The index {% raw %}{{ctx.index}}{% endraw %} is being deleted"
              }
            }
          },
          {
            "delete": {}
          }
        ]
      }
    ],
    "ism_template": {
      "index_patterns": ["log*"],
      "priority": 100
    }
  }
}
```

此图将上述策略的、 `transitions` 和 `actions` 显示 `states` 为有限状态机。有关有限状态机的更多信息，请参见[Wikipedia](https://en.wikipedia.org/wiki/Finite-state_machine)。

![策略状态机]({{site.baseurl}}/images/ism.png)
