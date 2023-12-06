---
layout: default
title: 日期索引名称
parent: 摄入的处理器
nav_order: 55
---

# 日期索引名称

这`date_index_name` 处理器用于将文档指向正确的时间-基于文档中的日期或时间戳字段的基于索引。处理器设置`_index` 元数据领域[日期数学]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/date/#date-math) 索引名称表达式。然后处理器从`field` 文档中的字段正在处理，并将其格式化为日期数学索引名称表达式。提取的日期，`index_name_prefix` 价值和`date_rounding` 然后将值组合在一起以创建日期数学索引表达式。例如，如果`field` 字段包含值`2023-10-30T12:43:29.000Z` 和`index_name_prefix` 被设定为`week_index-` 和`date_rounding` 被设定为`w`，然后日期数学索引名称表达式为`week_index-2023-10-30`。您可以使用`date_formats` 字段以指定日期数学索引表达式中的日期应格式。

以下是`date_index_name` 处理器：

```json
{
  "date_index_name": {
    "field": "your_date_field or your_timestamp_field",
    "date_rounding": "rounding_value"
  }
}
```
{% include copy-curl.html %}

## 配置参数

下表列出了所需的和可选参数`date_index_name` 处理器。

范围| 必需/可选| 描述|
|-----------|-----------|-----------|
`field`  | 必需的| 传入文档中的日期或时间戳字段。支持[模板片段]({{site.url}}{{site.baseurl}}/ingest-pipelines/create-ingest/#template-snippets)。|
`date_rounding`  | 必需的| 索引名称中的圆形日期格式。有效值是`y` （年），`M` （月），`w` （星期），`d` （天），`h` （小时），`m` （分钟），然后`s` （第二）。|
`date_formats` | 选修的| 日期格式的数组用于解析日期或时间戳字段。有效的选项包括Java时间模式或以下格式之一：ISO8601，UNIX，UNIX_MS或TAI64N。默认为`yyyy-MM-dd'T'HH:mm:ss.SSSXX`。|
`index_name_format` | 选修的| 日期格式。默认为`yyyy-MM-dd`。支持[模板片段]({{site.url}}{{site.baseurl}}/ingest-pipelines/create-ingest/#template-snippets)。|
`index_name_prefix` | 选修的| 索引名称前缀在日期之前附加。支持[模板片段]({{site.url}}{{site.baseurl}}/ingest-pipelines/create-ingest/#template-snippets)。
`description`  | 选修的| 处理器的简要说明。|
`if` | 选修的| 运行此处理器的条件。|
`ignore_failure` | 选修的| 如果设置为`true`，失败被忽略。默认为`false`。|
`locale` | `locale`  | 选修的| 解析月份的名称和日期的工作日时要使用的语言环境。默认为`ENGLISH`。支持[模板片段]({{site.url}}{{site.baseurl}}/ingest-pipelines/create-ingest/#template-snippets)。|
`on_failure` | 选修的| 如果处理器失败，则可以运行的处理器列表。|
`tag` | 选修的| 处理器的标识符标签。可用于调试以区分同一类型的处理器。|
`timezone`  | 选修的| 解析日期时要使用的时区。默认为`UTC`。|

## 使用处理器

按照以下步骤在管道中使用处理器。

**步骤1：创建管道。**

以下查询创建了一个命名的管道`date-index-name1`，使用`date_index_name` 索引登录到每月索引的处理器：

```json
PUT /_ingest/pipeline/date-index-name1
{
  "description": "Create weekly index pipeline",
  "processors": [
    {
      "date_index_name": {
        "field": "date_field",
        "index_name_prefix": "week_index-",
        "date_rounding": "w",
        "date_formats": ["YYYY-MM-DD"]
      }
    }
  ]
}
```
{% include copy-curl.html %}

**步骤2（可选）：测试管道。**

建议您在摄入文档之前测试管道。
{: .tip}

要测试管道，请运行以下查询：

```json
POST _ingest/pipeline/date-index-name1/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "date_field": "2023-10-30"
      }
    }
  ]
}
```
{% include copy-curl.html %}

#### 回复

以下示例响应确认管道按预期工作：

```json
{
  "docs": [
    {
      "doc": {
        "_index": "<week_index-{2023-10-01||/w{yyyy-MM-dd|UTC}}>",
        "_id": "1",
        "_source": {
          "date_field": "2023-10-30"
        },
        "_ingest": {
          "timestamp": "2023-11-13T18:23:10.408593092Z"
        }
      }
    }
  ]
}
```

**步骤3：摄取文档。**

以下查询将文档摄入到名为的索引中`testindex1`：

```json
PUT testindex1/_doc/1?pipeline=date-index-name1
{
  "date_field": "2023-10-30"
}
```
{% include copy-curl.html %}

#### 回复

请求将文档索引到索引`week_index-2023-10-23` 并将在那一周内用时间戳将所有文档索引到同一指数，因为该管道每周都会回合。

```json
{
  "_index": "week_index-2023-10-30",
  "_id": "1",
  "_version": 4,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 3,
  "_primary_term": 1
}
```

**步骤4（可选）：检索文档。**

要检索文档，请运行以下查询：

```json
GET week_index-2023-10-30/_doc/1
```
{% include copy-curl.html %}

