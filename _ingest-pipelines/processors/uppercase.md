---
layout: default
title: 大写
parent: 摄入的处理器
nav_order: 310
redirect_from:
   - /api-reference/ingest-apis/processors/uppercase/
---

# 大写
**引入1.0**
{: .label .label-purple }

这`uppercase` 处理器将特定字段中的所有文本转换为大写字母。

## 例子
以下是`uppercase` 处理器：

```json
{
  "uppercase": {
    "field": "field_name"
  }
}
```
{% include copy-curl.html %}

#### 配置参数

下表列出了所需的和可选参数`uppercase` 处理器。

| 姓名| 必需的| 描述|
|---|---|---|
`field`  | 必需的| 应附加数据的字段名称。支持模板片段。|
`description`  | 选修的| 处理器的简要说明。|
`if` | 选修的| 运行此处理器的条件。|
`ignore_failure` | 选修的| 如果设置为`true`，失败被忽略。默认为`false`。|
`ignore_missing`  | 选修的| 指定处理器是否应忽略没有指定字段的文档。默认为`false`。|
`on_failure` | 选修的| 如果处理器失败，则可以运行的处理器列表。|
`tag` | 选修的| 处理器的标识符标签。可用于调试以区分同一类型的处理器。|
`target_field`  | 选修的| 存储解析数据的字段名称。默认为`field`。默认情况下，`field` 已更新到位。|

## 使用处理器

按照以下步骤在管道中使用处理器。

**步骤1：创建管道。** 

以下查询创建了一个命名的管道`uppercase`，将文本转换为`field` 大写字段：

```json
PUT _ingest/pipeline/uppercase
{
  "processors": [
    {
      "uppercase": {
        "field": "name"
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
POST _ingest/pipeline/uppercase/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "name": "John"
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
          "name": "JOHN"
        },
        "_ingest": {
          "timestamp": "2023-08-28T19:54:42.289624792Z"
        }
      }
    }
  ]
}
```

**步骤3：摄取文档。**

以下查询将文档摄入到名为的索引中`testindex1`：

```json
PUT testindex1/_doc/1?pipeline=uppercase
{
  "name": "John"
}
```
{% include copy-curl.html %}

**步骤4（可选）：检索文档。**

要检索文档，请运行以下查询：

```json
GET testindex1/_doc/1
```
{% include copy-curl.html %}

