---
layout: default
title: 获取模型
parent: 模型API
grand_parent: ML Commons API
nav_order: 20
---

# 获取模型

要检索有关模型的信息，您可以：

- [通过ID获取模型](#get-a-model-by-id)
- [搜索模型](#search-for-a-model)

## 通过ID获取模型

您可以使用`model_id`。

有关此API的用户访问的信息，请参见[模型访问控制注意事项]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/model-apis/index/#model-access-control-considerations)。

## 路径和HTTP方法

```json
GET /_plugins/_ml/models/<model-id>
```

#### 示例请求

```json
GET /_plugins/_ml/models/N8AE1osB0jLkkocYjz7D
```
{% include copy-curl.html %}

#### 示例响应

```json
{
"name" : "all-MiniLM-L6-v2_onnx",
"algorithm" : "TEXT_EMBEDDING",
"version" : "1",
"model_format" : "TORCH_SCRIPT",
"model_state" : "LOADED",
"model_content_size_in_bytes" : 83408741,
"model_content_hash_value" : "9376c2ebd7c83f99ec2526323786c348d2382e6d86576f750c89ea544d6bbb14",
"model_config" : {
    "model_type" : "bert",
    "embedding_dimension" : 384,
    "framework_type" : "SENTENCE_TRANSFORMERS",
    "all_config" : """{"_name_or_path":"nreimers/MiniLM-L6-H384-uncased","architectures":["BertModel"],"attention_probs_dropout_prob":0.1,"gradient_checkpointing":false,"hidden_act":"gelu","hidden_dropout_prob":0.1,"hidden_size":384,"initializer_range":0.02,"intermediate_size":1536,"layer_norm_eps":1e-12,"max_position_embeddings":512,"model_type":"bert","num_attention_heads":12,"num_hidden_layers":6,"pad_token_id":0,"position_embedding_type":"absolute","transformers_version":"4.8.2","type_vocab_size":2,"use_cache":true,"vocab_size":30522}"""
},
"created_time" : 1665961344044,
"last_uploaded_time" : 1665961373000,
"last_loaded_time" : 1665961815959,
"total_chunks" : 9
}
```

## 搜索模型

使用此命令搜索您已经创建的模型。

响应将仅包含您可以访问的那些模型版本。例如，如果您发送匹配所有查询，则将返回以下模型组类型的模型版本：

- 索引中的所有公共模型组。
- 您是模型所有者的私人模型组。
- 模型组在您的后端角色中至少具有一个后端角色。

有关更多信息，请参阅[模型访问控制]({{site.url}}{{site.baseurl}}/ml-commons-plugin/model-access-control/)。

### 路径和HTTP方法

```json
GET /_plugins/_ml/models/_search
POST /_plugins/_ml/models/_search
```

#### 示例请求：搜索所有模型

```json
POST /_plugins/_ml/models/_search
{
  "query": {
    "match_all": {}
  },
  "size": 1000
}
```
{% include copy-curl.html %}

#### 示例请求：搜索使用算法的模型"FIT_RCF"

```json
POST /_plugins/_ml/models/_search
{
  "query": {
    "term": {
      "algorithm": {
        "value": "FIT_RCF"
      }
    }
  }
}
```
{% include copy-curl.html %}

#### 示例响应

```json
{
    "took" : 8,
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
      "max_score" : 2.4159138,
      "hits" : [
        {
          "_index" : ".plugins-ml-model",
          "_id" : "-QkKJX8BvytMh9aUeuLD",
          "_version" : 1,
          "_seq_no" : 12,
          "_primary_term" : 15,
          "_score" : 2.4159138,
          "_source" : {
            "name" : "FIT_RCF",
            "version" : 1,
            "content" : "xxx",
            "algorithm" : "FIT_RCF"
          }
        },
        {
          "_index" : ".plugins-ml-model",
          "_id" : "OxkvHn8BNJ65KnIpck8x",
          "_version" : 1,
          "_seq_no" : 2,
          "_primary_term" : 8,
          "_score" : 2.4159138,
          "_source" : {
            "name" : "FIT_RCF",
            "version" : 1,
            "content" : "xxx",
            "algorithm" : "FIT_RCF"
          }
        }
      ]
    }
  }
```
