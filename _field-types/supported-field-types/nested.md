---
layout: default
title: 嵌套
nav_order: 42
has_children: false
parent: Object字段类型
grand_grand_parent: 支持的字段类型
redirect_from:
  - /opensearch/supported-field-types/nested/
  - /field-types/nested/
---

# 嵌套字段类型

嵌套场类型是一种特殊类型[Object字段类型]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/object/)。

任何对象字段都可以采用一系列对象。数组中的每个对象都被动态映射为Object字段类型，并以扁平的形式存储。这意味着数组中的对象被分解为单个字段，并且在所有对象上的每个字段的值一起存储在一起。有时有必要使用嵌套类型保留整体嵌套对象，以便您可以对其进行搜索。

## 扁平的形式

默认情况下，每个嵌套对象都被动态映射为Object字段类型。任何对象字段都可以采用一系列对象。

```json
PUT testindex1/_doc/100
{ 
  "patients": [ 
    {"name" : "John Doe", "age" : 56, "smoker" : true},
    {"name" : "Mary Major", "age" : 85, "smoker" : false}
  ] 
}
```
{% include copy-curl.html %}

当这些对象存储时，它们会被扁平，因此它们的内部表示为每个字段的所有值数组：

```json
{
    "patients.name" : ["John Doe", "Mary Major"],
    "patients.age" : [56, 85],
    "patients.smoker" : [true, false]
}
```

在此表示中，一些查询将正确起作用。如果您搜索超过75岁或吸烟者的患者，则文件100应匹配。

```json
GET testindex1/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "patients.smoker": true
          }
        },
        {
          "range": {
            "patients.age": {
              "gte": 75
            }
          }
        }
      ]
    }
  }
}
```
{% include copy-curl.html %}

查询正确返回文档100：

```json
{
  "took" : 3,
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
    "max_score" : 1.3616575,
    "hits" : [
      {
        "_index" : "testindex1",
        "_type" : "_doc",
        "_id" : "100",
        "_score" : 1.3616575,
        "_source" : {
          "patients" : [
            {
              "name" : "John Doe",
              "age" : "56",
              "smoker" : true
            },
            {
              "name" : "Mary Major",
              "age" : "85",
              "smoker" : false
            }
          ]
        }
      }
    ]
  }
}
```

另外，如果您搜索超过75岁的患者和吸烟者，则文件100不应匹配。

```json
GET testindex1/_search 
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "patients.smoker": true
          }
        },
        {
          "range": {
            "patients.age": {
              "gte": 75
            }
          }
        }
      ]
    }
  }
}
```
{% include copy-curl.html %}

但是，此查询仍然错误地返回文档100。这是因为当创建了单个字段的值数组时，年龄和吸烟之间的关系丢失了。

## 嵌套字段类型

嵌套对象作为单独的文档存储，而父对象则引用其子女。要将对象标记为嵌套，请使用嵌套字段类型创建映射。

```json
PUT testindex1
{
  "mappings" : {
    "properties": {
      "patients": { 
        "type" : "nested"
      }
    }
  }
}
```
{% include copy-curl.html %}

然后，用嵌套字段类型索引文档：

```json
PUT testindex1/_doc/100
{ 
  "patients": [ 
    {"name" : "John Doe", "age" : 56, "smoker" : true},
    {"name" : "Mary Major", "age" : 85, "smoker" : false}
  ] 
}
```
{% include copy-curl.html %}

您可以使用以下嵌套查询来搜索75岁以上的患者或吸烟者：

```json
GET testindex1/_search
{
  "query": {
    "nested": {
      "path": "patients",
      "query": {
        "bool": {
          "should": [
            {
              "term": {
                "patients.smoker": true
              }
            },
            {
              "range": {
                "patients.age": {
                  "gte": 75
                }
              }
            }
          ]
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

查询正确返回两个患者：

```json
{
  "took" : 7,
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
    "max_score" : 0.8465736,
    "hits" : [
      {
        "_index" : "testindex1",
        "_id" : "100",
        "_score" : 0.8465736,
        "_source" : {
          "patients" : [
            {
              "name" : "John Doe",
              "age" : 56,
              "smoker" : true
            },
            {
              "name" : "Mary Major",
              "age" : 85,
              "smoker" : false
            }
          ]
        }
      }
    ]
  }
}
```

您可以使用以下嵌套查询来寻找75岁以上的患者和吸烟者：

```json
GET testindex1/_search
{
  "query": {
    "nested": {
      "path": "patients",
      "query": {
        "bool": {
          "must": [
            {
              "term": {
                "patients.smoker": true
              }
            },
            {
              "range": {
                "patients.age": {
                  "gte": 75
                }
              }
            }
          ]
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

前面的查询没有预期的结果，没有结果：

```json
{
  "took" : 7,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  }
}
```

## 参数

下表列出了Object字段类型接受的参数。所有参数都是可选的。

范围| 描述
：--- | ：--- 
[`dynamic`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/object#the-dynamic-parameter) | 指定是否可以将新字段动态添加到此对象中。有效值是`true`，`false`， 和`strict`。默认为`true`。
`include_in_parent` | 布尔值指定是否还应以扁平的形式添加子嵌套对象中的所有字段。默认为`false`。
`include_in_root` | 布尔值指定是否还应以扁平的形式添加子嵌套对象中的所有字段。默认为`false`。
`properties` | 该对象的字段，可以是任何受支持类型的字段。如果新属性可以动态添加到此对象中`dynamic` 被设定为`true`。

