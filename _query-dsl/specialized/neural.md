---
layout: default
title: 神经
parent: 专业查询
grand_parent: 查询DSL
nav_order: 50
---

# 神经查询

使用`neural` 查询矢量字段搜索[神经搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-search/)。

## 请求字段

在`neural` 询问：

```json
"neural": {
  "<vector_field>": {
    "query_text": "<query_text>",
    "query_image": "<image_binary>",
    "model_id": "<model_id>",
    "k": 100
  }
}
```

顶端-等级`vector_field` 指定运行搜索查询的向量字段。下表列出了其他神经查询字段。

场地| 数据类型| 必需/可选| 描述
:--- | :--- | :--- 
`query_text` | 细绳| 选修的| 从中生成向量嵌入的查询文本。您必须至少指定一个`query_text` 或者`query_image`。
`query_image` | 细绳| 选修的| 基地-64编码的字符串对应于从中生成向量嵌入的查询图像。您必须至少指定一个`query_text` 或者`query_image`。
`model_id` | 细绳| 如果未设置默认模型ID，则需要。有关更多信息，请参阅[在索引或字段上设置默认模型]({{site.url}}{{site.baseurl}}/search-plugins/neural-text-search/#setting-a-default-model-on-an-index-or-field)。| 将用于从查询文本生成向量嵌入的模型的ID。该模型必须在Opensearch中部署，然后才能用于神经搜索。有关更多信息，请参阅[在OpenSearch中使用自定义模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/) 和[语义搜索]({{site.url}}{{site.baseurl}}/ml-commons-plugin/semantic-search/)。
`k` | 整数| 选修的| K返回的结果数-nn搜索。默认值为10。

#### 示例请求

```json
GET /my-nlp-index/_search
{
  "query": {
    "neural": {
      "passage_embedding": {
        "query_text": "Hi world",
        "query_image": "iVBORw0KGgoAAAAN...",
        "k": 100
      }
    }
  }
}
```
{％包含副本-curl.html％

