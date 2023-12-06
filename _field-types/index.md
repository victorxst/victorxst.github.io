---
layout: default
title: 映射和字段类型
nav_order: 1
nav_exclude: true
redirect_from: 
  - /opensearch/mappings/
  - /field-types/mappings/
---

# 映射和字段类型

您可以通过创建_mapping_来定义如何存储和索引文档及其字段。该映射指定文档的字段列表。文档中的每个字段都有一个_field Type_，该字段对应于该字段所包含的数据类型。例如，您可能需要指定`year` 字段应为类型`date`。要了解更多，请参阅[支持的字段类型]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/index/)。

如果您刚刚开始构建群集和数据，则可能不知道应该如何存储数据。在这种情况下，您可以使用动态映射，该映射告诉Opensearch动态添加数据及其字段。但是，如果您确切地知道数据类型属于哪种类型并想要执行该标准，则可以使用明确的映射。

例如，如果您想指出`year` 应该是类型`text` 而不是一个`integer`， 和`age` 应该是一个`integer`，您可以通过明确的映射进行操作。通过使用动态映射，OpenSearch可能会解释`year` 和`age` 作为整数。

本节提供了一个如何创建索引映射以及如何向其添加文档的示例，以验证IP_range。

#### 目录
1. TOC
{：toc}


---
## Dynamic mapping

索引文档时，OpenSearch会自动添加字段，并使用动态映射。您还可以将字段明确添加到索引映射中。

#### Dynamic mapping types

类型| 描述
:--- | :---
无效的| A`null` 字段无法索引或搜索。当将字段设置为null时，OpenSearch的行为就像该字段没有值。
布尔| OpenSearch接受`true` 和`false` 作为布尔值。一个空字符串等于`false.`
漂浮| 一个-精度32-位浮点数。
双倍的| 双重-精度64-位浮点数。
整数| 签名的32-位号。
目的| 对象是标准的JSON对象，可以具有自己的字段和映射。例如，`movies` 对象可以具有其他属性，例如`title`，`year`， 和`director`。
大批| OpenSearch中的数组只能存储一种类型的值，例如仅整数或字符串的数组。空数组被对待，好像它们是没有值的字段。
文本| 一个代表完整字符的字符串序列-文本值。
关键词| 结构化字符的弦序列，例如电子邮件地址或邮政编码。
日期检测字符串| 默认启用，如果新字符串字段匹配日期的格式，则将字符串处理为`date` 场地。例如，`date: "2012/03/11"` 作为日期处理。
数字检测字符串| 如果禁用，OpenSearch可能会自动处理数字值作为字符串，那么它们应作为数字处理。启用后，OpenSearch可以将字符串处理到`long`，`integer`，`short`，`byte`，`double`，`float`，`half_float`，`scaled_float`， 和`unsigned_long`。默认值为禁用。

## Explicit mapping

如果您确切地知道您的字段数据类型需要什么，则可以在创建索引时在请求正文中指定它们。

```json
放置样本-索引1
{
  "mappings"：{
    "properties"：{
      "year"：{"type" ："text" }，，
      "age"：{"type" ："integer" }，，
      "director"：{"type" ："text" }
    }
  }
}
```

### Response
```json
{
    "acknowledged"： 真的，
    "shards_acknowledged"： 真的，
    "index"："sample-index1"
}
```

要将映射添加到现有索引或数据流中，您可以将请求发送到`_mapping` 使用端点`PUT` 或者`POST` HTTP方法：

```json
帖子样本-index1/_mapping
{
  "properties"：{
    "year"：{"type" ："text" }，，
    "age"：{"type" ："integer" }，，
    "director"：{"type" ："text" }
  }
}
```

您不能更改现有字段的映射，只能修改字段的映射参数。
{: .note}

---
## 映射示例用法

下面的示例显示了如何创建映射以指定OpenSearch应该忽略任何不符合符合畸形的IP地址的文档[`ip`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/ip/) 数据类型。您通过设置`ignore_malformed` 参数为`true`。

### 用一个`ip` 映射

要创建索引，请使用PUT请求：

```json
PUT /test-index 
{
  "mappings" : {
    "properties" :  {
      "ip_address" : {
        "type" : "ip",
        "ignore_malformed": true
      }
    }
  }
}
```

您可以将具有畸形IP地址的文档添加到您的索引：

```json
PUT /test-index/_doc/1 
{
  "ip_address" : "malformed ip address"
}
```

此索引的IP地址不会引发错误，因为`ignore_malformed` 设置为true。

您可以使用以下请求查询索引：

```json
GET /test-index/_search
```

回应表明`ip_address` 索引文档中忽略了字段：

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
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "test-index",
        "_id": "1",
        "_score": 1,
        "_ignored": [
          "ip_address"
        ],
        "_source": {
          "ip_address": "malformed ip address"
        }
      }
    ]
  }
}
```

## 获取映射

要获取一个或多个索引的所有映射，请使用以下请求：

```json
GET <index>/_mapping
```

在上述请求中，`<index>` 可能是索引名称或逗号-索引名称的分开列表。

要获取所有索引的所有映射，请使用以下请求：

```json
GET _mapping
```

要获取特定字段的映射，请提供索引名称和字段名称：

```json
GET _mapping/field/<fields>
GET /<index>/_mapping/field/<fields>
```

两个都`<index>` 和`<fields>` 可以指定为一个值或逗号-分开列表。

例如，以下请求检索`year` 和`age` 字段中`sample-index1`：

```json
GET sample-index1/_mapping/field/year,age
```

响应包含指定的字段：

```json
{
  "sample-index1" : {
    "mappings" : {
      "year" : {
        "full_name" : "year",
        "mapping" : {
          "year" : {
            "type" : "text"
          }
        }
      },
      "age" : {
        "full_name" : "age",
        "mapping" : {
          "age" : {
            "type" : "integer"
          }
        }
      }
    }
  }
}
```
