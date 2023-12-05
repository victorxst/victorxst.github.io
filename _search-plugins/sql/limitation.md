---
layout: default
title: 限制
parent: SQL和PPL
nav_order: 99
redirect_from:
  - /search-plugins/sql/limitation/
---

# 限制

SQL插件具有以下限制：

## 不支持表达的汇总

您只能将聚合应用于字段。聚合不能接受表达式作为参数。例如，`avg(log(age))` 不支持。

## 从子句中的子查询

子查询`FROM` 该格式的条款：`SELECT outer FROM (SELECT inner)` 仅当查询合并到一个查询中时才会支持。例如，支持以下查询：

```sql
SELECT t.f, t.d
FROM (
    SELECT FlightNum as f, DestCountry as d
    FROM opensearch_dashboards_sample_data_flights
    WHERE OriginCountry = 'US') t
```

但是，如果外部查询具有`GROUP BY` 或者`ORDER BY`，那么不支持。

## 加入不支持在加入结果上的聚合

这`join` 查询不支持加入结果上的聚合。
例如，例如`SELECT depo.name, avg(empo.age) FROM empo JOIN depo WHERE empo.id == depo.id GROUP BY depo.name` 不支持。

## 分页只支持基本查询

分页查询使您能够恢复分页的响应。

目前，分页仅支持基本查询。例如，以下查询将使用光标ID返回数据。

```json
POST _plugins/_sql/
{
  "fetch_size" : 5,
  "query" : "SELECT OriginCountry, DestCountry FROM opensearch_dashboards_sample_data_flights ORDER BY OriginCountry ASC"
}
```

具有光标ID的JDBC格式的响应。

```json
{
  "schema": [
    {
      "name": "OriginCountry",
      "type": "keyword"
    },
    {
      "name": "DestCountry",
      "type": "keyword"
    }
  ],
  "cursor": "d:eyJhIjp7fSwicyI6IkRYRjFaWEo1UVc1a1JtVjBZMmdCQUFBQUFBQUFCSllXVTJKVU4yeExiWEJSUkhsNFVrdDVXVEZSYkVKSmR3PT0iLCJjIjpbeyJuYW1lIjoiT3JpZ2luQ291bnRyeSIsInR5cGUiOiJrZXl3b3JkIn0seyJuYW1lIjoiRGVzdENvdW50cnkiLCJ0eXBlIjoia2V5d29yZCJ9XSwiZiI6MSwiaSI6ImtpYmFuYV9zYW1wbGVfZGF0YV9mbGlnaHRzIiwibCI6MTMwNTh9",
  "total": 13059,
  "datarows": [[
    "AE",
    "CN"
  ]],
  "size": 1,
  "status": 200
}
```

查询`aggregation` 和`join` 目前不支持分页。

## 查询处理引擎

SQL插件具有两个查询处理引擎，`V1` 和`V2`。大多数功能都得到了两个发动机的支持，但是只有新引擎正在积极开发。首先在`V2` 发动机倒回到`V1` 引擎在失败的情况下。如果支持查询`V2` 但不包括`V1`，查询将通过错误响应失败。

### V1发动机限制

*没有的文字表达`FROM` 条款不支持。例如，`SELECT 1` 不支持。
* 这`WHERE` 条款不支持表达式。例如，`SELECT FlightNum FROM opensearch_dashboards_sample_data_flights where (AvgTicketPrice + 100) <= 1000` 不支持。
* 最多[相关搜索功能]({{site.url}}{{site.baseurl}}/search-plugins/sql/full-text/) 在`V2` 仅发动机。

此类查询已由`V2` 发动机，除非他们有`V1`-特定功能。您可能永远不会达到这些限制。

### V2发动机限制

* 这[光标功能](#pagination-only-supports-basic-queries) 由`V1` 仅发动机。
  *以支持`cursor`/`pagination` 在里面`V2` 引擎，轨道[Github问题#656](https://github.com/opensearch-project/sql/issues/656)。
*`json` 在`V1` 仅发动机。
* 这`V2` 引擎没有跟踪查询执行时间，因此未报告查询缓慢。
* 这`V2` 查询引擎不仅在OpenSearch Engine中运行查询，还支持发布-处理复杂查询。因此，`explain` 输出不再是OpenSearch域-特定语言（DSL），但还包括来自的查询计划信息`V2` 查询引擎。
建议更改
* 这`V2` 查询引擎不支持聚合查询，例如`histogram`，`date_histogram`，`percentiles`，`topHits`，`stats`，`extended_stats`，`terms`， 或者`range`。
*加入和sub-不支持查询。保持最新的加入和sub的开发-查询，跟踪[Github问题#1441](https://github.com/opensearch-project/sql/issues/1441) 和[Github问题#892](https://github.com/opensearch-project/sql/issues/892)。
* partiql语法`nested` 不支持查询。此外，对象和原始类型的数组返回数组的第一个索引，而`V1` 他们将整个数组作为JSON对象返回。

