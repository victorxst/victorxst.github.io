---
layout: default
title: 总计的
parent: 处理器
grand_parent: 管道
nav_order: 41
---

# 总计的

这`aggregate` 处理器组事件基于`identification_keys`。然后，处理器对每个组执行一个操作，有助于减少不必要的日志量并随着时间的推移创建聚合日志。您可以使用现有操作或使用Java代码创建自己的自定义聚合。


## 配置

下表描述了您可以使用的选项来配置`aggregate` 处理器。

选项| 必需的| 类型| 描述
:--- | :--- | :--- | :---
dissideification_keys| 是的| 列表| 一个无序的列表进行分组事件。与这些密钥相同的事件放入同一组中。如果事件不包含一个`identification_keys`，那么该密钥的价值被认为等于`null`。至少需要一个识别figation_key（例如，`["sourceIp", "destinationIp", "port"]`）。
行动| 是的| 聚集| 每个组要执行的动作。中的一个[可用的汇总操作](#available-aggregate-actions) 必须提供，或者您可以创建自定义的汇总操作。`remove_duplicates` 和`put_all` 是可用的动作。有关更多信息，请参阅[创建新的汇总操作](https://github.com/opensearch-project/data-prepper/tree/main/data-prepper-plugins/aggregate-processor#creating-new-aggregate-actions)。
group_duration| 不| 细绳| 组应自动结束之前应该存在的时间。支持ISO_8601符号字符串（"PT20.345S"，"PT15M"等等）以及几秒钟的简单符号（`"60s"`）和毫秒（`"1500ms"`）。默认值是`180s`。

## 可用的汇总操作

使用以下汇总操作来确定`aggregate` 处理器处理每个组中的事件。

### remove_duplicates

这`remove_duplicates` 动作处理一个小组的第一个事件，并删除从源复制第一个事件的所有事件。例如，使用`identification_keys: ["sourceIp", "destination_ip"]`：

1. 这`remove_duplicates` 动作过程`{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "status": 200 }`，源中的第一个事件。
2. 数据预先删除`{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "bytes": 1000 }` 事件是因为`sourceIp` 和`destinationIp` 匹配源中的第一个事件。
3. 这`remove_duplicates` 动作过程下一个事件，`{ "sourceIp": "127.0.0.2", "destinationIp": "192.168.0.1", "bytes": 1000 }`。因为`sourceIp` 与小组的第一个事件不同，Data Prepper根据事件创建了一个新组。

### put_all

这`put_all` 动作结合了属于同一组的事件，通过覆盖现有密钥和添加新密钥，类似于Java`Map.putAll`。动作丢弃了组成合并事件的所有事件。例如，使用`identification_keys: ["sourceIp", "destination_ip"]`， 这`put_all` 操作处理以下三个事件：

```
{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "status": 200 }
{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "bytes": 1000 }
{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "http_verb": "GET" }
```

然后，动作将事件结合在一起。然后，管道使用以下组合事件：

```
{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "status": 200, "bytes": 1000, "http_verb": "GET" }
```

### 数数

这`count` 事件计算属于同一组的事件，并生成一个具有值的新事件`identification_keys` 和计数，指示新事件的数量。您可以使用以下配置选项自定义处理器：


 *`count_key`：用于存储计数的密钥。默认名称为`aggr._count`。
*`start_time_key`：用于存储开始时间的密钥。默认名称为`aggr._start_time`。
*`output_format`：聚合事件的格式。
     *`otel_metrics`：默认输出格式。OTEL METICS SUM类型中的输出为值。
    *`raw` - 用`count_key` 字段作为计数值和`start_time_key` 集合启动时间为值。

例如，使用`identification_keys: ["sourceIp", "destination_ip"]`， 这`count` 行动计算并处理以下事件：

```json
{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "status": 200 }
{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "status": 503 }
{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "status": 400 }
```

处理器创建以下事件：

```json
{"isMonotonic":true,"unit":"1","aggregationTemporality":"AGGREGATION_TEMPORALITY_DELTA","kind":"SUM","name":"count","description":"Number of events","startTime":"2022-12-02T19:29:51.245358486Z","time":"2022-12-02T19:30:15.247799684Z","value":3.0,"sourceIp":"127.0.0.1","destinationIp":"192.168.0.1"}
```

### 直方图

这`histogram` Action汇总属于同一组的事件，并生成一个具有值的新事件`identification_keys` 和基于配置的聚合事件的直方图`key`。直方图包含事件，总和，存储桶，计数，以及可选的与对应的值的最小和最大值`key`。动作丢弃了组成合并事件的所有事件。

您可以使用以下配置选项自定义处理器：

*`key`：直方图生成事件中的字段名称。
*`generated_key_prefix`：`key_prefix` 在汇总事件中创建的所有字段使用。具有前缀可确保直方图事件的名称与事件中的字段名称不冲突。
*`units`：在`key`。
*`record_minmax`：一个布尔值，指示直方图是否应包括聚合中值的最小值和最大值。
*`buckets`：一个存储桶列表（类型的值`double`）指示直方图中的存储桶。
*`output_format`：聚合事件的格式。
    *`otel_metrics`：默认输出格式。OTEL METICS SUM类型中的输出为值。
    *`raw`：与`count_key` 具有数值的字段，并且`start_time_key` 集合启动时间为值。


例如，使用`identification_keys: ["sourceIp", "destination_ip", "request"]`，`key: latency`， 和`buckets: [0.0, 0.25, 0.5]`， 这`histogram` 操作处理以下事件：

```
{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "request" : "/index.html", "latency": 0.2 }
{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "request" : "/index.html", "latency": 0.55}
{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "request" : "/index.html", "latency": 0.25 }
{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "request" : "/index.html", "latency": 0.15 }
```

然后处理器创建以下事件：

```json
{"max":0.55,"kind":"HISTOGRAM","buckets":[{"min":-3.4028234663852886E38,"max":0.0,"count":0},{"min":0.0,"max":0.25,"count":2},{"min":0.25,"max":0.50,"count":1},{"min":0.50,"max":3.4028234663852886E38,"count":1}],"count":4,"bucketCountsList":[0,2,1,1],"description":"Histogram of latency in the events","sum":1.15,"unit":"seconds","aggregationTemporality":"AGGREGATION_TEMPORALITY_DELTA","min":0.15,"bucketCounts":4,"name":"histogram","startTime":"2022-12-14T06:43:40.848762215Z","explicitBoundsCount":3,"time":"2022-12-14T06:44:04.852564623Z","explicitBounds":[0.0,0.25,0.5],"request":"/index.html","sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "key": "latency"}
```

### rate_limiter

这`rate_limiter` 动作控制每秒汇总的事件数量。默认情况下，`rate_limiter` 阻止`aggregate` 如果处理器收到的事件比允许的配置数字更多，则该处理器的运行更多。您可以覆盖触发的数字事件`rate_limited` 通过使用`when_exceeds` 配置选项。

您可以使用以下配置选项自定义处理器：

*`events_per_second`：每秒允许的事件数量。
*`when_exceeds`：指示什么动作`rate_limiter` 当收到的事件数量大于每秒允许的事件数量时进行。默认值是`block`，将阻止处理器在每秒允许的最大事件数量之后运行，直到下一秒钟。或者`drop` 选项会丢弃在第二个中收到的多余事件。

例如，如果`events_per_second` 被设定为`1` 和`when_exceeds` 被设定为`drop`，该操作试图在第二次间隔内收到以下事件处理以下事件：

```json
{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "status": 200 }
{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "bytes": 1000 }
{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "http_verb": "GET" }
```

处理以下事件，但是所有其他事件都被忽略了，因为`rate_limiter` 阻止它们：

```json
{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "status": 200 }
```

如果`when_exceeds` 被设定为`drop`，所有三个事件均已处理。

### 百分比_sampler

这`percent_sampler` 行动控制基于一定百分比的事件数量。该动作会删除百分比中未包含的任何事件。

您可以使用`percent` 配置，该配置指示在一个秒间隔内处理的事件的百分比（0％--100％）。

例如，如果将百分比设置为`50`，该动作试图处理一个事件-第二间隔：

```
{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "bytes": 2500 }
{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "bytes": 500 }
{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "bytes": 1000 }
{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "bytes": 3100 }
```

管道处理50％的事件，删除其他事件，并且不会产生新事件：

```
{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "bytes": 500 }
{ "sourceIp": "127.0.0.1", "destinationIp": "192.168.0.1", "bytes": 3100 }
```

## 指标

下表描述了常见[抽象处理器](https://github.com/opensearch-project/data-prepper/blob/main/data-prepper-api/src/main/java/org/opensearch/dataprepper/model/processor/AbstractProcessor.java) 指标。

| 公制名称| 类型| 描述|
| ------------- | ---- | -----------|
| `recordsIn` | 柜台| 表示记录进入管道组件的公制。|
| `recordsOut` | 柜台| 表示管道组件中记录出口的公制。|
| `timeElapsed` | 计时器| 公制表示管道组件执行期间经过的时间。|


这`aggregate` 处理器包括以下自定义指标。

**柜台**

*`actionHandleEventsOut`：已从`handleEvent` 致电配置[行动](https://github.com/opensearch-project/data-prepper/tree/main/data-prepper-plugins/aggregate-processor#action)。
*`actionHandleEventsDropped`：尚未从该事件中返回的事件数量`handleEvent` 致电配置[行动](https://github.com/opensearch-project/data-prepper/tree/main/data-prepper-plugins/aggregate-processor#action)。
*`actionHandleEventsProcessingErrors`：拨打电话的数量`handleEvent` 对于配置[行动](https://github.com/opensearch-project/data-prepper/tree/main/data-prepper-plugins/aggregate-processor#action) 这导致了错误。
*`actionConcludeGroupEventsOut`：已从`concludeGroup` 致电配置[行动](https://github.com/opensearch-project/data-prepper/tree/main/data-prepper-plugins/aggregate-processor#action)。
*`actionConcludeGroupEventsDropped`：尚未从该事件中返回的事件数量`condludeGroup` 致电配置[行动](https://github.com/opensearch-project/data-prepper/tree/main/data-prepper-plugins/aggregate-processor#action)。
*`actionConcludeGroupEventsProcessingErrors`：拨打电话的数量`concludeGroup` 对于配置[行动](https://github.com/opensearch-project/data-prepper/tree/main/data-prepper-plugins/aggregate-processor#action) 这导致了错误。

**测量**

*`currentAggregateGroups`：当前的组数。当一个小组结束并增加事件时，该量规会降低

