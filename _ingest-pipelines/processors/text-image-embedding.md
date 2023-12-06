---
layout: default
title: 文本/图像嵌入
parent: 摄入的处理器
nav_order: 270
redirect_from:
   - /api-reference/ingest-apis/processors/text-image-embedding/
---

# 文本/图像嵌入

这`text_image_embedding` 处理器用于从文本和图像字段中生成合并的矢量嵌入[多模式神经搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-multimodal-search/)。

**先决条件**<br>
使用之前`text_image_embedding` 处理器，您必须设置机器学习（ML）模型。有关更多信息，请参阅[在OpenSearch中使用自定义模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/) 和[语义搜索]({{site.url}}{{site.baseurl}}/ml-commons-plugin/semantic-search/)。
{: .note}

以下是`text_image_embedding` 处理器：

```json
{
  "text_image_embedding": {
    "model_id": "<model_id>",
    "embedding": "<vector_field>",
    "field_map": {
      "text": "<input_text_field>",
      "image": "<input_image_field>"
    }
  }
}
```
{% include copy-curl.html %}

## 参数

下表列出了所需的和可选参数`text_image_embedding` 处理器。

| 姓名| 数据类型| 必需的| 描述|
|:---|:---|:---|:---|
`model_id` | 细绳| 必需的| 将用于生成嵌入的模型的ID。该模型必须在Opensearch中部署，然后才能用于神经搜索。有关更多信息，请参阅[在OpenSearch中使用自定义模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/) 和[语义搜索]({{site.url}}{{site.baseurl}}/ml-commons-plugin/semantic-search/)。
`embedding` | 细绳| 必需的| 存储生成的嵌入的矢量字段的名称。两者都生成一个单个嵌入`text` 和`image` 字段。
`field_map` | 目的| 必需的| 包含密钥-指定生成嵌入的字段的值对。
`field_map.text` | 细绳| 选修的| 获取用于生成矢量嵌入文本的字段的名称。您必须至少指定一个`text` 或者`image`。
`field_map.image`  | 细绳| 选修的| 获取用于生成矢量嵌入图像的字段的名称。您必须至少指定一个`text` 或者`image`。
`description`  | 细绳| 选修的| 处理器的简要说明。|
`tag` | 细绳| 选修的| 处理器的标识符标签。可用于调试以区分同一类型的处理器。|

## 使用处理器

按照以下步骤在管道中使用处理器。创建处理器时必须提供模型ID。有关更多信息，请参阅[在OpenSearch中使用自定义模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/)。

**步骤1：创建管道。** 

以下示例请求创建了一个摄入的管道，其中文本来自`image_description` 以及来自`image_binary` 将转换为矢量嵌入，嵌入将存储在`vector_embedding`：

```json
PUT /_ingest/pipeline/nlp-ingest-pipeline
{
  "description": "A text/image embedding pipeline",
  "processors": [
    {
      "text_image_embedding": {
        "model_id": "bQ1J8ooBpBj3wT4HVUsb",
        "embedding": "vector_embedding",
        "field_map": {
          "text": "image_description",
          "image": "image_binary"
        }
      }
    }
  ]
}
```
{% include copy-curl.html %}

您可以在一个管道中设置多个处理器，以生成多个字段的嵌入式。
{: .note}

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
         "image_description": "Orange table",
         "image_binary": "bGlkaHQtd29rfx43..."
      }
    }
  ]
}
```
{% include copy-curl.html %}

#### 回复

回答证实，除了`image_description` 和`image_binary` 字段，处理器已经在`vector_embedding` 场地：

```json
{
  "docs": [
    {
      "doc": {
        "_index": "testindex1",
        "_id": "1",
        "_source": {
          "vector_embedding": [
            -0.048237972,
            -0.07612712,
            0.3262124,
            ...
            -0.16352308
          ],
          "image_description": "Orange table",
          "image_binary": "bGlkaHQtd29rfx43..."
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

- 学习如何使用`neural` 查询多模式搜索，请参阅[神经查询]({{site.url}}{{site.baseurl}}/query-dsl/specialized/neural/)。
- 要了解有关多模式神经搜索的更多信息，请参阅[多模式搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-multimodal-search/)。
- 要了解有关在OpenSearch中使用模型的更多信息，请参见[在OpenSearch中使用自定义模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/)。
- 有关语义搜索教程，请参阅[语义搜索]({{site.url}}{{site.baseurl}}/ml-commons-plugin/semantic-search/)

