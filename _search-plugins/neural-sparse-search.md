---
layout: default
title: 稀疏搜索
nav_order: 30
has_children: false
parent: 神经搜索
---

# 神经稀疏搜索
引入2.11
{: .label .label-purple }

[神经文本搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-text-search/) 依赖于基于文本嵌入模型的密集检索。但是，密集的方法使用k-NN搜索，它消耗了大量内存和CPU资源。替代神经文本搜索，使用倒置索引实现稀疏神经搜索，因此与BM25一样有效。稀疏的嵌入模型促进了稀疏搜索。当您执行稀疏搜索时，它会创建一个稀疏的向量（列表`token: weight` 钥匙-值对表示条目及其权重），并将数据摄入等级功能索引。

选择模型时，请选择以下选项之一：

- 在摄入时间和搜索时间（高性能，相对较高的延迟）都使用稀疏编码模型。
- 在摄入时间使用稀疏的编码模型，在搜索时间（低性能，相对较低的延迟）时使用令牌模型。

**先决条件**<br>
在使用稀疏搜索之前，请确保设置[验证的稀疏嵌入模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/pretrained-models/#sparse-encoding-models) 或您自己的稀疏嵌入模型。有关更多信息，请参阅[在OpenSearch中使用ML模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/) 和[连接到远程型号]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/index/)。
{:.note}

## 使用稀疏搜索

要使用稀疏搜索，请执行以下步骤：

1. [创建摄入管道](#step-1-create-an-ingest-pipeline)。
1. [创建摄入索引](#step-2-create-an-index-for-ingestion)。
1. [摄取索引的文档](#step-3-ingest-documents-into-the-index)。
1. [使用神经搜索搜索索引](#step-4-search-the-index-using-neural-search)。

## 步骤1：创建一个摄入管道

要生成向量嵌入，您需要创建一个[摄入管道]({{site.url}}{{site.baseurl}}/api-reference/ingest-apis/index/) 其中包含一个[`sparse_encoding` 处理器]({{site.url}}{{site.baseurl}}/api-reference/ingest-apis/processors/sparse-encoding/)，这将将文档字段中的文本转换为向量嵌入。处理器的`field_map` 确定从中生成向量嵌入的输入字段以及用于存储嵌入的输出字段。

以下示例请求创建了一个摄入的管道，其中文本来自`passage_text` 将转换为文本嵌入，嵌入将存储在`passage_embedding`：

```json
PUT /_ingest/pipeline/nlp-ingest-pipeline-sparse
{
  "description": "An sparse encoding ingest pipeline",
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

## 步骤2：创建摄入索引

为了使用管道中定义的文本嵌入处理器，请创建等级功能索引，并将上一步中创建的管道添加为默认管道。确保在`field_map` 被映射为正确的类型。继续以示例为例`passage_embedding` 字段必须映射为[`rank_features`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/rank/#rank-features)。同样，`passage_text` 字段应映射为`text`。

以下示例请求创建一个级别功能索引，该索引设置为默认的摄入管道：

```json
PUT /my-nlp-index
{
  "settings": {
    "default_pipeline": "nlp-ingest-pipeline-sparse"
  },
  "mappings": {
    "properties": {
      "id": {
        "type": "text"
      },
      "passage_embedding": {
        "type": "rank_features"
      },
      "passage_text": {
        "type": "text"
      }
    }
  }
}
```
{% include copy-curl.html %}

为了节省磁盘空间，您可以将嵌入向量从源中排除，如下所示：

```json
PUT /my-nlp-index
{
  "settings": {
    "default_pipeline": "nlp-ingest-pipeline-sparse"
  },
  "mappings": {
      "_source": {
      "excludes": [
        "passage_embedding"
      ]
    },
    "properties": {
      "id": {
        "type": "text"
      },
      "passage_embedding": {
        "type": "rank_features"
      },
      "passage_text": {
        "type": "text"
      }
    }
  }
}
```
{% include copy-curl.html %}

一旦`<token, weight>` 对从源头排除在外，无法恢复它们。在应用此优化之前，请确保您不需要`<token, weight>` 配对用于您的申请。
{: .important}

## 步骤3：将文档摄入索引

要将文档摄取到上一步中创建的索引中，请发送以下请求：

```json
PUT /my-nlp-index/_doc/1
{
  "passage_text": "Hello world",
  "id": "s1"
}
```
{% include copy-curl.html %}

```json
PUT /my-nlp-index/_doc/2
{
  "passage_text": "Hi planet",
  "id": "s2"
}
```
{% include copy-curl.html %}

在将文档摄入索引之前，摄入管道运行`sparse_encoding` 文档上的处理器，生成矢量嵌入`passage_text` 场地。索引文档包括`passage_text` 字段，其中包含原始文本，`passage_embedding` 字段，其中包含向量嵌入。

## 步骤4：使用神经搜索搜索索引

要在您的索引上执行稀疏的向量搜索，请使用`neural_sparse` 查询子句[查询DSL]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/index/) 查询。

以下示例请求使用`neural_sparse` 查询以搜索相关文档：

```json
GET my-nlp-index/_search
{
  "query": {
    "neural_sparse": {
      "passage_embedding": {
        "query_text": "Hi world",
        "model_id": "aP2Q8ooBpBj3wT4HVS8a",
        "max_token_score": 2
      }
    }
  }
}
```
{% include copy-curl.html %}

响应包含匹配文档：

```json
{
  "took" : 688,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 30.0029,
    "hits" : [
      {
        "_index" : "my-nlp-index",
        "_id" : "1",
        "_score" : 30.0029,
        "_source" : {
          "passage_text" : "Hello world",
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
          "id" : "s1"
        }
      },
      {
        "_index" : "my-nlp-index",
        "_id" : "2",
        "_score" : 16.480486,
        "_source" : {
          "passage_text" : "Hi planet",
          "passage_embedding" : {
            "hi" : 4.338913,
            "planets" : 2.7755864,
            "planet" : 5.0969057,
            "mars" : 1.7405145,
            "earth" : 2.6087382,
            "hello" : 3.3210192
          },
          "id" : "s2"
        }
      }
    ]
  }
}
```

