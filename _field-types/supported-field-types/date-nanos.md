---
layout: default
title: 日期纳秒
nav_order: 35
has_children: false
parent: 日期字段类型
grand_parent: 支持的字段类型
---

# 日期纳秒字段类型

这`date_nanos` 现场类型类似于[`date`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/date/) 字段类型在其中保留日期。然而，`date` 将日期存储在毫秒的分辨率中，而`date_nanos` 将日期存储在纳秒分辨率中。日期存储为`long` 自时代以来与纳秒相对应的值。因此，支持日期的范围约为1970年--2262。

查询`date_nanos` 字段转换为字段值的范围查询`long` 表示。然后使用字段上的格式设置将存储的字段和聚合结果转换为字符串。

这`date_nanos` 字段支持一切[格式]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/date#formats) 和[参数]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/date#parameters) 那`date` 支持。您可以使用多种格式分开`||`。
{: .note}

为了`date_nanos` 字段，您可以使用`strict_date_optional_time_nanos` 格式以保留纳秒分辨率。如果您在将字段映射为`date_nanos`，默认格式为`strict_date_optional_time||epoch_millis` 这使您可以通过任何一个`strict_date_optional_time` 或者`epoch_millis` 格式。这`strict_date_optional_time` 格式支持纳秒分辨率中的日期，但`epoch_millis` 格式仅支持毫秒分辨率的日期。

## 例子

用`date` 类型字段`date_nanos` 那就是`strict_date_optional_time_nanos` 格式：

```json
PUT testindex/_mapping
{
  "properties": {
      "date": {
        "type": "date_nanos",
        "format" : "strict_date_optional_time_nanos"
      }
    }
}
```
{% include copy-curl.html %}

将两个文档索引到索引：

```json
PUT testindex/_doc/1
{ "date": "2022-06-15T10:12:52.382719622Z" }
```
{% include copy-curl.html %}

```json
PUT testindex/_doc/2
{ "date": "2022-06-15T10:12:52.382719624Z" }
```
{% include copy-curl.html %}

您可以使用范围查询搜索日期范围：

```json
GET testindex/_search
{
  "query": {
    "range": {
      "date": {
        "gte": "2022-06-15T10:12:52.382719621Z",
        "lte": "2022-06-15T10:12:52.382719623Z"
      }
    }
  }
}
```
{% include copy-curl.html %}

响应包含其日期在指定范围内的文档：

```json
{
  "took": 43,
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
          "date": "2022-06-15T10:12:52.382719622Z"
        }
      }
    ]
  }
}
```

查询文件时`date_nanos` 字段，您可以使用`fields` 或者`docvalue_fields`：

```json
GET testindex/_search
{
  "fields": ["date"]
}
```
{% include copy-curl.html %}

```json
GET testindex/_search
{
  "docvalue_fields" : [
    {
      "field" : "date"
    }
  ]
}
```
{% include copy-curl.html %}

对任何一个查询的响应都包含两个索引文档：

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
    "max_score": 1,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 1,
        "_source": {
          "date": "2022-06-15T10:12:52.382719622Z"
        },
        "fields": {
          "date": [
            "2022-06-15T10:12:52.382719622Z"
          ]
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 1,
        "_source": {
          "date": "2022-06-15T10:12:52.382719624Z"
        },
        "fields": {
          "date": [
            "2022-06-15T10:12:52.382719624Z"
          ]
        }
      }
    ]
  }
}
```

你可以排序`date_nanos` 字段如下：

```json
GET testindex/_search
{
  "sort": { 
    "date": "asc"
  } 
}
```
{% include copy-curl.html %}

响应包含分类的文档：

```json
{
  "took": 5,
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
    "max_score": null,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": null,
        "_source": {
          "date": "2022-06-15T10:12:52.382719622Z"
        },
        "sort": [
          1655287972382719700
        ]
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": null,
        "_source": {
          "date": "2022-06-15T10:12:52.382719624Z"
        },
        "sort": [
          1655287972382719700
        ]
      }
    ]
  }
}
```

您还可以使用无痛脚本来访问该领域的纳秒部分：

```json
GET testindex/_search
{
  "script_fields" : {
    "my_field" : {
      "script" : {
        "lang" : "painless",
        "source" : "doc['date'].value.nano" 
      }
    }
  }
}
```
{% include copy-curl.html %}

该响应仅包含字段的纳秒部分：

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
    "max_score": 1,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 1,
        "fields": {
          "my_field": [
            382719622
          ]
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 1,
        "fields": {
          "my_field": [
            382719624
          ]
        }
      }
    ]
  }
}
```
