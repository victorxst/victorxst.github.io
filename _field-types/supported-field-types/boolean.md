---
layout: default
title: 布尔
nav_order: 20
has_children: false
parent: 支持的字段类型
redirect_from:
  - /opensearch/supported-field-types/boolean/
  - /field-types/boolean/
---

# 布尔字段类型

布尔场类型采用`true` 或者`false` 值，或`"true"` 或者`"false"` 字符串。您也可以通过一个空字符串（`""`）代替`false` 价值。

## 例子

创建一个映射，其中a，b和c是布尔字段：

```json
PUT testindex
{
  "mappings" : {
    "properties" :  {
      "a" : {
        "type" : "boolean"
      },
      "b" : {
        "type" : "boolean"
      },
      "c" : {
        "type" : "boolean"
      }
    }
  }
}
```
{% include copy-curl.html %}

索引具有布尔值的文档：

```json
PUT testindex/_doc/1 
{
  "a" : true,
  "b" : "true",
  "c" : ""
}
```
{% include copy-curl.html %}

因此，`a` 和`b` 将设置为`true`， 和`c` 将设置为`false`。

搜索所有文档`c` 是错误的：

```json
GET testindex/_search 
{
  "query": {
      "term" : {
        "c" : false
    }
  }
}
```
{% include copy-curl.html %}

## 参数

下表列出了布尔字段类型接受的参数。所有参数都是可选的。

范围| 描述
:--- | :--- 
`boost` | 浮动-指定该字段对相关性分数的重量的点值。值高于1.0的值增加了该领域的相关性。 0.0至1.0之间的值降低了该场的相关性。默认值为1.0。
`doc_values` | 布尔值指定是否应将字段存储在磁盘上，以便将其用于聚合，排序或脚本。默认为`true`。
`index` | 布尔值指定是否应搜索该字段。默认为`true`。
`meta` | 接受该领域的元数据。
[`null_value`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/index#null-value) | 用于代替的值`null`。必须与字段相同。如果未指定此参数，则该字段在其值为时被视为丢失`null`。默认为`null`。
`store` | 布尔值指定是否应存储字段值，并且可以与_source字段分开检索。默认为`false`。

## 汇总和脚本中的布尔值

在布尔领域的汇总中，`key` 返回数字值（1`true` 或0`false`）， 和`key_as_string` 返回字符串（`"true"` 或者`"false"`）。脚本返回`true` 和`false` 对于布尔值。

### 例子

在现场运行术语聚合查询`a`：

```json
GET testindex/_search
{
  "aggs": {
    "agg1": {
      "terms": {
        "field": "a"
      }
    }
  },
  "script_fields": {
    "a": {
      "script": {
        "lang": "painless",
        "source": "doc['a'].value"
      }
    }
  }
}
```
{% include copy-curl.html %}

脚本返回的值`a` 作为`true`，`key` 返回的价值`a` 作为`1`， 和`key_as_string` 返回的价值`a` 作为`"true"`：

```json
{
  "took" : 1133,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "testindex",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "fields" : {
          "a" : [
            true
          ]
        }
      }
    ]
  },
  "aggregations" : {
    "agg1" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : 1,
          "key_as_string" : "true",
          "doc_count" : 1
        }
      ]
    }
  }
}
```

