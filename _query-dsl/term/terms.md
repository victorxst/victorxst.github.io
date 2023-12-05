---
layout: default
title: Terms
parent: 术语级查询
grand_parent: 查询DSL
nav_order: 80
---

# terms查询

使用`terms` 查询在同一字段中搜索多个术语。例如，以下查询搜索具有IDS的行`61809` 和`61810`：

```json
GET shakespeare/_search
{
  "query": {
    "terms": {
      "line_id": [
        "61809",
        "61810"
      ]
    }
  }
}
```
{% include copy-curl.html %}

如果文档匹配数组中的任何条款，则将返回。

默认情况下，在`terms` 查询是65,536。要更改最大条款数，请更新`index.max_terms_count` 环境。

的能力[突出显示结果]({{site.url}}{{site.baseurl}}/search-plugins/searching-data/highlight/) 对于术语，可以保证查询，具体取决于荧光笔类型和查询中的条款数量。
{: .note}

## 参数

查询接受以下参数。所有参数都是可选的。

范围| 数据类型| 描述
:--- | :--- | :---
`<field>` | 细绳| 搜索的字段。仅当结果的字段值与正确的间距和资本化完全匹配时，仅在结果中返回文档。
`boost` | 漂浮的-观点| 通过给定的乘数增强查询。对于包含多个查询的搜索很有用。[0，1）中的值降低了相关性，并且值大于1的相关性。默认为`1`。

## 术语查找

术语查找可检索单个文档的字段值，并将其用作搜索术语。您可以使用术语查找来搜索大量术语。

要使用术语查找，您必须启用`_source` 映射字段是因为术语查找从文档中获取值。这`_source` 默认情况下启用字段。

术语查找试图从本地数据节点上的碎片中获取文档字段值。因此，使用单个主碎片的索引，该索引在所有适用的数据节点上具有完整的复制品可减少网络流量。

### 例子

例如，创建一个包含学生数据的索引，映射`student_id` 作为一个`keyword`：

```json
PUT students
{
  "mappings": {
    "properties": {
      "student_id": { "type": "keyword" }
    }
  }
}
```
{% include copy-curl.html %}

接下来，索引与学生相对应的三个文档：

```json
PUT students/_doc/1
{
  "name": "Jane Doe",
  "student_id" : "111"
}
```
{% include copy-curl.html %}

```json
PUT students/_doc/2
{
  "name": "Mary Major",
  "student_id" : "222"
}
```
{% include copy-curl.html %}

```json
PUT students/_doc/3
{
  "name": "John Doe",
  "student_id" : "333"
}
```
{% include copy-curl.html %}

创建一个包含类信息的单独索引，包括类名称和一系列与班级学生相对应的学生ID：

```json
PUT classes/_doc/101
{
  "name": "CS101",
  "enrolled" : ["111" , "222"]
}
```
{% include copy-curl.html %}

寻找入学的学生`CS101` 类，指定与类，该文档的索引以及术语所在的字段路径相对应的文档ID：

```json
GET students/_search
{
  "query": {
    "terms": {
      "student_id": {
        "index": "classes",
        "id": "101",
        "path": "enrolled"
      }
    }
  }
}
```
{% include copy-curl.html %}

响应包含文档`students` 每个ID匹配的学生的索引`enrolled` 大批：

```json
{
  "took": 13,
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
        "_index": "students",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "Jane Doe",
          "student_id": "111"
        }
      },
      {
        "_index": "students",
        "_id": "2",
        "_score": 1,
        "_source": {
          "name": "Mary Major",
          "student_id": "222"
        }
      }
    ]
  }
}
```

### 示例：嵌套字段

第二个示例演示了查询嵌套字段。考虑具有以下文档的索引：

```json
PUT classes/_doc/102
{
  "name": "CS102",
  "enrolled_students" : {
    "id_list" : ["111" , "333"]
  }
}
```
{% include copy-curl.html %}

寻找入学的学生`CS102`，使用点路径符号指定通往该字段的完整路径`path` 范围：

```json
ET students/_search
{
  "query": {
    "terms": {
      "student_id": {
        "index": "classes",
        "id": "102",
        "path": "enrolled_students.id_list"
      }
    }
  }
}
```
{% include copy-curl.html %}

响应包含匹配文档：

```json
{
  "took": 18,
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
        "_index": "students",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "Jane Doe",
          "student_id": "111"
        }
      },
      {
        "_index": "students",
        "_id": "3",
        "_score": 1,
        "_source": {
          "name": "John Doe",
          "student_id": "333"
        }
      }
    ]
  }
}
```

### 参数

下表列出了术语查找参数。

范围| 数据类型| 描述
:--- | :--- | :---
`index` | 细绳| 获取字段值的索引的名称。必需的。
`id` | 细绳| 从中获取字段值的文档的文档ID。必需的。
`path` | 细绳| 获取字段值的字段名称。使用点路径符号指定嵌套字段。必需的。
`routing` | 细绳| 从中获取字段值的文档的自定义路由值。选修的。如果在索引文档时提供了自定义路由值

