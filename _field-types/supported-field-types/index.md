---
layout: default
title: 支持的字段类型
nav_order: 80
has_children: true
has_toc: false
redirect_from:
  - /opensearch/supported-field-types/
  - /opensearch/supported-field-types/index/
---

# 支持的字段类型

创建映射时，您可以为字段指定数据类型。下表列出了OpenSearch支持的所有数据字段类型。

类别| 现场类型和描述
:--- | :---
别名| [`alias`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/alias/)：现有字段的附加名称。
二进制| [`binary`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/binary/)：Base64编码中的二进制值。
[数字]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/numeric/) | 数字值（`byte`，`double`，`float`，`half_float`，`integer`，`long`，[`unsigned_long`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/unsigned-long/)，`scaled_float`，`short`）。
布尔| [`boolean`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/boolean/)：布尔值。
[日期]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/dates/)|  [`date`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/date/)：存储在毫秒中的日期。<br>[`date_nanos`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/date-nanos/)：存储在纳秒中的日期。
IP| [`ip`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/ip/)：IPv4或IPv6格式的IP地址。
[范围]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/range/) | 一系列值（`integer_range`，`long_range`，`double_range`，`float_range`，`date_range`，`ip_range`）。
[目的]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/object-fields/)| [`object`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/object/)：一个JSON对象。<br>[`nested`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/nested/)：当数组中的对象需要独立作为单独文档索引时使用。<br>[`flat_object`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/flat-object/)：一个被视为字符串的JSON对象。<br>[`join`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/join/)：建立父母-同一指数中的文件之间的儿童关系。
[细绳]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/string/)|[`keyword`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/keyword/)：包含一个未分析的字符串。<br>[`text`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/text/)：包含一个分析的字符串。<br>[`token_count`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/token-count/)：将分析令牌的数量存储在字符串中。
[自动完成]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/autocomplete/) |[`completion`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/completion/)：通过完成建议提供自动完成功能。<br>[`search_as_you_type`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/search-as-you-type/)：提供搜索-作为-你-使用前缀和Infix完成键入功能。
[地理]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/geographic/)| [`geo_point`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/geo-point/)：地理点。<br>[`geo_shape`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/geo-shape/)：地理形状。
[秩]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/rank/) | 提高或降低文档的相关得分（`rank_feature`，`rank_features`）。
[k-nn矢量]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/knn-vector/) | 允许索引k-nn向量进行开放搜索并执行不同类型的k-nn搜索。
渗滤剂| [`percolator`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/percolator/)：指定将此字段视为查询。

## 数组

OpenSearch中没有专用数组字段类型。相反，您可以将值数组传递到任何字段中。数组中的所有值必须具有相同的字段类型。

```json
PUT testindex1/_doc/1
{
  "number": 1 
}

PUT testindex1/_doc/2
{
  "number": [1, 2, 3] 
}
```

## 多场

多场用于不同的索引。字符串通常被映射为`text` 满足-文字查询和`keyword` 准确-值查询。

可以使用`fields` 范围。例如，您可以映射一本书`title` 类型`text` 并保持一个`title.raw` 类型子字段`keyword`。

```json
PUT books
{
  "mappings" : {
    "properties" : {
      "title" : {
        "type" : "text",
        "fields" : {
          "raw" : {
            "type" : "keyword"
          }
        }
      }
    }
  }
}
```

## 空值

将字段的值设置为`null`，一个空数组或一个数组`null` 值使此字段等效于一个空字段。因此，您无法搜索具有`null` 在这个领域里。

使一个字段可搜索`null` 值，您可以指定其`null_value` 索引映射中的参数。然后，全部`null` 传递给该字段的值将被指定的`null_value`。

这`null_value` 参数必须与字段相同。例如，如果您的字段是字符串，则`null_value` 对于此字段，也必须是字符串。
{: .note}

### 例子

创建映射以替换`null` 值`emergency_phone` 字符串字段"NONE"：

```json
PUT testindex
{
  "mappings": {
    "properties": {
      "name": {
        "type": "keyword"
      },
      "emergency_phone": {
        "type": "keyword",
        "null_value": "NONE" 
      }
    }
  }
}
```

将三个文档索引到TestIndex。这`emergency_phone` 文档1和3包含`null`，而`emergency_phone` 文件2的字段有一个空数组：

```json
PUT testindex/_doc/1
{
  "name": "Akua Mansa",
  "emergency_phone": null
}
```

```json
PUT testindex/_doc/2
{
  "name": "Diego Ramirez",
  "emergency_phone" : []
}
```

```json
PUT testindex/_doc/3 
{
  "name": "Jane Doe",
  "emergency_phone": [null, null]
}
```

搜索没有紧急电话的人：

```json
GET testindex/_search
{
  "query": {
    "term": {
      "emergency_phone": "NONE"
    }
  }
}
```

响应包含文档1和3，但不包含文件2，因为仅明确`null` 值替换为字符串"NONE"：

```json
{
  "took" : 1,
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
    "max_score" : 0.18232156,
    "hits" : [
      {
        "_index" : "testindex",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.18232156,
        "_source" : {
          "name" : "Akua Mansa",
          "emergency_phone" : null
        }
      },
      {
        "_index" : "testindex",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 0.18232156,
        "_source" : {
          "name" : "Jane Doe",
          "emergency_phone" : [
            null,
            null
          ]
        }
      }
    ]
  }
}
```

这`_source` 字段仍然包含明确的`null` 值是因为它不受`null_value`。
{: .note}

