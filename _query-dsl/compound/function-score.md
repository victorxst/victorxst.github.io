---
layout: default
title: 功能分数
parent: 复合查询
grand_parent: 查询DSL
nav_order: 60
has_math: true
redirect_from:
  - /query-dsl/query-dsl/compound/function-score/
---

# 功能得分查询

用一个`function_score` 查询如果您需要更改结果中返回的文档的相关性得分。A`function_score` 查询定义一个查询和一个或多个功能，可以应用于结果的所有结果或子集以重新计算其相关性分数。

## 使用一个评分功能

一个最基本的例子`function_score` 查询使用一个函数来重新计算分数。以下查询使用`weight` 功能使所有相关得分加倍。此功能适用于结果中的所有文档，因为没有`query` 内部指定的参数`function_score`：

```json
GET shakespeare/_search
{
  "query": {
    "function_score": {
      "weight": "2"
    }
  }
}
```
{％包含副本-curl.html％}

## 将评分函数应用于文档子集

要将评分函数应用于文档子集，请在该函数中提供查询：

```json
GET shakespeare/_search
{
  "query": {
    "function_score": {
      "query": { 
        "match": {
          "play_name": "Hamlet"
        } 
      },
      "weight": "2"
    }
  }
}
```
{％包含副本-curl.html％}

## 支持的功能

这`function_score` 查询类型支持以下功能：

- 建造-在：
    - `weight`：将文档得分乘以预定义的增长因子。
    - `random_score`：提供一个随机分数，对于单个用户而言是一致但用户之间不同的分数。
    - `field_value_factor`：使用指定文档字段的值重新计算得分。
    - 衰减功能（`gauss`，，，，`exp`， 和`linear`）：使用指定的衰减功能重新计算得分。
- 风俗：
    - `script_score`：使用脚本来得分文档。

## 重量功能

当您使用`weight` 功能，原始相关得分乘以浮动-点值`weight`：

```json
GET shakespeare/_search
{
  "query": {
    "function_score": {
      "weight": "2"
    }
  }
}
```
{％包含副本-curl.html％}

不像`boost` 价值，`weight` 功能未归一化。

## 随机分数函数

这`random_score` 功能提供了一个随机分数，对于单个用户而言是一致但用户之间不同的分数。分数是浮动-[0，1）范围内的点号。默认情况下，`random_score` 函数使用内部Lucene文档ID作为种子值，使随机值不可验化，因为在合并后可以重新编写文档。为了实现生成随机值的一致性，您可以提供`seed` 和`field` 参数。这`field` 必须是一个领域`fielddata` 启用（通常是数字字段）。分数是使用`seed`， 这`fielddata` 值的值`field`，以及使用索引名称和碎片ID计算的盐。因为居住在同一碎片中的文档的索引名称和碎片ID相同，所以文档具有相同`field` 值将分配相同的分数。要确保同一碎片中所有文档的不同分数，请使用`field` 这对于所有文档都有独特的值。一种选择是使用`_seq_no` 场地。但是，如果您选择此字段，则由于文档的更新而可以更改分数`_seq_no` 更新。

以下查询使用`random_score` 功能`seed` 和`field`：

```json
GET blogs/_search
{
  "query": {
    "function_score": {
      "random_score": {
        "seed": 20,
        "field": "_seq_no"
      }
    }
  }
}
```
{％包含副本-curl.html％}

## 场值因子函数

这`field_value_factor` 功能使用指定文档字段的值重新计算得分。如果该字段是多人-有价值的字段，仅将其第一个值用于计算，而其他值则不考虑。

这`field_value_factor` 功能支持以下选项：

- `field`：用于分数计算的字段。

- `factor`：一个可选的因素，该因素将乘以乘以。默认值为1。

- `modifier`：适用于现场值$$ v $$的修饰符之一。下表列出了所有受支持的修饰符。
    
    修饰符| 公式| 描述
    ：--- | ：--- | ：---
    `log`| $$ \ log v $$| 拿起基地-值的10对数。进行非对数-正数是一个非法操作，将导致错误。对于0（独家）和1（包含）的值，此功能返回非-负值将导致错误。我们建议使用`log1p` 或者`log2p` 代替`log`。
    `log1p`| $$ \ log（1 + v）$$| 拿起基地-10和1和值的对数。
    `log2p`| $$ \ log（2 + v）$$| 拿起基地-10和2的对数和值。
    `ln`| $$ \ ln v $$| 取值的自然对数。进行非对数-正数是一个非法操作，将导致错误。对于0（独家）和1（包含）的值，此功能返回非-负值将导致错误。我们建议使用`ln1p` 或者`ln2p` 代替`ln`。
    `ln1p`| $$ \ ln（1 + v）$$| 取1和值的自然对数。
    `ln2p`| $$ \ ln（2 + v）$$| 取2和值的自然对数。
    `reciprocal`| $$ \ frac {1} {v} $$| 取值。
    `square`| $$ v^2 $$| 平方值。
    `sqrt`| $$ \ sqrt v $$| 取值的平方根。占负数的平方根是非法操作，将导致错误。确保$$ v $$是非-消极的。
    `none`| N/A。| 请勿应用任何修饰符。

- `missing`：如果文档中缺少字段，则使用的值。这`factor` 和`modifier` 应用于此值，而不是缺少字段值。

例如，以下查询使用`field_value_factor` 功能使重量更大`views` 场地：

```json
GET blogs/_search
{
  "query": {
    "function_score": {
      "field_value_factor": {
        "field": "views",
        "factor": 1.5,
        "modifier": "log1p",
        "missing": 1
      }
    }
  }
}
```
{％包含副本-curl.html％}

前面的查询使用以下公式计算相关得分：

$$ \ text {corce} = \ text {原始分数} \ cdot \ log（1 + 1.5 \ cdot \ text {views}）$$

## 脚本分数功能

使用`script_score` 功能，您可以编写一个定制脚本以进行评分文档，可选地将字段值合并到文档中。原始相关得分可在`_score` 多变的。

计算出的分数不能为负。负分数将导致错误。文档分数为正面32-位浮动-点值。更高精度的分数转换为最近的32-位浮动-点号。
{： 。重要的}

例如，以下查询使用`script_score` 根据原始分数以及博客文章的视图和喜欢的数量来计算分数的功能。为了给出视图数量并喜欢的重量较小，该公式会吸引视图和喜欢的对数。即使视图和喜欢的数量是，使对数有效`0`，，，，`1` 被添加到他们的总和中：

```json
GET blogs/_search
{
  "query": {
    "function_score": {
      "query": {"match": {"name": "opensearch"}},
      "script_score": {
        "script": "_score * Math.log(1 + doc['likes'].value + doc['views'].value)"
      }
    }
  }
}
```
{％包含副本-curl.html％}

编译和缓存脚本以更快的性能。因此，最好重复使用相同的脚本并传递脚本所需的任何参数：

```json
GET blogs/_search
{
  "query": {
    "function_score": {
      "query": {
        "match": { "name": "opensearch" }
      },
      "script_score": {
        "script": {
          "params": {
            "add": 1
          },
          "source": "_score * Math.log(params.add + doc['likes'].value + doc['views'].value)"
        }
      }
    }
  }
}
```
{％包含副本-curl.html％}

## 衰减功能

对于许多应用程序，您需要根据接近度或新近度对结果进行排序。您可以使用衰减功能来执行此操作。衰减功能使用三个衰减曲线之一计算文档得分：高斯，指数或线性。

衰减功能仅在[数字]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/numeric/)，，，，[日期]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/dates/)， 和[地理点]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/geo-point/) 字段。
{： 。重要的}

衰减功能根据`origin`，，，，`scale`，，，，`offset`， 和`decay`，如下图所示。

<img src="{{site.url}}{{site.baseurl}}/images/decay-functions.png" alt="Decay function curves" width="600">

### 示例：地理点字段

假设您正在寻找办公室附近的酒店。您创建一个`hotels` 绘制映射的索引`location` 字段作为地理点：

```json
PUT hotels
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_point"
      }
    }
  }
}
```
{％包含副本-curl.html％}

您为与附近酒店相对应的两个文档索引：

```json
PUT hotels/_doc/1
{
  "name": "Hotel Within 200",
  "location": { 
    "lat": 40.7105,
    "lon": 74.00
  }
}
```
{％包含副本-curl.html％}

```json
PUT hotels/_doc/2
{
  "name": "Hotel Outside 500",
  "location": { 
    "lat": 40.7115,
    "lon": 74.00
  }
}
```
{％包含副本-curl.html％}

这`origin` 定义计算距离的点（办公室位置）。这`offset` 指定距离文档的原点的距离为1。您可以在办公室200英尺以内的酒店以相同的最高分。这`scale` 定义图的衰减率，`decay` 定义分配给文档的分数`scale` +`offset` 距离原点的距离。一旦超出200英尺半径，您可能会决定如果您必须再走300英尺才能到达酒店（`scale` = 300英尺），您将分配原始分数的四分之一（`decay` = 0.25）。

您可以使用以下查询`origin` （74.00，40.71）：

```json
GET hotels/_search
{
  "query": {
    "function_score": {
      "functions": [
        {
          "exp": {
            "location": { 
              "origin": "40.71,74.00",
              "offset": "200ft",
              "scale":  "300ft",
              "decay": 0.25
            }
          }
        }
      ]
    }
  }
}
```
{％包含副本-curl.html％}

响应都包含两个酒店。该办公室200英尺以内的酒店的得分为1，500英尺半径以外的酒店的得分为0.20，该酒店比`decay` 参数0.25：

<详细信息打开降价="block">
  <summary>
    回复
  </summary>
  {： 。文本-三角洲}

```json
{
  "took": 854,
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
    "max_score": 1,
    "hits": [
      {
        "_index": "hotels",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "Hotel Within 200",
          "location": {
            "lat": 40.7105,
            "lon": 74
          }
        }
      },
      {
        "_index": "hotels",
        "_id": "2",
        "_score": 0.20099315,
        "_source": {
          "name": "Hotel Outside 500",
          "location": {
            "lat": 40.7115,
            "lon": 74
          }
        }
      }
    ]
  }
}
```
</delect>

### 参数

下表列出了由`gauss`，，，，`exp`， 和`linear` 功能。

范围| 描述
：--- | ：---
`origin` | 计算距离的点。必须作为数字字段的数字，日期字段的日期或地理点字段的地理点提供。地理点和数字字段所需。可选的日期字段（默认为`now`）。对于日期字段，支持日期数学（例如，`now-2d`）。
`offset` | 定义与文档的分数为1的距离的距离。可选。默认值为0。
`scale` | 远处的文件`scale` +`offset` 来自`origin` 分配得分`decay`。必需的。<br>对于数字字段，`scale` 可以是任何数字。<br>对于日期字段，`scale` 可以定义为一个数字[单位]({{site.url}}{{site.baseurl}}/api-reference/units/) （（`5h`，，，，`1d`）。如果未提供单位，`scale` 默认为毫秒。<br>对于地理点字段，`scale` 可以定义为一个数字[单位]({{site.url}}{{site.baseurl}}/api-reference/units/) （（`1mi`，，，，`5km`）。如果未提供单位，`scale` 默认为米。
`decay` | 定义文档的分数`scale` +`offset` 来自`origin`。选修的。默认值为0.5。

对于文档中缺少的字段，衰减功能返回分数为1。
{： 。笔记}

### 示例：数字字段

以下查询使用指数衰减函数来通过注释数量来确定博客文章：

```json
GET blogs/_search
{
  "query": {
    "function_score": {
      "functions": [
        {
          "exp": {
            "comments": { 
              "origin":  "20",
              "offset": "5",
              "scale":  "10"
            }
          }
        }
      ]
    }
  }
}
```
{％包含副本-curl.html％}

结果中的前两个博客帖子的得分为1，因为一个是原点（20），另一个是16的距离，在偏移范围内（文档接收到的完整分数的范围为20 $$ \ pm $$ 5，是[15，25]）。第三篇博客文章的距离`scale` +`offset` 来自`origin` （20＆减去;（5 + 10）= 15），因此给出了默认值`decay` 得分（0.5）：

<详细信息打开降价="block">
  <summary>
    回复
  </summary>
  {： 。文本-三角洲}

```json
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 4,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "blogs",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "Semantic search in OpenSearch",
          "views": 1200,
          "likes": 150,
          "comments": 16,
          "date_posted": "2022-04-17"
        }
      },
      {
        "_index": "blogs",
        "_id": "2",
        "_score": 1,
        "_source": {
          "name": "Get started with OpenSearch 2.7",
          "views": 1400,
          "likes": 100,
          "comments": 20,
          "date_posted": "2022-05-02"
        }
      },
      {
        "_index": "blogs",
        "_id": "3",
        "_score": 0.5,
        "_source": {
          "name": "Distributed tracing with Data Prepper",
          "views": 800,
          "likes": 50,
          "comments": 5,
          "date_posted": "2022-04-25"
        }
      },
      {
        "_index": "blogs",
        "_id": "4",
        "_score": 0.4352753,
        "_source": {
          "name": "A very old blog",
          "views": 100,
          "likes": 20,
          "comments": 3,
          "date_posted": "2000-04-25"
        }
      }
    ]
  }
}
```
</delect>

### 示例：日期字段

以下查询使用高斯衰减功能来确定2002年4月24日左右发布的博客文章：

```json
GET blogs/_search
{
  "query": {
    "function_score": {
      "functions": [
        {
          "gauss": {
            "date_posted": { 
              "origin":  "2022-04-24",
              "offset": "1d",
              "scale":  "6d",
              "decay": 0.25
            }
          }
        }
      ]
    }
  }
}
```
{％包含副本-curl.html％}

在结果中，第一篇博客文章在04/24/2022的一天内发表，因此它的分数最高为1。第二篇博客文章发表于04/17/2022`offset` +`scale` （（`1d` +`6d`），因此得分等于`decay` （0.25）。第三篇博客文章发表在04/24/2022之后的7天以上，因此得分较低。上一篇博客文章的分数为0，因为它是几年前出版的：

<详细信息打开降价="block">
  <summary>
    回复
  </summary>
  {： 。文本-三角洲}

```json
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 4,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "blogs",
        "_id": "3",
        "_score": 1,
        "_source": {
          "name": "Distributed tracing with Data Prepper",
          "views": 800,
          "likes": 50,
          "comments": 5,
          "date_posted": "2022-04-25"
        }
      },
      {
        "_index": "blogs",
        "_id": "1",
        "_score": 0.25,
        "_source": {
          "name": "Semantic search in OpenSearch",
          "views": 1200,
          "likes": 150,
          "comments": 16,
          "date_posted": "2022-04-17"
        }
      },
      {
        "_index": "blogs",
        "_id": "2",
        "_score": 0.15154076,
        "_source": {
          "name": "Get started with OpenSearch 2.7",
          "views": 1400,
          "likes": 100,
          "comments": 20,
          "date_posted": "2022-05-02"
        }
      },
      {
        "_index": "blogs",
        "_id": "4",
        "_score": 0,
        "_source": {
          "name": "A very old blog",
          "views": 100,
          "likes": 20,
          "comments": 3,
          "date_posted": "2000-04-25"
        }
      }
    ]
  }
}
```
</delect>

### 多-有价值的田地

如果您指定的衰减计算字段包含多个值，则可以使用`multi_value_mode` 范围。此参数指定以下功能之一来确定用于计算的字段值：

- `min`：（默认）距离`origin`。
- `max`：与`origin`。
- `avg`：与`origin`。
- `sum`：与`origin`。

例如，您为具有一系列距离的文档索引：

```json
PUT testindex/_doc/1
{
  "distances": [1, 2, 3, 4, 5]
}
```

以下查询使用`max` 多距离-有价值的领域`distances` 计算衰减：

```json
GET testindex/_search
{
  "query": {
    "function_score": {
      "functions": [
        {
          "exp": {
            "distances": { 
              "origin":  "6",
              "offset": "5",
              "scale":  "1"
            },
            "multi_value_mode": "max"
          }
        }
      ]
    }
  }
}
```
{％包含副本-curl.html％}

该文档的分数为1，因为距离原点的最大距离（1）在`offset` 来自`origin`：

```json
{
  "took": 3,
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
    "max_score": 1,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 1,
        "_source": {
          "distances": [
            1,
            2,
            3,
            4,
            5
          ]
        }
      }
    ]
  }
}
```

### 衰减曲线计算

以下公式定义了各种衰减功能的分数计算（$$ v $$表示文档字段值）。

**高斯**
    
$$ \ text {corce} = \ exp \ left（-\ frac {（\ max（0，\ lvert v- \ text {oneration} \ rvert- \ text {offset}））^2} {2 \ sigma^2} \ right），$$

计算$$ \ sigma $$以确保分数等于`decay` 在远处`offset` +`scale` 来自`origin`：

$$ \ sigma^2 =- \ frac {\ text {scale}^2} {2 \ ln（\ text {decay}）} $$

**指数**

$$ \ text {score} = \ exp（\ lambda \ cdot \ max（0，\ lvert v v）- \ text {oneration} \ rvert- \ text {offset}）），$$

在哪里计算$$ \ lambda $$，以确保分数等于`decay` 在远处`offset` +`scale` 来自`origin`：

$$ \ lambda = \ frac {\ ln（\ text {decay}）}} {\ text {scale}} $$

**线性**

$$ \ text {score} = \ max \ left（\ frac {s s s- \ max（0，\ lvert v- \ text {oneration} \ rvert- \ text {offset}）} {s} \ right），$$

计算$$ S $$以确保分数等于`decay` 在远处`offset` +`scale` 来自`origin`：

$$ s = \ frac {\ text {scale}} {1- \ text {decay}} $$

## 使用多个评分功能

您可以通过在功能分数查询中指定多个评分函数来通过在`functions` 大批。

### 组合来自多个功能的分数

不同的功能可以使用不同的量表进行评分。例如，`random_score` 功能提供了0到1之间的分数，但是`field_value_factor` 分数没有特定的量表。此外，您可能需要以不同的方式称量不同功能给出的分数。要调整不同功能的分数，您可以指定`weight` 每个功能的参数。然后将每个功能给出的分数乘以`weight` 产生该功能的最终分数。这`weight` 必须在`functions` 为了将其区分与[重量功能](#the-weight-function)，，，，

每个功能给出的分数都使用`score_mode` 参数，该参数采用以下值之一：

- `multiply`：（默认）分数乘以。
- `sum`：添加分数。
- `avg`：平均得分。如果`weight` 指定，这是[加权平均](https://en.wikipedia.org/wiki/Weighted_arithmetic_mean)。例如，如果重量$$ 1 $$的第一个功能将返回分数$$ 10 $$，而重量$$ 4 $$的第二个功能将返回分数$$ 20 $$，则平均值为$ \ frac{10 \ CDOT 1 + 20 \ CDOT 4} {1 + 4} = 18 $$。
- `first`：从具有匹配过滤器的第一个函数的分数。
- `max`：最高分数。
- `min`：最低分数。

### 指定分数的上限

您可以指定功能分数的上限`max_boost` 范围。默认的上限是A的最大幅度`float` 价值：（2＆minus; 2 <sup>＆sinus; 23 </sup>）＆middot;2 <sup> 127 </sup>。

### 将所有功能的分数与查询分数相结合

您可以指定使用所有功能计算的分数与查询分数合并`boost_mode` 参数，该参数采用以下值之一：

- `multiply`：（默认值）将查询分数乘以功能分数。
- `replace`：忽略查询分数并使用功能分数。
- `sum`：添加查询分数和功能分数。
- `avg`：平均查询分数和功能分数。
- `max`：取得更大的查询分数和功能分数。
- `min`：以查询分数和功能分数的较小。

### 过滤不符合阈值的文档

更改相关性分数不会更改匹配文档的列表。要排除某些不符合阈值的文档，请在`min_score` 范围。然后，使用阈值值对查询返回的所有文档进行评分和过滤。

### 例子

以下请求搜索包括单词的博客文章"OpenSearch Data Prepper"，更喜欢在04/24/2022左右发布的帖子。此外，考虑了视图和喜欢的数量。最后，截止阈值设定为10：

```json
GET blogs/_search
{
  "query": {
    "function_score": {
      "boost": "5", 
      "functions": [
        {
          "gauss": {
            "date_posted": {
              "origin": "2022-04-24",
              "offset": "1d",
              "scale": "6d"
            }
          }, 
          "weight": 1
        },
        {
          "gauss": {
            "likes": {
              "origin": 200,
              "scale": 200
            }
          }, 
          "weight": 4
        },
        {
          "gauss": {
            "views": {
              "origin": 1000,
              "scale": 800
            }
          }, 
          "weight": 2
        }
      ],
      "query": {
        "match": {
          "name": "opensearch data prepper"
        }
      },
      "max_boost": 10,
      "score_mode": "max",
      "boost_mode": "multiply",
      "min_score": 10
    }
  }
}
```
{％包含副本-curl.html％}

结果包含三个匹配的博客文章：

<详细信息打开降价="block">
  <summary>
    回复
  </summary>
  {： 。文本-三角洲}

```json
{
  "took": 14,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": 31.191923,
    "hits": [
      {
        "_index": "blogs",
        "_id": "3",
        "_score": 31.191923,
        "_source": {
          "name": "Distributed tracing with Data Prepper",
          "views": 800,
          "likes": 50,
          "comments": 5,
          "date_posted": "2022-04-25"
        }
      },
      {
        "_index": "blogs",
        "_id": "1",
        "_score": 13.907352,
        "_source": {
          "name": "Semantic search in OpenSearch",
          "views": 1200,
          "likes": 150,
          "comments": 16,
          "date_posted": "2022-04-17"
        }
      },
      {
        "_index": "blogs",
        "_id": "2",
        "_score": 11.150461,
        "_source": {
          "name": "Get started with OpenSearch 2.7",
          "views": 1400,
          "likes": 100,
          "comments": 20,
          "date_posted": "2022-05-02"
        }
      }
    ]
  }
}
```
</详细信息

