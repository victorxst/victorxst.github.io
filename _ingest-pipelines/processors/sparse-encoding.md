---
layout: default
title: 稀疏编码
parent: 摄入的处理器
nav_order: 240
redirect_from:
   - /api-reference/ingest-apis/processors/sparse-encoding/
---

# 稀疏编码

这`sparse_encoding` 处理器用于生成稀疏的向量/令牌，并从文本字段中的权重生成[神经搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-search/) 使用稀疏的检索。

**先决条件**<br>
使用之前`sparse_encoding` 处理器，您必须设置机器学习（ML）模型。有关更多信息，请参阅[在OpenSearch中使用自定义模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/) 和[语义搜索]({{site.url}}{{site.baseurl}}/ml-commons-plugin/semantic-search/)。
{: .note}

以下是`sparse_encoding` 处理器：

```json
{
  "sparse_encoding": {
    "model_id": "<model_id>",
    "field_map": {
      "<input_field>": "<vector_field>"
    }
  }
}
```
{% include copy-curl.html %}

#### 配置参数

下表列出了所需的和可选参数`sparse_encoding` 处理器。

| 姓名| 数据类型| 必需的| 描述|
|:---|:---|:---|:---|
`model_id` | 细绳| 必需的| 将用于生成嵌入的模型的ID。该模型必须在Opensearch中部署，然后才能用于神经搜索。有关更多信息，请参阅[在OpenSearch中使用自定义模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/) 和[语义搜索]({{site.url}}{{site.baseurl}}/ml-commons-plugin/semantic-search/)。
`field_map` | 目的| 必需的| 包含密钥-值对，将文本字段映射到一个`rank_features` 场地。
`field_map.<input_field>` | 细绳| 必需的| 获取用于生成矢量嵌入文本的字段的名称。
`field_map.<vector_field>`  | 细绳| 必需的| 存储生成的向量嵌入的向量字段的名称。
`description`  | 细绳| 选修的| 处理器的简要说明。|
`tag` | 细绳| 选修的| 处理器的标识符标签。可用于调试以区分同一类型的处理器。|

## 使用处理器

按照以下步骤在管道中使用处理器。创建处理器时必须提供模型ID。有关更多信息，请参阅[在OpenSearch中使用自定义模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/)。

**步骤1：创建管道。** 

以下示例请求创建了一个摄入的管道，其中文本来自`passage_text` 将转换为文本嵌入，嵌入将存储在`passage_embedding`：

```json
PUT /_ingest/pipeline/nlp-ingest-pipeline
{
  "description": "A sparse encoding ingest pipeline",
  "processors": [
    {
      "sparse_encoding": {
        "model_id": "aP2Q8ooBpBj3wT4HVS8a",
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
  "docs" : [
    {
      "doc" : {
        "_index" : "testindex1",
        "_id" : "1",
        "_source" : {
          "passage_embedding" : {
            "!" : 0.8708904,
            "door" : 0.8587369,
            "hi" : 2.3929274,
            "worlds" : 2.7839446,
            "yes" : 0.75845814,
            "##world" : 2.5432441,
            "born" : 0.2682308,
            "nothing" : 0.8625516,
            "goodbye" : 0.17146169,
            "greeting" : 0.96817183,
            "birth" : 1.2788506,
            "come" : 0.1623208,
            "global" : 0.4371151,
            "it" : 0.42951578,
            "life" : 1.5750692,
            "thanks" : 0.26481047,
            "world" : 4.7300377,
            "tiny" : 0.5462298,
            "earth" : 2.6555297,
            "universe" : 2.0308156,
            "worldwide" : 1.3903781,
            "hello" : 6.696973,
            "so" : 0.20279501,
            "?" : 0.67785245
          },
          "passage_text" : "hello world"
        },
        "_ingest" : {
          "timestamp" : "2023-10-11T22:35:53.654650086Z"
        }
      }
    }
  ]
}
```

## 下一步

- 学习如何使用`neural_sparse` 查询稀疏搜索，请参阅[神经稀疏查询]({{site.url}}{{site.baseurl}}/query-dsl/specialized/neural-sparse/)。
- 要了解有关稀疏神经搜索的更多信息，请参阅[稀疏搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-search/)。
- 要了解有关在OpenSearch中使用模型的更多信息，请参见[在OpenSearch中使用自定义模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/)。
- 有关语义搜索教程，请参阅[语义搜索]({{site.url}}{{site.baseurl}}/ml-commons-plugin/semantic-search/)。

