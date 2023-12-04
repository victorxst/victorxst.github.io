---
layout: default
title: 执行简单的存储脚本
parent: 脚本API
nav_order: 2
---

# 执行简单的存储脚本
**引入1.0**
{: .label .label-purple }

运行一个用无痛语言写的存储脚本。

OpenSearch提供了几种运行脚本的方法；以下各节显示了如何通过在请求主体中传递脚本信息来运行脚本`GET <index>/_search` 要求。

## 请求字段

| 场地| 数据类型| 描述| 
:--- | :--- | :---
| 询问| 目的| 指定要处理的文档的过滤器。|
| script_fields| 目的| 输出中包括的字段。| 
| 脚本| 目的| 产生字段值的脚本的ID。|

#### 示例请求

以下请求运行了在[创建或更新存储的脚本]({{site.url}}{{site.baseurl}}/api-reference/script-apis/create-stored-script/)。脚本总和每本书的评分，并在`total_ratings` 输出中的字段。

*脚本的目标是`books` 指数。

* 这`"match_all": {}` 属性值是一个空的对象，指示在索引中处理每个文档。

* 这`total_ratings` 现场值是`my-first-script` 执行。看[创建或更新存储的脚本]({{site.url}}{{site.baseurl}}/api-reference/script-apis/create-stored-script/)。

````json
GET books/_search
{
   "query": {
    "match_all": {}
  },
  "script_fields": {
    "total_ratings": {
      "script": {
        "id": "my-first-script" 
      }
    }
  }
}
````
{% include copy-curl.html %}

#### 示例响应

这`GET books/_search` 请求返回以下字段：

````json
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "books",
        "_id" : "1",
        "_score" : 1.0,
        "fields" : {
          "total_ratings" : [
            12
          ]
        }
      },
      {
        "_index" : "books",
        "_id" : "2",
        "_score" : 1.0,
        "fields" : {
          "total_ratings" : [
            15
          ]
        }
      },
      {
        "_index" : "books",
        "_id" : "3",
        "_score" : 1.0,
        "fields" : {
          "total_ratings" : [
            8
          ]
        }
      }
    ]
  }
}
````

## 响应字段

| 场地| 数据类型| 描述| 
:--- | :--- | :---
| 拿| 整数| 操作花了多长时间。|
| 时间到| 布尔| 该操作是否定时。|
| _沙尔德| 目的| 处理的碎片总数以及成功，跳过且未处理的总数。|
| 命中| 目的| 包含高-有关所处理文档的级别信息和一系列`hits` 对象。看[命中对象](#hits-object)。| 

#### 命中对象

| 场地| 数据类型| 描述| 
:--- | :--- | :---
| 全部的| 目的| 处理的文件总数及其与`match` 请求字段。|
| max_score| 双倍的| 最高相关性得分从所有命中返回。|
| 命中| 大批| 有关处理的每个文档的信息。看[文档对象](#Document-object)。|

#### 文档对象

| 场地| 数据类型| 描述| 
:--- | :--- | :---
| _指数| 细绳| 包含文档的索引。|
| _ID| 细绳| 文档ID。|
| _分数| 漂浮| 文档的相关性得分。|
| 字段| 目的| 字段及其价值从脚本返回。|

## 使用参数运行无痛的存储脚本

每次运行查询时，要将不同的参数传递到脚本，请定义`params` 在`script_fields`。

#### 例子

以下请求运行了在[创建或更新存储的脚本]({{site.url}}{{site.baseurl}}/api-reference/script-apis/create-stored-script/)。脚本总和每本书的评分，将总价值乘以`multiplier` 参数，并在输出中显示结果。

*脚本的目标是`books` 指数。

* 这`"match_all": {}` 属性值是一个空对象，表明它处理索引中的每个文档。

* 这`total_ratings` 现场值是`multiplier-script` 执行。看[使用参数创建或更新存储的脚本]({{site.url}}{{site.baseurl}}/api-reference/script-apis/create-stored-script/)。

*`"multiplier": 2` 在里面`params` 字段是传递给存储脚本的变量`multiplier-script`：

```json
GET books/_search
{
   "query": {
    "match_all": {}
  },
  "script_fields": {
    "total_ratings": {
      "script": {
        "id": "multiplier-script", 
        "params": {
          "multiplier": 2
        }        
      }
    }
  }
}
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "took" : 12,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "books",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 1.0,
        "fields" : {
          "total_ratings" : [
            16
          ]
        }
      },
      {
        "_index" : "books",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.0,
        "fields" : {
          "total_ratings" : [
            30
          ]
        }
      },
      {
        "_index" : "books",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "fields" : {
          "total_ratings" : [
            24
          ]
        }
      }
    ]
  }
}
```

**使用无痛脚本对结果进行排序
您可以使用无痛的存储脚本对结果进行排序。**

#### 样本请求

```json
GET books/_search
{
   "query": {
    "match_all": {}
  },
  "script_fields": {
    "total_ratings": {
      "script": {
        "id": "multiplier-script",
        "params": {
          "multiplier": 2
        }
      }
    }
  },
  "sort": {
    "_script": {
       "type": "number",
       "script": {
         "id": "multiplier-script",
         "params": {
           "multiplier": 2
          }
       },
       "order": "desc"
    }
  }
}
```

#### 样本响应

```json
{
  "took" : 90,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "books",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : null,
        "fields" : {
          "total_ratings" : [
            30
          ]
        },
        "sort" : [
          30.0
        ]
      },
      {
        "_index" : "books",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : null,
        "fields" : {
          "total_ratings" : [
            24
          ]
        },
        "sort" : [
          24.0
        ]
      },
      {
        "_index" : "books",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : null,
        "fields" : {
          "total_ratings" : [
            16
          ]
        },
        "sort" : [
          16.0
        ]
      }
    ]
  }
}
```

