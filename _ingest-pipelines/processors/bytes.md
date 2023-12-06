---
layout: default
title: 字节
parent: 摄入的处理器
nav_order: 20
redirect_from:
   - /api-reference/ingest-apis/processors/bytes/
---

# 字节
**引入1.0**
{: .label .label-purple }

这`bytes` 处理器转换人类-可读取的字节值与其在字节中的等效值。该字段可以是标量或数组。如果字段是标量，则将值转换并存储在字段中。如果字段是数组，则将转换数组的所有值。

### 例子
以下是`bytes` 处理器：

```json
{
    "bytes": {
        "field": "your_field_name"
    }
}
```
{% include copy-curl.html %}

## 配置参数

下表列出了所需的和可选参数`bytes` 处理器。

范围| 必需的| 描述|
|-----------|-----------|-----------|
`field`  | 必需的| 应该转换数据的字段的名称。支持模板片段。|
`description`  | 选修的| 处理器的简要说明。|
`if` | 选修的| 运行此处理器的条件。|
`ignore_failure` | 选修的| 如果设置为`true`，失败被忽略。默认为`false`。|
`ignore_missing`  | 选修的| 如果设置为`true`，如果字段不存在或为`null`。默认为`false`。|
`on_failure` | 选修的| 如果处理器失败，则可以运行的处理器列表。|
`tag` | 选修的| 处理器的标识符标签。可用于调试以区分同一类型的处理器。|
`target_field`  | 选修的| 存储解析数据的字段名称。如果未指定，该值将存储在适当的位置`field` 场地。默认为`field`。|

## 使用处理器

按照以下步骤在管道中使用处理器。

**步骤1：创建管道。** 

以下查询创建了一个命名的管道`file_upload`，有一个`bytes` 处理器。它转换`file_size` 在其字节等效上，并将其存储在一个名为的新字段中`file_size_bytes`：

```json
PUT _ingest/pipeline/file_upload
{
  "description": "Pipeline that converts file size to bytes",
  "processors": [
    {
      "bytes": {
        "field": "file_size",
        "target_field": "file_size_bytes"
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
POST _ingest/pipeline/file_upload/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "file_size_bytes": "10485760",
        "file_size":
          "10MB"
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
        "_index": "testindex1",
        "_id": "1",
        "_source": {
          "event_types": [
            "event_type"
          ],
          "file_size_bytes": "10485760",
          "file_size": "10MB"
        },
        "_ingest": {
          "timestamp": "2023-08-22T16:09:42.771569211Z"
        }
      }
    }
  ]
}
```

**步骤3：摄取文档。**

以下查询将文档摄入到名为的索引中`testindex1`：

```json
PUT testindex1/_doc/1?pipeline=file_upload
{
  "file_size": "10MB"
}
```
{% include copy-curl.html %}

**步骤4（可选）：检索文档。** 

要检索文档，请运行以下查询：

```json
GET testindex1/_doc/1
```
{% include copy-curl.html %}

