---
layout: default
title: CSV
parent: 摄入的处理器
nav_order: 40
redirect_from:
   - /api-reference/ingest-apis/processors/csv/
---

# CSV
**引入1.0**
{: .label .label-purple }

这`csv` 处理器用于解析CSV并将其存储在文档中。处理器忽略了空字段。

## 例子
以下是`csv` 处理器：

```json
{
  "csv": {
    "field": "field_name",
    "target_fields": ["field1, field2, ..."]
  }
}
```
{% include copy-curl.html %}

## 配置参数

下表列出了所需的和可选参数`csv` 处理器。

范围| 必需的| 描述|
|-----------|-----------|-----------|
`field`  | 必需的| 包含要转换的数据的字段名称。支持模板片段。|
`target_fields`  | 必需的| 存储解析数据的字段名称。|
`description`  | 选修的| 处理器的简要说明。|
`empty_value`  | 选修的| 表示不需要或不适用的可选参数。|
`if` | 选修的| 运行此处理器的条件。|
`ignore_failure` | 选修的| 如果设置为`true`，失败被忽略。默认为`false`。|
`ignore_missing`  | 选修的| 如果设置为`true`，如果该字段不存在，则处理器将不会失败。默认为`true`。| 
`on_failure` | 选修的| 如果处理器失败，则可以运行的处理器列表。|
`quote`  | 选修的| 用于在CSV数据中引用字段的字符。默认为`"`。|
`separator`  | 选修的| 定界符用于将CSV数据中的字段分开。默认为`,`。|
`tag` | 选修的| 处理器的标识符标签。可用于调试以区分同一类型的处理器。|
`trim`  | 选修的| 如果设置为`true`，处理器从文本的开头和结尾从修剪空白空间。默认为`false`。|

## 使用处理器

按照以下步骤在管道中使用处理器。

**步骤1：创建管道。**

以下查询创建了一个命名的管道`csv-processor`，那是分裂的`resource_usage` 分为三个名称的新领域`cpu_usage`，，，，`memory_usage`， 和`disk_usage`：

```json
PUT _ingest/pipeline/csv-processor
{
  "description": "Split resource usage into individual fields",
  "processors": [
    {
      "csv": {
        "field": "resource_usage",
        "target_fields": ["cpu_usage", "memory_usage", "disk_usage"],
        "separator": ","
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
POST _ingest/pipeline/csv-processor/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "resource_usage": "25,4096,10",
        "memory_usage": "4096",
        "disk_usage": "10",
        "cpu_usage": "25"
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
          "memory_usage": "4096",
          "disk_usage": "10",
          "resource_usage": "25,4096,10",
          "cpu_usage": "25"
        },
        "_ingest": {
          "timestamp": "2023-08-22T16:40:45.024796379Z"
        }
      }
    }
  ]
}
```

**步骤3：摄取文档。**

以下查询将文档摄入到名为的索引中`testindex1`：

```json
PUT testindex1/_doc/1?pipeline=csv-processor
{
  "resource_usage": "25,4096,10"
}
```
{% include copy-curl.html %}

**步骤4（可选）：检索文档。**

要检索文档，请运行以下查询：

```json
GET testindex1/_doc/1
```
{% include copy-curl.html %}

