---
layout: default
title: 定期频率
parent: 令牌过滤器
nav_order: 100
---

# 定期频率令牌过滤器

这`delimited_term_freq` 令牌滤波器根据提供的定界符将令牌流分隔为具有相应项频率的令牌。一个令牌由定界符之前的所有字符组成，术语频率是定界符之后的整数。例如，如果定界符是`|`，然后对于字符串`foo|5`，`foo` 是令牌和`5` 是其术语频率。如果没有定界符，则令牌过滤器不会修改术语频率。

您可以使用预配置`delimited_term_freq` 令牌过滤器或创建自定义过滤器。

## 预先配置`delimited_term_freq` 令牌过滤器

预配置`delimited_term_freq` 令牌过滤器使用`|` 默认分界符。要使用预配置的令牌过滤器分析文本，请将以下请求发送到`_analyze` 端点：

```json
POST /_analyze
{
  "text": "foo|100",
  "tokenizer": "keyword",
  "filter": ["delimited_term_freq"],
  "attributes": ["termFrequency"],
  "explain": true
}
```
{% include copy-curl.html %}

这`attributes` 数组指定要过滤的输出`explain` 仅返回的参数`termFrequency`。响应包含包含术语频率的令牌过滤器的原始令牌和解析输出：

```json
{
  "detail": {
    "custom_analyzer": true,
    "charfilters": [],
    "tokenizer": {
      "name": "keyword",
      "tokens": [
        {
          "token": "foo|100",
          "start_offset": 0,
          "end_offset": 7,
          "type": "word",
          "position": 0,
          "termFrequency": 1
        }
      ]
    },
    "tokenfilters": [
      {
        "name": "delimited_term_freq",
        "tokens": [
          {
            "token": "foo",
            "start_offset": 0,
            "end_offset": 7,
            "type": "word",
            "position": 0,
            "termFrequency": 100
          }
        ]
      }
    ]
  }
}
```

## 风俗`delimited_term_freq` 令牌过滤器

配置自定义`delimited_term_freq` 令牌过滤器，首先在映射请求中指定定界符，在此示例中，`^`：

```json
PUT /testindex
{
  "settings": {
    "analysis": {
      "filter": {
        "my_delimited_term_freq": {
          "type": "delimited_term_freq",
          "delimiter": "^"
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

然后使用您创建的自定义令牌过滤器分析文本：

```json
POST /testindex/_analyze
{
  "text": "foo^3",
  "tokenizer": "keyword",
  "filter": ["my_delimited_term_freq"],
  "attributes": ["termFrequency"],
  "explain": true
}
```
{% include copy-curl.html %}

响应包含具有频率一词的原始令牌和解析版本：

```json
{
  "detail": {
    "custom_analyzer": true,
    "charfilters": [],
    "tokenizer": {
      "name": "keyword",
      "tokens": [
        {
          "token": "foo|100",
          "start_offset": 0,
          "end_offset": 7,
          "type": "word",
          "position": 0,
          "termFrequency": 1
        }
      ]
    },
    "tokenfilters": [
      {
        "name": "delimited_term_freq",
        "tokens": [
          {
            "token": "foo",
            "start_offset": 0,
            "end_offset": 7,
            "type": "word",
            "position": 0,
            "termFrequency": 100
          }
        ]
      }
    ]
  }
}
```

## 结合`delimited_token_filter` 与脚本

您可以编写无痛脚本来计算结果中文档的自定义分数。

首先，创建索引并提供以下映射和设置：

```json
PUT /test
{
  "settings": {
    "number_of_shards": 1,
    "analysis": {
      "tokenizer": {
        "keyword_tokenizer": {
          "type": "keyword"
        }
      },
      "filter": {
        "my_delimited_term_freq": {
          "type": "delimited_term_freq",
          "delimiter": "^"
        }
      },
      "analyzer": {
        "custom_delimited_analyzer": {
          "tokenizer": "keyword_tokenizer",
          "filter": ["my_delimited_term_freq"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "f1": {
        "type": "keyword"
      },
      "f2": {
        "type": "text",
        "analyzer": "custom_delimited_analyzer",
        "index_options": "freqs"
      }
    }
  }
}
```
{% include copy-curl.html %}

这`test` 索引使用关键字令牌，这是一个定界术语频率令牌过滤器（定界符为`^`），以及一个包括关键字令牌和界限频率令牌过滤器的自定义分析仪。映射指定字段`f1` 是关键字字段和字段`f2` 是文本字段。场`f2` 使用设置中定义的自定义分析仪进行文本分析。另外，指定`index_options` 向OpenSearch进行信号，将术语频率添加到倒置索引中。您将使用术语频率将重复项的文档提供更高的分数。

接下来，使用批量上传索引两个文档：

```json
POST /_bulk?refresh=true
{"index": {"_index": "test", "_id": "doc1"}}
{"f1": "v0|100", "f2": "v1^30"}
{"index": {"_index": "test", "_id": "doc2"}}
{"f2": "v2|100"}
```
{% include copy-curl.html %}

以下查询搜索索引中的所有文档，并将文档分数计算为术语的术语频率`v1` 在该领域`f2`：

```json
GET /test/_search
{
  "query": {
    "function_score": {
      "query": {
        "match_all": {}
      },
      "script_score": {
        "script": {
          "source": "termFreq(params.field, params.term)",
          "params": {
            "field": "f2",
            "term": "v1"
          }
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

在响应中，文档1的得分为30，因为该术语的术语频率`v1` 在该领域`f2` IS 30.文件2的得分为0，因为该术语`v1` 没有出现在`f2`：

```json
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 30,
    "hits": [
      {
        "_index": "test",
        "_id": "doc1",
        "_score": 30,
        "_source": {
          "f1": "v0|100",
          "f2": "v1^30"
        }
      },
      {
        "_index": "test",
        "_id": "doc2",
        "_score": 0,
        "_source": {
          "f2": "v2|100"
        }
      }
    ]
  }
}
```

## 参数

下表列出了所有参数`delimited_term_freq` 支持。

范围| 必需/可选| 描述
:--- | :--- | :--- 
`delimiter` | 选修的| 定界符用于将令牌与术语频率分开。必须是一个非-空字符。默认为`|`

