---
layout: default
title: 文本嵌入
parent: 摄入的处理器
nav_order: 260
redirect_from:
   - /api-reference/ingest-apis/processors/text-embedding/
---

# 文本嵌入

这`text_embedding` 处理器用于从文本字段生成向量嵌入[神经搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-search/)。

**先决条件**<br>
使用之前`text_embedding` 处理器，您必须设置机器学习（ML）模型。有关更多信息，请参阅[在OpenSearch中使用自定义模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/) 和[语义搜索]({{site.url}}{{site.baseurl}}/ml-commons-plugin/semantic-search/)。
{: .note}

以下是`text_embedding` 处理器：

```json
{
  "text_embedding": {
    "model_id": "<model_id>",
    "field_map": {
      "<input_field>": "<vector_field>"
    }
  }
}
```
{% include copy-curl.html %}

#### 配置参数

下表列出了所需的和可选参数`text_embedding` 处理器。

| 姓名| 数据类型| 必需的| 描述|
|:---|:---|:---|:---|
`model_id` | 细绳| 必需的| 将用于生成嵌入的模型的ID。该模型必须在Opensearch中部署，然后才能用于神经搜索。有关更多信息，请参阅[在OpenSearch中使用自定义模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/) 和[语义搜索]({{site.url}}{{site.baseurl}}/ml-commons-plugin/semantic-search/)。
`field_map` | 目的| 必需的| 包含密钥-值对将文本字段映射到向量字段的映射。
`field_map.<input_field>` | 细绳| 必需的| 获取用于生成文本嵌入文本的字段的名称。
`field_map.<vector_field>`  | 细绳| 必需的| 存储生成的文本嵌入的矢量字段的名称。
`description`  | 细绳| 选修的| 处理器的简要说明。|
`tag` | 细绳| 选修的| 处理器的标识符标签。可用于调试以区分同一类型的处理器。|

## 使用处理器

按照以下步骤在管道中使用处理器。创建处理器时必须提供模型ID。有关更多信息，请参阅[在OpenSearch中使用自定义模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/)。

**步骤1：创建管道。** 

以下示例请求创建了一个摄入的管道，其中文本来自`passage_text` 将转换为文本嵌入，嵌入将存储在`passage_embedding`：

```json
PUT /_ingest/pipeline/nlp-ingest-pipeline
{
  "description": "A text embedding pipeline",
  "processors": [
    {
      "text_embedding": {
        "model_id": "bQ1J8ooBpBj3wT4HVUsb",
        "field_map": {
          "passage_text": "passage_embedding"
        }
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
POST _ingest/pipeline/nlp-ingest-pipeline/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source":{
         "passage_text": "hello world"
      }
    }
  ]
}
```
{% include copy-curl.html %}

#### 回复

回答证实，除了`passage_text` 字段，处理器已经在该文本中生成了文本嵌入`passage_embedding` 场地：

```json
{
  "docs": [
    {
      "doc": {
        "_index": "testindex1",
        "_id": "1",
        "_source": {
          "passage_embedding": [
            -0.048237972,
            -0.07612712,
            0.3262124,
            ...
            -0.16352308
          ],
          "passage_text": "hello world"
        },
        "_ingest": {
          "timestamp": "2023-10-05T15:15:19.691345393Z"
        }
      }
    }
  ]
}
```

## 下一步

- 学习如何使用`neural` 查询文本搜索，请参阅[神经查询]({{site.url}}{{site.baseurl}}/query-dsl/specialized/neural/)。
- 要了解有关神经文本搜索的更多信息，请参阅[文字搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-text-search/)。
- 要了解有关在OpenSearch中使用模型的更多信息，请参见[在OpenSearch中使用自定义模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/)。
- 有关语义搜索教程，请参阅[语义搜索]({{site.url}}{{site.baseurl}}/ml-commons-plugin/semantic-search/)

