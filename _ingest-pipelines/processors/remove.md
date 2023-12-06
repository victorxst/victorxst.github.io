---
layout: default
title: 消除
parent: 摄入的处理器
nav_order: 230
redirect_from:
   - /api-reference/ingest-apis/processors/remove/
---

# 消除
**引入1.0**
{: .label .label-purple }

这`remove` 处理器用于从文档中删除字段。

## 例子
以下是`remove` 处理器：

```json
{
    "remove": {
        "field": "field_name"
    }
}
```
{% include copy-curl.html %}

#### 配置参数

下表列出了所需的和可选参数`remove` 处理器。

| 姓名| 必需的| 描述|
|---|---|---|
`field`  | 必需的| 应附加数据的字段名称。支持模板片段。|
`description`  | 选修的| 处理器的简要说明。|
`if` | 选修的| 运行此处理器的条件。|
`ignore_failure` | 选修的| 如果设置为`true`，失败被忽略。默认为`false`。|
`on_failure` | 选修的| 如果处理器失败，则可以运行的处理器列表。|
`tag` | 选修的| 处理器的标识符标签。可用于调试以区分同一类型的处理器。|

## 使用处理器

按照以下步骤在管道中使用处理器。

**步骤1：创建管道。** 

以下查询创建了一个命名的管道`remove_ip`，这消除了`ip_address` 文档中的字段：

```json
PUT /_ingest/pipeline/remove_ip
{
  "description": "Pipeline that excludes the ip_address field.",
  "processors": [
    {
      "remove": {
        "field": "ip_address"
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
POST _ingest/pipeline/remove_ip/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source":{
         "ip_address": "203.0.113.1",
         "name": "John Doe"
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
          "name": "John Doe"
        },
        "_ingest": {
          "timestamp": "2023-08-24T18:02:13.218986756Z"
        }
      }
    }
  ]
}
```

**步骤3：摄取文档。**

以下查询将文档摄入到名为的索引中`testindex1`：

```json
PPUT testindex1/_doc/1?pipeline=remove_ip
{
  "ip_address": "203.0.113.1",
  "name": "John Doe"
}
```
{% include copy-curl.html %}

**步骤4（可选）：检索文档。**

要检索文档，请运行以下查询：

```json
GET testindex1/_doc/1
```
{% include copy-curl.html %}

