---
layout: default
title: 数据流
nav_order: 13
---

# 数据流

如果你将连续生成的时序数据（如日志、事件和指标）提取到 OpenSearch 中，则可能处于文档数量快速增长且无需更新旧文档的情况下。

管理时序数据的典型工作流涉及多个步骤，例如创建滚动更新索引别名、定义写入索引以及定义支持索引的通用映射和设置。

数据流简化了此过程，并强制实施了最适合时间序列数据的设置，例如主要为仅追加数据而设计，并确保每个文档都有一个时间戳字段。

数据流在内部由多个后备索引组成。搜索请求将路由到所有后备索引，而索引请求将路由到最新的写入索引。[ISM]({{site.url}}{{site.baseurl}}/im-plugin/ism/index/)策略允许你自动处理索引滚动更新或删除。


## 数据流入门

### 步骤 1：创建索引模板

若要创建数据流，首先需要创建一个索引模板，该模板将一组索引配置为数据流。该 `data_stream` 对象指示它是数据流，而不是常规索引模板。索引模式与数据流的名称匹配：

```json
PUT _index_template/logs-template
{
  "index_patterns": [
    "my-data-stream",
    "logs-*"
  ],
  "data_stream": {},
  "priority": 100
}
```

在这种情况下，每个引入的文档都必须有一个 `@timestamp` 字段。你还可以将自己的自定义时间戳字段定义为对象中的 `data_stream` 属性。你还可以在此处添加索引映射和其他设置，就像添加常规索引模板一样。

```json
PUT _index_template/logs-template-nginx
{
  "index_patterns": "logs-nginx",
  "data_stream": {
    "timestamp_field": {
      "name": "request_time"
    }
  },
  "priority": 200,
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0
    }
  }
}
```

在本例中， `logs-nginx` index 同时 `logs-template` 匹配和 `logs-template-nginx` 模板。当你出现平局时，OpenSearch 会选择具有较高优先级值的匹配索引模板。

### 步骤 2：创建数据流

创建索引模板后，你可以创建数据流。你可以使用数据流 API 显式创建数据流。数据流 API 初始化第一个后备索引：

```json
PUT _data_stream/logs-redis
PUT _data_stream/logs-nginx
```

你也可以直接开始摄取数据，而无需创建数据流。

由于我们有一个与 data_stream 对象匹配的索引模板，因此 OpenSearch 会自动创建数据流：

```json
POST logs-staging/_doc
{
  "message": "login attempt failed",
  "@timestamp": "2013-03-01T00:00:00"
}
```

要查看有关特定数据流的信息，请执行以下操作：

```json
GET _data_stream/logs-nginx
```

#### 响应示例

```json
{
  "data_streams" : [
    {
      "name" : "logs-nginx",
      "timestamp_field" : {
        "name" : "request_time"
      },
      "indices" : [
        {
          "index_name" : ".ds-logs-nginx-000001",
          "index_uuid" : "-VhmuhrQQ6ipYCmBhn6vLw"
        }
      ],
      "generation" : 1,
      "status" : "GREEN",
      "template" : "logs-template-nginx"
    }
  ]
}
```

你可以看到时间戳字段的名称、支持索引的列表以及用于创建数据流的模板。你还可以查看数据流的运行状况，它表示其所有支持索引的最低状态。

若要查看有关数据流的更多见解，请使用 `_stats` 终结点：

```json
GET _data_stream/logs-nginx/_stats
```

#### 响应示例

```json
{
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "data_stream_count" : 1,
  "backing_indices" : 1,
  "total_store_size_bytes" : 208,
  "data_streams" : [
    {
      "data_stream" : "logs-nginx",
      "backing_indices" : 1,
      "store_size_bytes" : 208,
      "maximum_timestamp" : 0
    }
  ]
}
```

若要查看有关所有数据流的信息，请使用以下请求：

```json
GET _data_stream
```

### 步骤 3：将数据引入数据流

若要将数据引入数据流，可以使用常规索引 API。确保索引的每个文档都有一个时间戳字段。如果尝试引入没有时间戳字段的文档，则会出现错误。

```json
POST logs-redis/_doc
{
  "message": "login attempt",
  "@timestamp": "2013-03-01T00:00:00"
}
```

### 步骤 4：搜索数据流

你可以像搜索常规索引或索引别名一样搜索数据流。搜索操作适用于所有后备索引（流中存在的所有数据）。

```json
GET logs-redis/_search
{
  "query": {
    "match": {
      "message": "login"
    }
  }
}
```

#### 响应示例

```json
{
  "took" : 514,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : ".ds-logs-redis-000001",
        "_type" : "_doc",
        "_id" : "-rhVmXoBL6BAVWH3mMpC",
        "_score" : 0.2876821,
        "_source" : {
          "message" : "login attempt",
          "@timestamp" : "2013-03-01T00:00:00"
        }
      }
    ]
  }
}
```

### 步骤 5：滚动更新数据流

滚动更新操作会创建一个新的后备索引，该索引将成为数据流的新写入索引。

要对数据流执行手动滚动更新操作，请执行以下操作：

```json
POST logs-redis/_rollover
```

#### 响应示例

```json
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "old_index" : ".ds-logs-redis-000001",
  "new_index" : ".ds-logs-redis-000002",
  "rolled_over" : true,
  "dry_run" : false,
  "conditions" : { }
}
```

如果现在对 `logs-redis` 数据流执行操作 `GET`，则会看到生成 ID 从 1 递增到 2。

你还可以设置一个[索引状态管理（ISM）策略]({{site.url}}{{site.baseurl}}/im-plugin/ism/policies/)自动执行数据流的滚动更新过程。ISM 策略在创建后备索引时应用于后备索引。当你将策略关联到数据流时，它只会影响该数据流的未来后备索引。

你也不需要提供设置 `rollover_alias`，因为 ISM 策略会从后备索引推断此信息。

### 步骤 6：在 OpenSearch 控制面板中管理数据流

要从 OpenSearch 控制面板管理数据流，请打开**OpenSearch 控制面板**、选择、选择**索引管理****指标**或**策略管理索引**。

你会看到数据流的切换开关，可用于显示或隐藏属于数据流的索引。

启用此开关后，你会看到一个数据流多选下拉菜单，可用于筛选数据流。你还会看到一个数据流列，其中显示包含索引的数据流的名称。

![数据流切换]({{site.url}}{{site.baseurl}}/images/data_streams_toggle.png)

你可以选择一个或多个数据流，并对其应用 ISM 策略。你还可以对任何单个支持索引应用策略。

你可以对数据流执行可视化，就像在常规索引或索引别名上执行可视化一样。

### 步骤 7：删除数据流

删除操作首先删除数据流的后备索引，然后删除数据流本身。

要删除数据流及其所有隐藏的支持索引，请执行以下操作：

```json
DELETE _data_stream/<name_of_data_stream>
```

你可以使用通配符删除多个数据流。

我们建议使用 ISM 策略从数据流中删除数据。

你还可以使用[异步搜索]({{site.url}}{{site.baseurl}}/search-plugins/async/index/)、[SQL]({{site.url}}{{site.baseurl}}/search-plugins/sql/index/)和[PPL]({{site.url}}{{site.baseurl}}/search-plugins/sql/ppl/index/)直接查询数据流。你还可以使用安全插件来定义数据流名称的精细权限。
