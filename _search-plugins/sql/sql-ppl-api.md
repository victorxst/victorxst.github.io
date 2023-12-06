---
layout: default
title: SQL和PPL API
parent: SQL和PPL
nav_order: 1
---

# SQL和PPL API

使用SQL和PPL API将查询发送到SQL插件。使用`_sql` 在SQL中发送查询的端点，以及`_ppl` 端点要在ppl中发送查询。对于这两者，您也可以使用`_explain` 将查询转化为[OpenSearch域-具体语言]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/) （DSL）或故障排除错误。

## 查询API

将SQL/PPL查询发送到SQL插件。您可以将响应格式作为查询参数传递。

### 查询参数

范围| 数据类型| 描述
:--- | :--- | :---
[格式]({{site.url}}{{site.baseurl}}/search-plugins/sql/response-formats/) | 细绳| 响应的格式。这`_sql` 端点支持`jdbc`，`csv`，`raw`， 和`json` 格式。这`_ppl` 端点支持`jdbc`，`csv`， 和`raw` 格式。默认为`jdbc`。
消毒| 布尔| 指定是否逃脱结果中的特殊角色。看[响应格式]({{site.url}}{{site.baseurl}}/search-plugins/sql/response-formats/) 了解更多信息。默认为`true`。

### 请求字段

场地| 数据类型| 描述
:--- | :--- | :---
询问| 细绳| 要执行的查询。必需的。
[筛选](#filtering-results) | JSON对象| 结果的过滤器。选修的。
[fetch_size](#paginating-results) | 整数| 一个响应中返回的结果数。用于登机结果。默认值为1,000。选修的。仅支持`jdbc` 响应格式。

#### 示例请求

```json
POST /_plugins/_sql 
{
  "query" : "SELECT * FROM accounts"
}
```

#### 示例响应

响应包含架构和结果：

```json
{
  "schema": [
    {
      "name": "account_number",
      "type": "long"
    },
    {
      "name": "firstname",
      "type": "text"
    },
    {
      "name": "address",
      "type": "text"
    },
    {
      "name": "balance",
      "type": "long"
    },
    {
      "name": "gender",
      "type": "text"
    },
    {
      "name": "city",
      "type": "text"
    },
    {
      "name": "employer",
      "type": "text"
    },
    {
      "name": "state",
      "type": "text"
    },
    {
      "name": "age",
      "type": "long"
    },
    {
      "name": "email",
      "type": "text"
    },
    {
      "name": "lastname",
      "type": "text"
    }
  ],
  "datarows": [
    [
      1,
      "Amber",
      "880 Holmes Lane",
      39225,
      "M",
      "Brogan",
      "Pyrami",
      "IL",
      32,
      "amberduke@pyrami.com",
      "Duke"
    ],
    [
      6,
      "Hattie",
      "671 Bristol Street",
      5686,
      "M",
      "Dante",
      "Netagy",
      "TN",
      36,
      "hattiebond@netagy.com",
      "Bond"
    ],
    [
      13,
      "Nanette",
      "789 Madison Street",
      32838,
      "F",
      "Nogal",
      "Quility",
      "VA",
      28,
      "nanettebates@quility.com",
      "Bates"
    ],
    [
      18,
      "Dale",
      "467 Hutchinson Court",
      4180,
      "M",
      "Orick",
      null,
      "MD",
      33,
      "daleadams@boink.com",
      "Adams"
    ]
  ],
  "total": 4,
  "size": 4,
  "status": 200
}
```

### 响应字段

场地| 数据类型| 描述
:--- | :--- | :---
模式| 大批| 指定所有字段的字段名称和类型。
data_rows| 2D数组| 一系列结果。每个结果代表一个匹配行（文档）。
全部的| 整数| 索引中的行总数（文档）。
尺寸| 整数| 一个响应中返回的结果数。
地位| 细绳| 运行查询后，HTTP响应状态opensearch返回。

## 解释API

SQL插件具有`explain` 功能显示了如何针对OpenSearch执行查询，这对于调试和开发非常有用。向邮政请求`_plugins/_sql/_explain` 或者`_plugins/_ppl/_explain` 端点返回[OpenSearch域-具体语言]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/) （DSL）以JSON格式解释查询。
您可以使用命令行中执行说明API操作`curl` 或在仪表板控制台中，如下示例。

#### 示例说明SQL查询请求

```json
POST _plugins/_sql/_explain
{
  "query": "SELECT firstname, lastname FROM accounts WHERE age > 20"
}
```

#### 示例SQL查询解释响应

```json
{
  "root": {
    "name": "ProjectOperator",
    "description": {
      "fields": "[firstname, lastname]"
    },
    "children": [
      {
        "name": "OpenSearchIndexScan",
        "description": {
          "request": """OpenSearchQueryRequest(indexName=accounts, sourceBuilder={"from":0,"size":200,"timeout":"1m","query":{"range":{"age":{"from":20,"to":null,"include_lower":false,"include_upper":true,"boost":1.0}}},"_source":{"includes":["firstname","lastname"],"excludes":[]},"sort":[{"_doc":{"order":"asc"}}]}, searchDone=false)"""
        },
        "children": []
      }
    ]
  }
}
```

#### 示例解释请求PPL查询

```json
POST _plugins/_ppl/_explain
{
  "query" : "source=accounts | fields firstname, lastname"
}
```

#### 示例PPL查询解释响应

```json
{
  "root": {
    "name": "ProjectOperator",
    "description": {
      "fields": "[firstname, lastname]"
    },
    "children": [
      {
        "name": "OpenSearchIndexScan",
        "description": {
          "request": """OpenSearchQueryRequest(indexName=accounts, sourceBuilder={"from":0,"size":200,"timeout":"1m","_source":{"includes":["firstname","lastname"],"excludes":[]}}, searchDone=false)"""
        },
        "children": []
      }
    ]
  }
}
```

对于需要发布的查询-处理，`explain` 响应还包括OpenSearch DSL之外的查询计划。对于那些不需要后处理的查询，您可以看到完整的DSL。

## 登上结果

要恢复分页的回应，请使用`fetch_size` 范围。的价值`fetch_size` 应大于0。默认值为1,000。值为0的值将返回到非-分页的响应。

这`fetch_size` 参数仅支持`jdbc` 响应格式。
{: .note}

### 例子

以下请求包含SQL查询，并指定一次返回五个结果：

```json
POST _plugins/_sql/
{
  "fetch_size" : 5,
  "query" : "SELECT firstname, lastname FROM accounts WHERE age > 20 ORDER BY state ASC"
}
```

响应包含所有查询的所有字段`fetch_size` 将包含，一个`cursor` 用于检索后续结果的字段：

```json
{
  "schema": [
    {
      "name": "firstname",
      "type": "text"
    },
    {
      "name": "lastname",
      "type": "text"
    }
  ],
  "cursor": "d:eyJhIjp7fSwicyI6IkRYRjFaWEo1UVc1a1JtVjBZMmdCQUFBQUFBQUFBQU1XZWpkdFRFRkZUMlpTZEZkeFdsWnJkRlZoYnpaeVVRPT0iLCJjIjpbeyJuYW1lIjoiZmlyc3RuYW1lIiwidHlwZSI6InRleHQifSx7Im5hbWUiOiJsYXN0bmFtZSIsInR5cGUiOiJ0ZXh0In1dLCJmIjo1LCJpIjoiYWNjb3VudHMiLCJsIjo5NTF9",
  "total": 956,
  "datarows": [
    [
      "Cherry",
      "Carey"
    ],
    [
      "Lindsey",
      "Hawkins"
    ],
    [
      "Sargent",
      "Powers"
    ],
    [
      "Campos",
      "Olsen"
    ],
    [
      "Savannah",
      "Kirby"
    ]
  ],
  "size": 5,
  "status": 200
}
```

要获取后续页面，请使用`cursor` 从以前的回应：

```json
POST /_plugins/_sql 
{
   "cursor": "d:eyJhIjp7fSwicyI6IkRYRjFaWEo1UVc1a1JtVjBZMmdCQUFBQUFBQUFBQU1XZWpkdFRFRkZUMlpTZEZkeFdsWnJkRlZoYnpaeVVRPT0iLCJjIjpbeyJuYW1lIjoiZmlyc3RuYW1lIiwidHlwZSI6InRleHQifSx7Im5hbWUiOiJsYXN0bmFtZSIsInR5cGUiOiJ0ZXh0In1dLCJmIjo1LCJpIjoiYWNjb3VudHMiLCJsIjo5NTF9"
}
```

下一个响应仅包含`datarows` 结果和新的`cursor`。

```json
{
  "cursor": "d:eyJhIjp7fSwicyI6IkRYRjFaWEo1UVc1a1JtVjBZMmdCQUFBQUFBQUFBQU1XZWpkdFRFRkZUMlpTZEZkeFdsWnJkRlZoYnpaeVVRPT0iLCJjIjpbeyJuYW1lIjoiZmlyc3RuYW1lIiwidHlwZSI6InRleHQifSx7Im5hbWUiOiJsYXN0bmFtZSIsInR5cGUiOiJ0ZXh0In1dLCJmIjo1LCJpIjoiYWNjb3VudHMabcde12345",
  "datarows": [
    [
      "Abbey",
      "Karen"
    ],
    [
      "Chen",
      "Ken"
    ],
    [
      "Ani",
      "Jade"
    ],
    [
      "Peng",
      "Hu"
    ],
    [
      "John",
      "Doe"
    ]
  ]
}
```

这`datarows` 可以比`fetch_size` 记录数量在嵌套字段时被扁平。
{: .note}

结果的最后一页只有`datarows` 和不`cursor`。这`cursor` 上面的上下文在最后一页上自动清除。

要明确清除光标上下文，请使用`_plugins/_sql/close` 端点操作：

```json
POST /_plugins/_sql/close 
{
   "cursor": "d:eyJhIjp7fSwicyI6IkRYRjFaWEo1UVc1a1JtVjBZMmdCQUFBQUFBQUFBQU1XZWpkdFRFRkZUMlpTZEZkeFdsWnJkRlZoYnpaeVVRPT0iLCJjIjpbeyJuYW1lIjoiZmlyc3RuYW1lIiwidHlwZSI6InRleHQifSx7Im5hbWUiOiJsYXN0bmFtZSIsInR5cGUiOiJ0ZXh0In1dLCJmIjo1LCJpIjoiYWNjb3VudHMiLCJsIjo5NTF9"
}'
```

响应是OpenSearch的确认：

```json
{"succeeded":true}
```

## 过滤结果

您可以使用`filter` 参数直接向OpenSearch DSL添加更多条件。

以下SQL查询返回所有客户的名称和帐户余额。然后对结果进行过滤，以仅包含那些余额不到$ 10,000的客户。

```json
POST /_plugins/_sql/ 
{
  "query" : "SELECT firstname, lastname, balance FROM accounts",
  "filter" : {
    "range" : {
      "balance" : {
        "lt" : 10000
      }
    }
  }
}
```

响应包含匹配结果：

```json
{
  "schema": [
    {
      "name": "firstname",
      "type": "text"
    },
    {
      "name": "lastname",
      "type": "text"
    },
    {
      "name": "balance",
      "type": "long"
    }
  ],
  "total": 2,
  "datarows": [
    [
      "Hattie",
      "Bond",
      5686
    ],
    [
      "Dale",
      "Adams",
      4180
    ]
  ],
  "size": 2,
  "status": 200
}
```

您可以使用Divell API查看该查询是如何针对OpenSearch执行的：

```json
POST /_plugins/_sql/_explain 
{
  "query" : "SELECT firstname, lastname, balance FROM accounts",
  "filter" : {
    "range" : {
      "balance" : {
        "lt" : 10000
      }
    }
  }
}'
```

该响应包含OpenSearch DSL中的布尔查询，该查询与上述查询相对应：

```json
{
  "from": 0,
  "size": 200,
  "query": {
    "bool": {
      "filter": [{
        "bool": {
          "filter": [{
            "range": {
              "balance": {
                "from": null,
                "to": 10000,
                "include_lower": true,
                "include_upper": false,
                "boost": 1.0
              }
            }
          }],
          "adjust_pure_negative": true,
          "boost": 1.0
        }
      }],
      "adjust_pure_negative": true,
      "boost": 1.0
    }
  },
  "_source": {
    "includes": [
      "firstname",
      "lastname",
      "balance"
    ],
    "excludes": []
  }
}
```

## 使用参数

您可以使用`parameters` 字段将参数值传递给准备好的SQL查询。

以下解释操作使用SQL查询`age` 范围：

```json
POST /_plugins/_sql/_explain 
{
  "query": "SELECT * FROM accounts WHERE age = ?",
  "parameters": [{
    "type": "integer",
    "value": 30
  }]
}
```

该响应包含OpenSearch DSL中的布尔查询，该查询与上面的SQL查询相对应：

```json
{
  "from": 0,
  "size": 200,
  "query": {
    "bool": {
      "filter": [{
        "bool": {
          "must": [{
            "term": {
              "age": {
                "value": 30,
                "boost": 1.0
              }
            }
          }],
          "adjust_pure_negative": true,
          "boost": 1.0
        }
      }],
      "adjust_pure_negative": true,
      "boost": 1.0
    }
  }
}

```

