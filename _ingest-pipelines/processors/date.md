---
layout: default
title: 日期
parent: 摄入的处理器
nav_order: 50
redirect_from:
   - /api-reference/ingest-apis/processors/date/
---

# 日期
**引入1.0**
{: .label .label-purple }

这`date` 处理器用于解析文档字段的日期，并将解析的数据添加到新字段中。默认情况下，解析数据存储在`@timestamp` 场地。

## 例子
以下是`date` 处理器：

```json
{
  "date": {
    "field": "date_field",
    "formats": ["yyyy-MM-dd'T'HH:mm:ss.SSSZZ"]
  }
}
```
{% include copy-curl.html %}

## 配置参数

下表列出了所需的和可选参数`date` 处理器。

范围| 必需的| 描述|
|-----------|-----------|-----------|
`field`  | 必需的| 应转换数据的字段名称。支持模板片段。|
`formats`  | 必需的| 预期日期格式的数组。可以是一个[日期格式]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/date/#formats) 或以下格式之一：ISO8601，UNIX，UNIX_MS或TAI64N。|
`description`  | 选修的| 处理器的简要说明。|
`if` | 选修的| 运行此处理器的条件。|
`ignore_failure` | 选修的| 如果设置为`true`，失败被忽略。默认为`false`。|
`locale`  | 选修的| 解析日期时要使用的语言环境。默认为`ENGLISH`。支持模板片段。|
`on_failure` | 选修的| 如果处理器失败，则可以运行的处理器列表。|
`output_format` | 选修的| 这[日期格式]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/date/#formats) 用于目标字段。默认为`yyyy-MM-dd'T'HH:mm:ss.SSSZZ`。|
`tag` | 选修的| 处理器的标识符标签。可用于调试以区分同一类型的处理器。|
`target_field`  | 选修的| 存储解析数据的字段名称。默认目标字段是`@timestamp`。| 
`timezone`  | 选修的| 解析日期时要使用的时区。默认为`UTC`。支持模板片段。|

## 使用处理器

按照以下步骤在管道中使用处理器。

**步骤1：创建管道。**

以下查询创建了一个命名的管道`date-output-format`，使用`date` 从欧洲日期格式转换为美国日期格式的处理器，添加了新字段`date_us` 带有所需的`output_format`：

```json
PUT /_ingest/pipeline/date-output-format
{
  "description": "Pipeline that converts European date format to US date format",
  "processors": [
    {
      "date": {
        "field" : "date_european",
        "formats" : ["dd/MM/yyyy", "UNIX"],
        "target_field": "date_us",
        "output_format": "MM/dd/yyy",
        "timezone" : "UTC"
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
POST _ingest/pipeline/date-output-format/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "date_us": "06/30/2023",
        "date_european": "30/06/2023"
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
        "_index": "testindex1",
        "_id": "1",
        "_source": {
          "date_us": "06/30/2023",
          "date_european": "30/06/2023"
        },
        "_ingest": {
          "timestamp": "2023-08-22T17:08:46.275195504Z"
        }
      }
    }
  ]
}
```

**步骤3：摄取文档。**

以下查询将文档摄入到名为的索引中`testindex1`：

```json
PUT testindex1/_doc/1?pipeline=date-output-format
{
  "date_european": "30/06/2023"
}
```
{% include copy-curl.html %}

**步骤4（可选）：检索文档。**

要检索文档，请运行以下查询：

```json
GET testindex1/_doc/1
```
{% include copy-curl.html %}

