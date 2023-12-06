---
layout: default
title: 附加
parent: 摄入的处理器
nav_order: 10
redirect_from:
   - /api-reference/ingest-apis/processors/append/
---
 
# 附加
**引入1.0**
{: .label .label-purple }

这`append` 处理器用于将值添加到字段：
- 如果字段是数组，则`append` 处理器将指定的值附加到该数组。
- 如果字段是标量字段，则`append` 处理器将其转换为数组，并将指定的值附加到该数组。
- 如果该字段不存在，则`append` 处理器创建一个带有指定值的数组。

### 例子
以下是`append` 处理器：

```json
{
  "append": {
    "field": "your_target_field",
    "value": ["your_appended_value"]
  }
}
```
{% include copy-curl.html %}

## 配置参数

下表列出了所需的和可选参数`append` 处理器。

范围| 必需的| 描述|
|-----------|-----------|-----------|
`field`  | 必需的| 应附加数据的字段名称。支持模板片段。|
`value`  | 必需的| 要附加的值。这可以是静态值或从现有字段得出的动态值。支持模板片段。| 
`description`  | 选修的| 处理器的简要说明。|
`if` | 选修的| 运行此处理器的条件。|
`ignore_failure` | 选修的| 如果设置为`true`，失败被忽略。默认为`false`。|
`on_failure` | 选修的| 如果处理器失败，则可以运行的处理器列表。|
`tag` | 选修的| 处理器的标识符标签。可用于调试以区分同一类型的处理器。|

## 使用处理器

按照以下步骤在管道中使用处理器。

**步骤1：创建管道。** 

以下查询创建了一个命名的管道`user-behavior`，它具有一个附加处理器。它附加了`page_view` 每个新文档的摄入到一个名称的数组字段中`event_types`：

```json
PUT _ingest/pipeline/user-behavior
{
  "description": "Pipeline that appends event type",
  "processors": [
    {
      "append": {
        "field": "event_types",
        "value": ["page_view"]
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
POST _ingest/pipeline/user-behavior/_simulate
{
  "docs":[
    {
      "_source":{
      }
    }
  ]
}
```
{% include copy-curl.html %}

#### 回复

以下响应证实了管道正常工作：

```json
{
  "docs": [
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "event_types": [
            "page_view"
          ]
        },
        "_ingest": {
          "timestamp": "2023-08-28T16:55:10.621805166Z"
        }
      }
    }
  ]
}
```

**步骤3：摄取文档。**

以下查询将文档摄入到名为的索引中`testindex1`：

```json
PUT testindex1/_doc/1?pipeline=user-behavior
{
}
```
{% include copy-curl.html %}

**步骤4（可选）：检索文档。**

要检索文档，请运行以下查询：

```json
GET testindex1/_doc/1
```
{% include copy-curl.html %}

因为该文档不包含`event_types` 字段，创建一个数组字段，并将事件附加到数组：

```json
{
  "_index": "testindex1",
  "_id": "1",
  "_version": 2,
  "_seq_no": 1,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "event_types": [
      "page_view"
    ]
  }
}
```

