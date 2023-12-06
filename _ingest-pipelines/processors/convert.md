---
layout: default
title: 转变
parent: 摄入的处理器
nav_order: 30
redirect_from:
   - /api-reference/ingest-apis/processors/convert/
---

# 转变
**引入1.0**
{: .label .label-purple }

这`convert` 处理器将文档中的字段转换为其他类型，例如，将字符串转换为整数或整数为字符串。对于数组字段，数组中的所有值都会转换。

## 例子
以下是`convert` 处理器：

```json
{
    "convert": {
        "field": "field_name",
        "type": "type-value"
    }
}
```
{% include copy-curl.html %}

## 配置参数

下表列出了所需的和可选参数`convert` 处理器。

范围| 必需的| 描述|
|-----------|-----------|-----------|
`field`  | 必需的| 包含要转换的数据的字段名称。支持模板片段。|
`type`  | 必需的| 将字段值转换为的类型。支持类型是`integer`，，，，`long`，，，，`float`，，，，`double`，，，，`string`，，，，`boolean`，，，，`ip`， 和`auto`。如果是`type` 是`boolean`，该值设置为`true` 如果字段值是字符串`true` （忽略案件）和`false` 如果字段值是字符串`false` （忽略案例）。如果该值不是允许的值之一，则会发生错误。|
`description`  | 选修的| 处理器的简要说明。|
`if` | 选修的| 运行此处理器的条件。|
`ignore_failure` | 选修的| 如果设置为`true`，失败被忽略。默认为`false`。|
`ignore_missing`  | 选修的| 如果设置为`true`，如果字段不存在或为`null`。默认为`false`。|
`on_failure` | 选修的| 如果处理器失败，则可以运行的处理器列表。|
`tag` | 选修的| 处理器的标识符标签。可用于调试以区分同一类型的处理器。|
`target_field`  | 选修的| 存储解析数据的字段名称。如果未指定，该值将存储在`field` 场地。默认为`field`。|

## 使用处理器

按照以下步骤在管道中使用处理器。

**步骤1：创建管道。** 

以下查询创建了一个命名的管道`convert-price`，转换`price` 浮动-点号，将转换值存储在`price_float` 字段，将值设置为`0` 如果小于`0`：

```json
PUT _ingest/pipeline/convert-price
{
  "description": "Pipeline that converts price to floating-point number and sets value to zero if price less than zero",
  "processors": [
    {
      "convert": {
        "field": "price",
        "type": "float",
        "target_field": "price_float"
      }
    },
    {
      "set": {
        "field": "price",
        "value": "0",
        "if": "ctx.price_float < 0"
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
POST _ingest/pipeline/convert-price/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
       "_source": {
        "price": "-10.5"
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
          "price_float": -10.5,
          "price": "0"
        },
        "_ingest": {
          "timestamp": "2023-08-22T15:38:21.180688799Z"
        }
      }
    }
  ]
}
```

**步骤3：摄取文档。**

以下查询将文档摄入到名为的索引中`testindex1`：

```json
PUT testindex1/_doc/1?pipeline=convert-price
{
  "price": "10.5"
}
```
{% include copy-curl.html %}

**步骤4（可选）：检索文档。**

要检索文档，请运行以下查询：

```json
GET testindex1/_doc/1
```
{% include copy-curl.html %}

