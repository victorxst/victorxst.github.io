---
layout: default
title: 索引汇总 API
parent: 索引汇总
nav_order: 9
---

# 索引汇总 API

使用索引汇总操作以编程方式处理索引汇总作业。

---

#### 目录
- 目录
{:toc}


---

## 创建或更新索引汇总作业
引入 1.0 {：.label .label-purple }

创建或更新索引汇总作业。你必须提供 `seq_no` 和 `primary_term` 参数。

#### 请求

```json
PUT _plugins/_rollup/jobs/<rollup_id> // Create
PUT _plugins/_rollup/jobs/<rollup_id>?if_seq_no=1&if_primary_term=1 // Update
{
  "rollup": {
    "source_index": "nyc-taxi-data",
    "target_index": "rollup-nyc-taxi-data",
    "schedule": {
      "interval": {
        "period": 1,
        "unit": "Days"
      }
    },
    "description": "Example rollup job",
    "enabled": true,
    "page_size": 200,
    "delay": 0,
    "roles": [
      "rollup_all",
      "nyc_taxi_all",
      "example_rollup_index_all"
    ],
    "continuous": false,
    "dimensions": [
      {
        "date_histogram": {
          "source_field": "tpep_pickup_datetime",
          "fixed_interval": "1h",
          "timezone": "America/Los_Angeles"
        }
      },
      {
        "terms": {
          "source_field": "PULocationID"
        }
      }
    ],
    "metrics": [
      {
        "source_field": "passenger_count",
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
      }
    ]
  }
}
```

你可以指定以下选项。

选项 | 描述 | 类型 | 必填
:--- | :--- |:--- |:--- |
 `source_index` | 检测器的名称。| 字符串 | 是的
 `target_index` | 指定将汇总数据引入到的目标索引。你可以创建新的目标索引，也可以使用现有索引。目标索引不能是原始数据和汇总数据的组合。此字段支持动态生成的索引名称，如 {% raw %}{% endraw %} `rollup_{{ctx.source_index}}`，其中 `source_index` 不能包含通配符。| 字符串 | 是的
 `schedule` | 索引汇总作业的计划，可以是间隔或 cron 表达式。| 对象 | 是的
 `schedule.interval`  | 指定汇总作业的执行频率。| 对象 | 不
 `schedule.interval.start_time` | 间隔的开始时间。| 时间戳 | 是的
 `schedule.interval.period` | 定义间隔周期。| 字符串 | 是的
 `schedule.interval.unit` | 指定间隔的时间单位。| 字符串 | 是的
 `schedule.interval.cron` |（可选）指定 cron 表达式以定义汇总频率。| 列表 | 不
 `schedule.interval.cron.expression` | 指定 Unix cron 表达式。| 字符串 | 是的
 `schedule.interval.cron.timezone` | 指定 IANA 时区数据库定义的时区。默认为 UTC。| 字符串 | 不
 `description` |（可选）描述汇总作业。| 字符串 | 不
 `enabled` | 如果为 true，则计划索引汇总作业。默认值为 true。| 布尔值 | 是的
 `continuous` | 指定索引汇总作业是永远连续汇总数据，还是只对当前数据集执行一次并停止。默认值为 false。| 布尔值 | 是的
 `error_notification` | 为错误通知设置 Mustache 消息模板。例如，如果索引汇总作业失败，系统会向 Slack 通道发送消息。| 对象 | 不
 `page_size` | 指定汇总期间一次要分页的存储桶数。| 编号 | 是的
 `delay` | 延迟执行索引汇总作业的毫秒数。| 长 | 不
 `dimensions` | 指定聚合以创建汇总时间窗口的维度。支持的组为 `terms`、 `histogram` 和 `date_histogram`。有关详细信息，请参见[存储桶聚合]({{site.url}}{{site.baseurl}}/opensearch/bucket-agg). | 阵列 | 是的
 `metrics` | 指定表示要计算的字段和指标的对象列表。支持的指标包括 `sum`、、 `max` `min` `value_count` 和 `avg`。有关详细信息，请参见[指标聚合]({{site.url}}{{site.baseurl}}/opensearch/metric-agg). | 阵列 | 不


#### 响应示例

```json
{
  "_id": "<rollup_id>",
  "_version": 3,
  "_seq_no": 1,
  "_primary_term": 1,
  "rollup": {
    "rollup_id": "<rollup_id>",
    "enabled": true,
    "schedule": {
      "interval": {
        "start_time": 1680159934649,
        "period": 1,
        "unit": "Days",
        "schedule_delay": 0
      }
    },
    "last_updated_time": 1680159934649,
    "enabled_time": 1680159934649,
    "description": "Example rollup job",
    "schema_version": 17,
    "source_index": "nyc-taxi-data",
    "target_index": "rollup-nyc-taxi-data",
    "metadata_id": null,
    "page_size": 200,
    "delay": 0,
    "continuous": false,
    "dimensions": [
      {
        "date_histogram": {
          "fixed_interval": "1h",
          "source_field": "tpep_pickup_datetime",
          "target_field": "tpep_pickup_datetime",
          "timezone": "America/Los_Angeles"
        }
      },
      {
        "terms": {
          "source_field": "PULocationID",
          "target_field": "PULocationID"
        }
      }
    ],
    "metrics": [
      {
        "source_field": "passenger_count",
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
      }
    ]
  }
}
```


## 获取索引汇总作业
引入 1.0 {：.label .label-purple }

返回有关 `rollup_id` 基于.

#### 请求

```json
GET _plugins/_rollup/jobs/<rollup_id>
```


#### 响应示例

```json
{
  "_id": "my_rollup",
  "_seqNo": 1,
  "_primaryTerm": 1,
  "rollup": { ... }
}
```


---

## 删除索引汇总作业
引入 1.0 {：.label .label-purple }

删除基于 `rollup_id`.

#### 请求

```json
DELETE _plugins/_rollup/jobs/<rollup_id>
```

#### 响应示例

```json
200 OK
```

---


## 启动或停止索引汇总作业
引入 1.0 {：.label .label-purple }

启动或停止索引汇总作业。

#### 请求

```json
POST _plugins/_rollup/jobs/<rollup_id>/_start
POST _plugins/_rollup/jobs/<rollup_id>/_stop
```


#### 响应示例

```json
200 OK
```


---

## 解释索引汇总作业
引入 1.0 {：.label .label-purple }

返回有关索引汇总作业及其当前进度的详细元数据信息。

#### 请求

```json
GET _plugins/_rollup/jobs/<rollup_id>/_explain
```


#### 响应示例

```json
{
  "example_rollup": {
    "rollup_id": "example_rollup",
    "last_updated_time": 1602014281,
    "continuous": {
      "next_window_start_time": 1602055591,
      "next_window_end_time": 1602075591
    },
    "status": "running",
    "failure_reason": null,
    "stats": {
      "pages_processed": 342,
      "documents_processed": 489359,
      "rollups_indexed": 3420,
      "index_time_in_ms": 30495,
      "search_time_in_ms": 584922
    }
  }
}
```
