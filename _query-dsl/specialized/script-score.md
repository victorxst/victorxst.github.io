---
layout: default
title: 脚本得分
parent: 专业查询
grand_parent: 查询DSL
nav_order: 60
---

# 脚本得分查询

用一个`script_score` 查询使用脚本自定义分数计算。对于昂贵的评分功能，您可以使用`script_score` 查询仅针对已过滤的返回文档计算分数。

## 例子

例如，以下请求创建一个包含一个文档的索引：

```json
PUT testindex1/_doc/1
{
  "name": "John Doe",
  "multiplier": 0.5
}
```
{% include copy-curl.html %}

您可以使用`match` 查询以返回所有包含的文档`John` 在里面`name` 场地：

```json
GET testindex1/_search
{
  "query": {
    "match": {
      "name": "John"
    }
  }
}
```
{% include copy-curl.html %}

在响应中，文件1的分数为`0.2876821`：

```json
{
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": "testindex1",
        "_id": "1",
        "_score": 0.2876821,
        "_source": {
          "name": "John Doe",
          "multiplier": 0.5
        }
      }
    ]
  }
}
```

现在，让我们使用将分数计算为值的脚本来更改文档分数`_score` 字段乘以`multiplier` 场地。在以下查询中，您可以访问文档中文档的当前相关性分数`_score` 变量和`multiplier` 值为`doc['multiplier'].value`：

```json
GET testindex1/_search
{
  "query": {
    "script_score": {
      "query": {
        "match": { 
            "name": "John" 
        }
      },
      "script": {
        "source": "_score * doc['multiplier'].value"
      }
    }
  }
}
```
{% include copy-curl.html %}

在响应中，文件1的分数是原始分数的一半：

```json
{
  "took": 8,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0.14384104,
    "hits": [
      {
        "_index": "testindex1",
        "_id": "1",
        "_score": 0.14384104,
        "_source": {
          "name": "John Doe",
          "multiplier": 0.5
        }
      }
    ]
  }
}
```

## 参数

这`script_score` 查询支持以下顶部-级别参数。

范围| 数据类型| 描述
:--- | :--- | :---
`query` | 目的| 用于搜索的查询。必需的。
`script` | 目的| 用于计算文档分数的脚本`query`。必需的。
`min_score` | 漂浮| 不包括低于分数的文件`min_score` 从结果。选修的。
`boost` | 漂浮| 通过给定的乘数提高文档的分数。小于1.0的值降低相关性，并且值大于1.0的相关性。默认值为1.0。

相关性分数由`script_score` 查询不能为负。
{： 。重要的}

## 用内置定制分数计算-在功能中

要自定义分数计算，您可以使用其中之一-在无痛功能中。对于每个功能，OpenSearch提供了一种或多种无痛方法，您可以在脚本分数上下文中访问。您可以直接调用以下各节中列出的无痛方法，而无需使用类名称或实例名称限定符。

### 饱和

饱和函数将饱和度计算为`score = value /(value + pivot)`， 在哪里`value` 是场值，`pivot` 选择以使得分数大于0.5`value` 大于`pivot` 如果不到0.5`value` 小于`pivot`。分数在（0，1）范围内。要应用饱和功能，请调用以下无痛方法：

- `double saturation(double <field-value>, double <pivot>)`
    
#### 例子

以下示例查询搜索文本`neural search` 在里面`articles` 指数。它结合了原始文档相关得分和`article_rank` 值，首先通过饱和函数转换：

```json
GET articles/_search
{
  "query": {
    "script_score": {
      "query": {
        "match": { "article_name": "neural search" }
      },
      "script" : {
        "source" : "_score + saturation(doc['article_rank'].value, 11)"
      }
    }
  }
}
```
{% include copy-curl.html %}

### 乙状结肠

与饱和函数类似，Sigmoid函数将分数计算为`score = value^exp/ (value^exp + pivot^exp)`， 在哪里`value` 是场值，`exp` 是指数缩放因素，并且`pivot` 选择以使得分数大于0.5`value` 大于`pivot` 如果不到0.5`value` 小于`pivot`。要应用Sigmoid功能，请调用以下无痛方法：

- `double sigmoid(double <field-value>, double <pivot>, double <exp>)`

#### 例子

以下示例查询搜索文本`neural search` 在里面`articles` 指数。它结合了原始文档相关得分和`article_rank` 值，首先通过Sigmoid函数转换：

```json
GET articles/_search
{
  "query": {
    "script_score": {
      "query": {
        "match": { "article_name": "neural search" }
      },
      "script" : {
        "source" : "_score + sigmoid(doc['article_rank'].value, 11, 2)"
      }
    }
  }
}
```
{% include copy-curl.html %}

### 随机分数

随机分数函数在[0，1）范围。要了解功能的工作原理，请参见[随机分数功能]({{site.url}}{{site.baseurl}}/query-dsl/compound/function-score#the-random-score-function)。要应用随机分数功能，请调用以下无痛方法之一：

- `double randomScore(int <seed>)`：使用内部Lucene文档ID作为种子值。
- `double randomScore(int <seed>, String <field-name>)`

#### 例子

以下查询使用`random_score` 功能`seed` 和`field`：

```json
GET articles/_search
{
  "query": {
    "script_score": {
      "query": {
        "match": { "article_name": "neural search" }
      },
      "script" : {
          "source" : "randomScore(20, '_seq_no')"
      }
    }
  }
}
```
{% include copy-curl.html %}

### 衰减功能

使用衰减功能，您可以根据接近度或新近度为结果得分。要了解更多，请参阅[衰减功能]({{site.url}}{{site.baseurl}}/query-dsl/compound/function-score#decay-functions)。您可以使用指数，高斯或线性衰减曲线来计算得分。要应用衰减功能，请根据字段类型调用以下无痛方法之一：

- [数字]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/numeric/) 字段：
    - `double decayNumericGauss(double <origin>, double <scale>, double <offset>, double <decay>, double <field-value>)`
    - `double decayNumericExp(double <origin>, double <scale>, double <offset>, double <decay>, double <field-value>)`
    - `double decayNumericLinear(double <origin>, double <scale>, double <offset>, double <decay>, double <field-value>)`
- [地理点]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/geo-point/) 字段：
    - `double decayGeoGauss(String <origin>, String <scale>, String <offset>, double <decay>, GeoPoint <field-value>)`
    - `double decayGeoExp(String <origin>, String <scale>, String <offset>, double <decay>, GeoPoint <field-value>)`
    - `double decayGeoLinear(String <origin>, String <scale>, String <offset>, double <decay>, GeoPoint <field-value>)`
- [日期]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/date/) 字段：
    - `double decayDateGauss(String <origin>, String <scale>, String <offset>, double <decay>, JodaCompatibleZonedDateTime <field-value>)`
    - `double decayDateExp(String <origin>, String <scale>, String <offset>, double <decay>, JodaCompatibleZonedDateTime <field-value>)`
    - `double decayDateLinear(String <origin>, String <scale>, String <offset>, double <decay>, JodaCompatibleZonedDateTime <field-value>)`

#### 示例：数字字段

以下查询在数字字段上使用指数衰减功能：

```json
GET articles/_search
{
  "query": {
    "script_score": {
      "query": {
        "match": {
          "article_name": "neural search"
        }
      },
      "script": {
        "source": "decayNumericExp(params.origin, params.scale, params.offset, params.decay, doc['article_rank'].value)",
        "params": {
          "origin": 50,
          "scale": 20,
          "offset": 30,
          "decay": 0.5
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

#### 示例：地理点字段

以下查询在地理点字段上使用高斯衰减功能：

```json
GET hotels/_search
{
  "query": {
    "script_score": {
      "query": {
        "match": {
          "name": "hotel"
        }
      },
      "script": {
        "source": "decayGeoGauss(params.origin, params.scale, params.offset, params.decay, doc['location'].value)",
        "params": {
          "origin": "40.71,74.00",
          "scale":  "300ft",
          "offset": "200ft",
          "decay": 0.25
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

#### 示例：日期字段

以下查询在日期字段上使用线性衰减功能：

```json
GET blogs/_search
{
  "query": {
    "script_score": {
      "query": {
        "match": {
          "name": "opensearch"
        }
      },
      "script": {
        "source": "decayDateLinear(params.origin, params.scale, params.offset, params.decay, doc['date_posted'].value)",
        "params": {
          "origin":  "2022-04-24",
          "scale":  "6d",
          "offset": "1d",
          "decay": 0.25
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

### 术语频率函数

术语频率函数公开术语-分数脚本源中的级别统计信息。您可以使用这些统计信息来实现自定义信息检索和排名算法，例如查询-时间乘法或添加分数通过受欢迎程度提高。要应用定期频率函数，请调用以下无痛方法之一：

- `int termFreq(String <field-name>, String <term>)`：检索字段中的术语频率以获取特定项。
- `long totalTermFreq(String <field-name>, String <term>)`：检索特定术语中字段中的总期限。
- `long sumTotalTermFreq(String <field-name>)`：检索字段中总期限的总和。

#### 例子

以下查询将分数计算为每个字段的总学期频率`fields` 列表乘以`multiplier` 价值：

```json
GET /demo_index_v1/_search
{
  "query": {
    "function_score": {
      "query": {
        "match_all": {}
      },
      "script_score": {
        "script": {
          "source": """
            for (int x = 0; x < params.fields.length; x++) {
              String field = params.fields[x];
              if (field != null) {
                return params.multiplier * totalTermFreq(field, params.term);
              }
            }
            return params.default_value;
            
          """,
          "params": {
              "fields": ["title", "description"],
              "term": "ai",
              "multiplier": 2,
              "default_value": 1
          }
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

如果[`search.allow_expensive_queries`]({{site.url}}{{site.baseurl}}/query-dsl/index/#expensive-queries) 被设定为`false`，`script_score` 查询未执行。
{： 。重要的

