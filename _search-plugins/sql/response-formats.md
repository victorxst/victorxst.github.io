---
layout: default
title: 响应格式
parent: SQL和PPL
nav_order: 2
---

# 响应格式

SQL插件提供`jdbc`，，，，`csv`，，，，`raw`， 和`json` 对不同目的有用的响应格式。这`jdbc` 格式被广泛使用，因为它提供了架构信息并添加了更多功能，例如分页。除JDBC驱动程序外，各种客户还可以从详细且良好的-格式的响应。

## JDBC格式

默认情况下，SQL插件以标准JDBC格式返回响应。为JDBC驱动程序和需要架构的JDBC驱动程序提供了这种格式，并且结果设置为良好的格式。

#### 示例请求

以下查询未指定响应格式，因此将格式设置为`jdbc`：

```json
POST _plugins/_sql
{
  "query" : "SELECT firstname, lastname, age FROM accounts ORDER BY age LIMIT 2"
}
```

#### 示例响应

在回应中`schema` 包含字段名称和类型，以及`datarows` 字段包含结果集：

```json
{
  "schema": [{
      "name": "firstname",
      "type": "text"
    },
    {
      "name": "lastname",
      "type": "text"
    },
    {
      "name": "age",
      "type": "long"
    }
  ],
  "total": 4,
  "datarows": [
    [
      "Nanette",
      "Bates",
      28
    ],
    [
      "Amber",
      "Duke",
      32
    ]
  ],
  "size": 2,
  "status": 200
}
```

如果发生任何类型的错误，OpenSearch返回错误消息。

以下查询搜索非-存在的字段`unknown`：

```json
POST /_plugins/_sql
{
  "query" : "SELECT unknown FROM accounts"
}
```

响应包含错误消息和错误的原因：

```json
{
  "error": {
    "reason": "Invalid SQL query",
    "details": "Field [unknown] cannot be found or used here.",
    "type": "SemanticAnalysisException"
  },
  "status": 400
}
```

## OpenSearch DSL JSON格式

如果将格式设置为`json`，原始OpenSearch响应以JSON格式返回。因为这是Opensearch的本地响应，因此需要额外的努力来解析和解释它。

#### 示例请求

以下查询将响应格式设置为`json`：

```json
POST _plugins/_sql?format=json
{
  "query" : "SELECT firstname, lastname, age FROM accounts ORDER BY age LIMIT 2"
}
```

#### 示例响应

响应是OpenSearch的原始响应：

```json
{
  "_shards": {
    "total": 5,
    "failed": 0,
    "successful": 5,
    "skipped": 0
  },
  "hits": {
    "hits": [{
        "_index": "accounts",
        "_type": "account",
        "_source": {
          "firstname": "Nanette",
          "age": 28,
          "lastname": "Bates"
        },
        "_id": "13",
        "sort": [
          28
        ],
        "_score": null
      },
      {
        "_index": "accounts",
        "_type": "account",
        "_source": {
          "firstname": "Amber",
          "age": 32,
          "lastname": "Duke"
        },
        "_id": "1",
        "sort": [
          32
        ],
        "_score": null
      }
    ],
    "total": {
      "value": 4,
      "relation": "eq"
    },
    "max_score": null
  },
  "took": 100,
  "timed_out": false
}
```

## CSV格式

您还可以指定以CSV格式返回结果。

#### 示例请求

```json
POST /_plugins/_sql?format=csv
{
  "query" : "SELECT firstname, lastname, age FROM accounts ORDER BY age"
}
```

#### 示例响应

```text
firstname,lastname,age
Nanette,Bates,28
Amber,Duke,32
Dale,Adams,33
Hattie,Bond,36
```
### 用CSV格式进行消毒结果

默认情况下，OpenSearch根据以下规则对标头单元（字段名称）和数据单元格（字段内容）进行了清理：

- 如果一个单元开始`+`，，，，`-`，，，，`=` ， 或者`@`，消毒剂插入一个报价（`'`）在细胞开始时。
- 如果一个单元包含一个或多个逗号（`,`），消毒剂用双引号围绕电池（`"`）。

### 例子

以下查询索引了一个文档，其中包含特殊字符开头或包含逗号的单元格：

```json
PUT /userdata/_doc/1?refresh=true
{
  "+firstname": "-Hattie",
  "=lastname": "@Bond",
  "address": "671 Bristol Street, Dente, TN"
}
```

您可以使用以下查询以CSV格式请求结果：

```json
POST /_plugins/_sql?format=csv
{
  "query" : "SELECT * FROM userdata"
}
```

在响应中，以特殊字符开头的单元格有前缀`'`。带有逗号的单元被引号包围：

```text
'+firstname,'=lastname,address
'Hattie,'@Bond,"671 Bristol Street, Dente, TN"
```

要跳过消毒，请设置`sanitize` 查询参数到false：

```json
POST /_plugins/_sql?format=csvandsanitize=false
{
  "query" : "SELECT * FROM userdata"
}
```

响应包含原始CSV格式的结果：

```text
=lastname,address,+firstname
@Bond,"671 Bristol Street, Dente, TN",-Hattie
```

## 原始格式

您可以使用原始格式将结果管输送到其他命令行工具以供发布-加工。

#### 示例请求

```json
POST /_plugins/_sql?format=raw
{
  "query" : "SELECT firstname, lastname, age FROM accounts ORDER BY age"
}
```

#### 示例响应

```text
Nanette|Bates|28
Amber|Duke|32
Dale|Adams|33
Hattie|Bond|36
```

默认情况下，OpenSearch对结果进行了清理`raw` 根据以下规则格式：

- 如果数据单元包含一个或多个管道字符（`|`），消毒剂用双引号围绕着细胞。

### 例子

以下查询索引带有管子字符的文档（`|`）在其领域：

```json
PUT /userdata/_doc/1?refresh=true
{
  "+firstname": "|Hattie",
  "=lastname": "Bond|",
  "|address": "671 Bristol Street| Dente| TN"
}
```

您可以使用下面的查询请求结果`raw` 格式：

```json
POST /_plugins/_sql?format=raw
{
  "query" : "SELECT * FROM userdata"
}
```

查询通过`|` 引号包围的角色：

```text
"|address"|=lastname|+firstname
"671 Bristol Street| Dente| TN"|"Bond|"|"|Hattie"
```
