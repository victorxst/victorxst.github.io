---
layout: default
title: 术语设置
parent: 术语级查询
grand_parent: 查询DSL
nav_order: 90
---

# 术语设置查询

使用术语设置查询，您可以搜索在指定字段中匹配最少数量确切条款的文档。A`terms_set` 查询类似于`terms` 查询，除了您可以指定为返回文档所需的最小匹配条款数。您可以在索引中的字段或脚本中指定此数字。

例如，考虑一个索引，其中包含学生的名字和这些学生所接受的课程。在设置此索引的映射时，您需要提供一个[数字]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/numeric/) 指定为返回文档所需的最小匹配条款数量的字段：

```json
PUT students
{
  "mappings": {
    "properties": {
      "name": {
        "type": "keyword"
      },
      "classes": {
        "type": "keyword"
      },
      "min_required": {
        "type": "integer"
      }
    }
  }
}
```
{％包含副本-curl.html％}

接下来，索引两个与学生相对应的文档：

```json
PUT students/_doc/1
{
  "name": "Mary Major",
  "classes": [ "CS101", "CS102", "MATH101" ],
  "min_required": 2
}
```
{％包含副本-curl.html％}

```json
PUT students/_doc/2
{
  "name": "John Doe",
  "classes": [ "CS101", "MATH101", "ENG101" ],
  "min_required": 2
}
```
{％包含副本-curl.html％}

现在搜索至少参加以下两个课程的学生：`CS101`，，，，`CS102`，，，，`MATH101`：

```json
GET students/_search
{
  "query": {
    "terms_set": {
      "classes": {
        "terms": [ "CS101", "CS102", "MATH101" ],
        "minimum_should_match_field": "min_required"
      }
    }
  }
}
```
{％包含副本-curl.html％}

响应包含两个文档：

```json
{
  "took" : 44,
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
    "max_score" : 1.4544616,
    "hits" : [
      {
        "_index" : "students",
        "_id" : "1",
        "_score" : 1.4544616,
        "_source" : {
          "name" : "Mary Major",
          "classes" : [
            "CS101",
            "CS102",
            "MATH101"
          ],
          "min_required" : 2
        }
      },
      {
        "_index" : "students",
        "_id" : "2",
        "_score" : 0.5013843,
        "_source" : {
          "name" : "John Doe",
          "classes" : [
            "CS101",
            "MATH101",
            "ENG101"
          ],
          "min_required" : 2
        }
      }
    ]
  }
}
```

要指定文档应与脚本匹配的最小术语数，请在`minimum_should_match_script` 场地：

```json
GET students/_search
{
  "query": {
    "terms_set": {
      "classes": {
        "terms": [ "CS101", "CS102", "MATH101" ],
        "minimum_should_match_script": {
          "source": "Math.min(params.num_terms, doc['min_required'].value)"
        }
      }
    }
  }
}
```
{％包含副本-curl.html％}

## 参数

查询接受字段的名称（`<field>`）作为顶部-级别参数：

```json
GET _search
{
  "query": {
    "terms_set": {
      "<field>": {
        "terms": [ "term1", "term2" ],
        ... 
      }
    }
  }
}
```
{％包含副本-curl.html％}

这`<field>` 接受以下参数。除所有参数外`terms` 是可选的。

范围| 数据类型| 描述
：--- | ：--- | ：---
`terms` | 弦数| 在指定的字段中搜索的条款数组`<field>`。仅当所需的术语数与文档的字段值与正确的间距和资本化完全匹配时，结果才能在结果中返回。
`minimum_should_match_field` | 细绳| 名称[数字]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/numeric/) 指定为返回结果中返回文档所需的匹配项数的字段。
`minimum_should_match_script` | 细绳| 返回为返回结果中返回文档所需的匹配条款数量的脚本

