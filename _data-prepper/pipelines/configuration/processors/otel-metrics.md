---
layout: default
title: otel_metrics
parent: 处理器
grand_parent: 管道
nav_order: 72
---

# otel_metrics

这`otel_metrics` 处理器序列化的集合`ExportMetricsServiceRequest` 从[Otel指标来源]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sources/otel-metrics-source/) 进入字符串记录的集合。

## 用法

首先，将以下处理器添加到您的`pipeline.yaml` 配置文件：

``` yaml
processor:
    - otel_metrics_raw_processor:
```
{％include copy.html％}

## 配置

您可以使用以下可选参数来配置直方图及其默认值。直方图通过将数据分组到存储桶中显示数值数据。您可以使用直方图存储桶查看由总事件计数和所有事件的汇总总和组织的事件集。有关更多详细信息，请参阅[Opentelemetry直方图](https://opentelemetry.io/docs/reference/specification/metrics/data-model/#histogram)。

| 范围| 默认值| 描述|
| ：---    | ：---    | ：---    |
| `calculate_histogram_buckets` | `True` | 是否计算直方图存储桶。|
| `calculate_exponential_histogram_buckets` | `True` | 是否计算指数直方图存储桶。|
| `exponential_histogram_max_allowed_scale` | `10` | 指数直方图计算中的最大允许比例。| 
| `flatten_attributes` | `False` | 是否要弄平`attributes` JSON数据中的字段。|

### calculate_histogram_buckets

如果`calculate_histogram_buckets` 没有设置`false`，然后以下`JSON` 文件将添加到每个直方图JSON中。如果`flatten_attributes` 被设定为`false`， 这`JSON` 指标的字符串格式不会更改属性字段。如果`flatten_attributes` 被设定为`true`，属性字段中的值放在父`JSON` 目的。默认值是`true`。请参阅以下内容`JSON` 例子：

```json
 "buckets": [
    {
      "min": 0.0,
      "max": 5.0,
      "count": 2
    },
    {
      "min": 5.0,
      "max": 10.0,
      "count": 5
    }
  ]
```

您可以创建直方图及其边界的详细表示。您可以使用以下参数来控制此功能`pipeline.yaml` 文件：

```yaml
  processor:
    - otel_metrics_raw_processor:
        calculate_histogram_buckets: true
        calculate_exponential_histogram_buckets: true
        exponential_histogram_max_allowed_scale: 10
        flatten_attributes: false
```
{％include copy.html％}

每个数组元素都描述一个水桶。每个桶包含下边界，上边界及其值计数。这是一种更详细的Opentelemetry表示形式的特定形式，是`JSON` 由`otel_metrics` 处理器。请参阅以下内容`JSON` 文件，每个文件都添加到每个`JSON` 直方图`otel_metrics` 处理器：

```json
 "explicitBounds": [
    5.0,
    10.0
  ],
   "bucketCountsList": [
    2,
    5
  ]
```



### calculate_exponential_histogram_buckets

如果`calculate_exponential_histogram_buckets` 被设定为`true` （默认设置），以下`JSON` 值添加到每个`JSON` 直方图：

```json

    "negativeBuckets": [
        {
        "min": 0.0,
        "max": 5.0,
        "count": 2
        },
        {
        "min": 5.0,
        "max": 10.0,
        "count": 5
        }
    ],
...
    "positiveBuckets": [
        {
        "min": 0.0,
        "max": 5.0,
        "count": 2
        },
        {
        "min": 5.0,
        "max": 10.0,
        "count": 5
        }
    ],
```

下列`JSON` 文件是OpenTelemetry表示形式的一种更详细的形式，由负和正积分组成，比例参数，偏移量和存储桶计数列表：


```json
    "negative": [
        1,
        2,
        3
    ],
    "positive": [
        1,
        2,
        3
    ],
    "scale" : -3,
    "negativeOffset" : 0,
    "positiveOffset" : 1
```


### exponential_histogram_max_allowed_scale

这`exponential_histogram_max_allowed_scale` 参数定义了指数直方图的最大比例。如果增加此参数，则将增加潜在的内存消耗。看到[Opentelemetry规范](https://github.com/open-telemetry/opentelemetry-proto/blob/main/opentelemetry/proto/metrics/v1/metrics.proto) 有关指数直方图及其计算复杂性的更多信息。

所有指数直方图，其比例比配置参数高于配置参数（默认情况下，`10`）丢弃并以错误级别记录。您可以检查数据Prepper创建的日志以查看`ERROR` 日志消息。

绝对比例值用于比较，因此`-11` 这是平等对待的`11` 超过了配置的值`10` 可以被丢弃。
{： 。笔记}

## 指标

下表描述了所有处理器共有的指标。

| 公制名称| 类型| 描述|
| ------------- | ---- | -----------|
| `recordsIn` | 柜台| 表示入口记录数量的公制。|
| `recordsOut` | 柜台| 表示出口记录的数量。|
| `timeElapsed` | 计时器| 表示执行记录期间经过的时间。|

