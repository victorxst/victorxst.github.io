---
layout: default
title: 分析API
has_children: true
nav_order: 7
redirect_from:
  - /opensearch/rest-api/analyze-apis/
  - /api-reference/analyze-apis/
---

# 分析API
**引入1.0**
{: .label .label-purple }

分析API允许您执行[文本分析]({{site.url}}{{site.baseurl}}/api-reference/analyze-apis/)，这是将非结构化文本转换为供搜索优化的单个令牌（通常是单词）的过程。

分析API分析文本字符串并返回所得令牌。

如果使用安全插件，则必须`manage index` 特权。如果您只想分析文本，则必须`manage cluster` 特权。
{:.note}

## 路径和HTTP方法

```
GET /_analyze
GET /{index}/_analyze
POST /_analyze
POST /{index}/_analyze
```

尽管您可以使用两者都会发出分析请求`GET` 和`POST` 请求，两者具有重要的区别。A`GET` 请求会导致数据缓存在索引中，以便下次请求数据时，将其检索得更快。A`POST` 请求发送了一个尚未存在的字符串，该字符串与索引中已经存在的数据进行了比较。`POST` 请求没有缓存。
{:.note}

## 路径参数

您可以在请求中包含以下可选路径参数。

范围| 数据类型| 描述
:--- | :--- | :---
指数| 细绳| 用于得出分析仪的索引。

## 查询参数

您可以在请求中包含以下可选查询参数。

场地| 数据类型| 描述
:--- | :--- | :---
分析仪| 细绳| 分析仪的名称适用于`text` 场地。分析仪可以在索引中内置或配置。<br /> <br />如果`analyzer` 未指定，分析API使用在映射中定义的分析仪`field` 字段。<br /> <br />如果`field` 未指定字段，分析API使用索引的默认分析仪。<br /> <br>如果未指定索引或索引没有默认分析仪，则分析API使用标准分析仪。
属性| 弦数| 用于过滤输出的令牌属性数组`explain` 场地。
char_filter| 弦数| 字符过滤器数组，用于预处理字符之前`tokenizer` 场地。
解释| 布尔| 如果为真，则导致响应包括令牌属性和其他详细信息。默认为`false`。
场地| 细绳| 用于得出分析仪的字段。<br /> <br>如果指定`field`，您还必须指定`index` 路径参数。<br /> <br>如果指定`analyzer` 字段，它覆盖了`field`。<br /> <br>如果未指定`field`，分析API使用索引默认分析仪。<br /> <br>如果未指定`index` 字段或索引没有默认分析仪，分析API使用标准分析仪。
筛选| 弦数| 在`tokenizer` 场地。
归一化器| 细绳| 用于将文本转换为单个令牌的标准器。
令牌| 细绳| 用于转换的令牌`text` 字段进入令牌。

需要以下查询参数。

场地| 数据类型| 描述
:--- | :--- | :---
文本| 字符串或字符串| 文本要分析。如果您提供一系列字符串，则将文本分析为并发-价值字段。

#### 示例请求

[分析文本字符串阵列](#analyze-array-of-text-strings)

[应用建筑物-在分析仪中](#apply-a-built-in-analyzer)

[应用自定义分析仪](#apply-a-custom-analyzer)

[应用自定义瞬态分析仪](#apply-a-custom-transient-analyzer)

[指定索引](#specify-an-index)

[从索引字段得出分析仪](#derive-the-analyzer-from-an-index-field)

[指定归一化器](#specify-a-normalizer)

[获取令牌详细信息](#get-token-details)

[设置令牌限制](#set-a-token-limit)

#### 分析文本字符串阵列

当您将一系列字符串传递给`text` 字段，将其分析为多物-价值字段。

````json
GET /_analyze
{
  "analyzer" : "standard",
  "text" : ["first array element", "second array element"]
}
````
{% include copy-curl.html %}

上一个请求返回以下字段：

````json
{
  "tokens" : [
    {
      "token" : "first",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "array",
      "start_offset" : 6,
      "end_offset" : 11,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "element",
      "start_offset" : 12,
      "end_offset" : 19,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "second",
      "start_offset" : 20,
      "end_offset" : 26,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "array",
      "start_offset" : 27,
      "end_offset" : 32,
      "type" : "<ALPHANUM>",
      "position" : 4
    },
    {
      "token" : "element",
      "start_offset" : 33,
      "end_offset" : 40,
      "type" : "<ALPHANUM>",
      "position" : 5
    }
  ]
}
````

#### 应用建筑物-在分析仪中

如果您省略了`index` 路径参数，您可以应用任何内置的-在分析仪的文本字符串中。

以下请求使用`standard` 建造-在分析仪中：

````json
GET /_analyze
{
  "analyzer" : "standard",
  "text" : "OpenSearch text analysis"
}
````
{% include copy-curl.html %}

上一个请求返回以下字段：

````json
{
  "tokens" : [
    {
      "token" : "opensearch",
      "start_offset" : 0,
      "end_offset" : 10,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "text",
      "start_offset" : 11,
      "end_offset" : 15,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "analysis",
      "start_offset" : 16,
      "end_offset" : 24,
      "type" : "<ALPHANUM>",
      "position" : 2
    }
  ]
}
````

#### 应用自定义分析仪

您可以创建自己的分析仪并在分析请求中指定。

在这种情况下，自定义分析仪`lowercase_ascii_folding` 已创建并与`books2` 指数。分析仪将文本转换为小写，并转换-ASCII字符到ASCII。

以下请求将自定义分析仪应用于提供的文本：

````json
GET /books2/_analyze
{
  "analyzer": "lowercase_ascii_folding",
  "text" : "Le garçon m'a SUIVI."
}
````
{% include copy-curl.html %}

上一个请求返回以下字段：

````json
{
  "tokens" : [
    {
      "token" : "le",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "garcon",
      "start_offset" : 3,
      "end_offset" : 9,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "m'a",
      "start_offset" : 10,
      "end_offset" : 13,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "suivi",
      "start_offset" : 14,
      "end_offset" : 19,
      "type" : "<ALPHANUM>",
      "position" : 3
    }
  ]
}
````

#### 应用自定义瞬态分析仪

您可以从代币，令牌过滤器或角色过滤器中构建自定义瞬态分析仪。使用`filter` 参数以指定令牌过滤器。

以下请求使用`uppercase` 字符过滤器将文本转换为大写：

````json
GET /_analyze
{
  "tokenizer" : "keyword",
  "filter" : ["uppercase"],
  "text" : "OpenSearch filter"
}
````
{% include copy-curl.html %}

上一个请求返回以下字段：

````json
{
  "tokens" : [
    {
      "token" : "OPENSEARCH FILTER",
      "start_offset" : 0,
      "end_offset" : 17,
      "type" : "word",
      "position" : 0
    }
  ]
}
````
<hr />

以下请求使用`html_strip` 过滤以从文本中删除HTML字符：

````json
GET /_analyze
{
  "tokenizer" : "keyword",
  "filter" : ["lowercase"],
  "char_filter" : ["html_strip"],
  "text" : "<b>Leave</b> right now!"
}
````
{% include copy-curl.html %}

上一个请求返回以下字段：

```` json
{
  "tokens" : [
    {
      "token" : "leave right now!",
      "start_offset" : 3,
      "end_offset" : 23,
      "type" : "word",
      "position" : 0
    }
  ]
}
````

<hr />

您可以使用数组组合过滤器。

以下请求结合了`lowercase` 用`stop` 滤除了删除单词中的单词`stopwords` 大批：

````json
GET /_analyze
{
  "tokenizer" : "whitespace",
  "filter" : ["lowercase", {"type": "stop", "stopwords": [ "to", "in"]}],
  "text" : "how to train your dog in five steps"
}
````
{% include copy-curl.html %}

上一个请求返回以下字段：

````json
{
  "tokens" : [
    {
      "token" : "how",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "train",
      "start_offset" : 7,
      "end_offset" : 12,
      "type" : "word",
      "position" : 2
    },
    {
      "token" : "your",
      "start_offset" : 13,
      "end_offset" : 17,
      "type" : "word",
      "position" : 3
    },
    {
      "token" : "dog",
      "start_offset" : 18,
      "end_offset" : 21,
      "type" : "word",
      "position" : 4
    },
    {
      "token" : "five",
      "start_offset" : 25,
      "end_offset" : 29,
      "type" : "word",
      "position" : 6
    },
    {
      "token" : "steps",
      "start_offset" : 30,
      "end_offset" : 35,
      "type" : "word",
      "position" : 7
    }
  ]
}
````

#### 指定索引

您可以使用索引的默认分析仪分析文本，也可以指定其他分析仪。

以下请求使用与`books` 指数：

````json
GET /books/_analyze
{
  "text" : "OpenSearch analyze test"
}
````
{% include copy-curl.html %}

上一个请求返回以下字段：

````json

  "tokens" : [
    {
      "token" : "opensearch",
      "start_offset" : 0,
      "end_offset" : 10,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "analyze",
      "start_offset" : 11,
      "end_offset" : 18,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "test",
      "start_offset" : 19,
      "end_offset" : 23,
      "type" : "<ALPHANUM>",
      "position" : 2
    }
  ]
````

<hr />

以下请求使用`keyword` 分析仪，将整个文本值返回为一个令牌：

````json
GET /books/_analyze
{
  "analyzer" : "keyword",
  "text" : "OpenSearch analyze test"
}
````
{% include copy-curl.html %}

上一个请求返回以下字段：

````json
{
  "tokens" : [
    {
      "token" : "OpenSearch analyze test",
      "start_offset" : 0,
      "end_offset" : 23,
      "type" : "word",
      "position" : 0
    }
  ]
}
````

#### 从索引字段得出分析仪

您可以在索引中传递文本和字段。API查找了字段的分析仪，并使用它来分析文本。

如果映射不存在，则API使用标准分析仪，该标准分析仪将所有文本转换为基于空白空间的小写和标记。

以下请求导致分析基于映射`name`：

````json
GET /books2/_analyze
{
  "field" : "name",
  "text" : "OpenSearch analyze test"
}
````
{% include copy-curl.html %}

上一个请求返回以下字段：

````json
{
  "tokens" : [
    {
      "token" : "opensearch",
      "start_offset" : 0,
      "end_offset" : 10,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "analyze",
      "start_offset" : 11,
      "end_offset" : 18,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "test",
      "start_offset" : 19,
      "end_offset" : 23,
      "type" : "<ALPHANUM>",
      "position" : 2
    }
  ]
}
````

#### 指定归一化器

您可以使用与索引关联的标准式，而不是使用关键字字段。标准器导致分析变化产生单个令牌。

在此示例中，`books2` 索引包括一个标准器称为`to_lower_fold_ascii` 将文本转换为小写并翻译非-ASCII文本给ASCII。

以下请求适用`to_lower_fold_ascii` 到文字：

````json
GET /books2/_analyze
{
  "normalizer" : "to_lower_fold_ascii",
  "text" : "C'est le garçon qui m'a suivi."
}
````
{% include copy-curl.html %}

上一个请求返回以下字段：

````json
{
  "tokens" : [
    {
      "token" : "c'est le garcon qui m'a suivi.",
      "start_offset" : 0,
      "end_offset" : 30,
      "type" : "word",
      "position" : 0
    }
  ]
}
````

<hr />

您可以使用令牌和字符过滤器创建自定义的瞬态标准器。

以下请求使用`uppercase` 字符过滤器将给定文本转换为所有大写：

````json
GET /_analyze
{
  "filter" : ["uppercase"],
  "text" : "That is the boy who followed me."
}
````
{% include copy-curl.html %}

上一个请求返回以下字段：

````json
{
  "tokens" : [
    {
      "token" : "THAT IS THE BOY WHO FOLLOWED ME.",
      "start_offset" : 0,
      "end_offset" : 32,
      "type" : "word",
      "position" : 0
    }
  ]
}
````

#### 获取令牌详细信息

您可以通过设置所有令牌获得所有令牌的其他详细信息`explain` 属性为`true`。

以下请求提供了详细的令牌信息`reverse` 滤波器与`standard` Tokenizer：

````json
GET /_analyze
{
  "tokenizer" : "standard",
  "filter" : ["reverse"],
  "text" : "OpenSearch analyze test",
  "explain" : true,
  "attributes" : ["keyword"] 
}
````
{% include copy-curl.html %}

上一个请求返回以下字段：

````json
{
  "detail" : {
    "custom_analyzer" : true,
    "charfilters" : [ ],
    "tokenizer" : {
      "name" : "standard",
      "tokens" : [
        {
          "token" : "OpenSearch",
          "start_offset" : 0,
          "end_offset" : 10,
          "type" : "<ALPHANUM>",
          "position" : 0
        },
        {
          "token" : "analyze",
          "start_offset" : 11,
          "end_offset" : 18,
          "type" : "<ALPHANUM>",
          "position" : 1
        },
        {
          "token" : "test",
          "start_offset" : 19,
          "end_offset" : 23,
          "type" : "<ALPHANUM>",
          "position" : 2
        }
      ]
    },
    "tokenfilters" : [
      {
        "name" : "reverse",
        "tokens" : [
          {
            "token" : "hcraeSnepO",
            "start_offset" : 0,
            "end_offset" : 10,
            "type" : "<ALPHANUM>",
            "position" : 0
          },
          {
            "token" : "ezylana",
            "start_offset" : 11,
            "end_offset" : 18,
            "type" : "<ALPHANUM>",
            "position" : 1
          },
          {
            "token" : "tset",
            "start_offset" : 19,
            "end_offset" : 23,
            "type" : "<ALPHANUM>",
            "position" : 2
          }
        ]
      }
    ]
  }
}
````

#### 设置令牌限制

您可以将生成的令牌数设置为限制。设置较低的值会减少节点的内存使用量。默认值为10000。

以下请求将令牌限制为四：

````json
PUT /books2
{
  "settings" : {
    "index.analyze.max_token_count" : 4
  }
}
````
{% include copy-curl.html %}

前面的请求是索引API，而不是分析API。看[动态索引-级索引设置]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index-settings/#dynamic-index-level-index-settings) 有关其他详细信息。
{:.note}

### 响应字段

文本分析端点返回以下响应字段。

场地| 数据类型| 描述
:--- | :--- | :---
令牌| 大批| 从`text`。看[令牌对象](#token-object)。
细节| 目的| 有关分析和每个令牌的详细信息。仅当您请求令牌详细信息时才包括。看[细节对象](#detail-object)。

#### 令牌对象

场地| 数据类型| 描述
:--- | :--- | :---
令牌| 细绳| 令牌的文字。
start_offset| 整数| 令牌在原始文本字符串中的起始位置。偏移为零-基于。
end_offset| 整数| 令牌在原始文本字符串中的结尾位置。
类型| 细绳| 令牌的分类：`<ALPHANUM>`，`<NUM>`， 等等。令牌器通常设置该类型，但是有些过滤器定义了自己的类型。例如，同义词过滤器定义`<SYNONYM>` 类型。
位置|  整数| 令牌在`tokens` 大批。

#### 细节对象

场地| 数据类型| 描述
:--- | :--- | :---
custom_analyzer| 布尔| 分析仪是在文本上应用的还是内置的。
charlingters| 大批| 应用于文本的字符过滤器列表。
令牌| 目的| 将令牌的名称应用于文本，并在应用令牌过滤器之前使用内容的令牌<sup>*</sup>列表。
tokenfilters| 大批| 应用于文本的令牌过滤器列表。每个令牌过滤器都包含过滤器的名称和一个令牌<sup>*</sup>的列表，并在应用过滤器后具有内容。令牌过滤器按请求中指定的顺序列出。

看[令牌对象](#token-object) 对于令牌字段描述。
{:.note}

