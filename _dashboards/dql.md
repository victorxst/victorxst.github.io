---
layout: default
title: DQL
nav_order: 130
redirect_from:
  - /dashboards/dql/
  - /dashboards/discover/dql/
---

# DQL

仪表板查询语言（DQL）是一个简单的文本-基于opensearch仪表板中过滤数据的基于查询语言。如同[查询DSL]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/index/)，DQL使用HTTP请求主体。例如，要显示美国主机的网站访问者数据，您将输入`geo.dest:US` 在搜索字段中，如下图所示。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/dql-interface.png" alt="Search term using DQL toolbar in Dashboard" width="500">

在可以在仪表板中搜索数据之前，必须将其索引。在OpenSearch中，数据的基本单元是JSON文档。在索引中，OpenSearch使用唯一的ID标识每个文档。要了解有关OpenSearch中索引的更多信息，请参见[索引数据]({{site.url}}{{site.baseurl}}/opensearch/index-data)。
{：.. note purple}

## 搜索术语查询

最基本的查询指定搜索词，例如：

```
host:www.example.com
```

要访问对象的嵌套字段，请列出按周期分隔的字段的完整路径。例如，使用以下路径检索`lat` 字段`coordinates` 目的：

```
coordinates.lat:43.7102
```

DQL支持领先和拖延通配符，因此您可以搜索与您的模式相匹配的任何术语，例如：

```
host.keyword:*.example.com/*
```

要检查一个字段是否存在或有任何数据，请使用通配符查看仪表板是否返回任何结果，例如：

```
host.keyword:*
```

## 用布尔查询搜索

要混合和匹配或组合多个查询以获得更精致的结果，您可以使用布尔操作员`and`，`or`， 和`not`。DQL不敏感，所以`AND` 和`and` 例如，相同：

```
host.keyword:www.example.com and response.keyword:200
```

您还可以在一个查询中使用多个布尔运算符，例如：

```
geo.dest:US or response.keyword:200 and host.keyword:www.example.com
```

请记住，布尔操作员遵循的逻辑优先顺序`not`，`and`， 和`or`，因此，如果您在上一个示例中的表达式类似于表达式，`response.keyword:200 and host.keyword:www.example.com` 首先评估。

为避免混乱，请使用括号决定要评估操作数的顺序。如果您想评估`geo.dest:US or response.keyword:200` 首先，您可以使用以下表达式：

```
(geo.dest:US or response.keyword:200) and host.keyword:www.example.com
```

## 查询日期和范围

DQL支持数字不等式，例如`bytes >= 15 and memory < 15`。

您可以使用相同的方法在查询中指定的日期之前或之后找到日期。`>` 指示搜索指定日期之后的日期，以及`<` 例如，在指定日期之前返回日期`@timestamp > "2020-12-14T09:35:33`。

## 查询嵌套字段

搜索文档[嵌套字段]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/nested/) 要求您指定要检索的字段的完整路径。在以下示例文档中`superheroes` 字段具有嵌套对象：

```json
{
 "superheroes":[
    {
      "hero-name": "Superman",
      "real-identity": "Clark Kent",
      "age": 28
    },
    {
      "hero-name": "Batman",
      "real-identity": "Bruce Wayne",
      "age": 26
    },
    {
      "hero-name": "Flash",
      "real-identity": "Barry Allen",
      "age": 28
    },
    {
      "hero-name": "Robin",
      "real-identity": "Dick Grayson",
      "age": 15
    }
  ]
}
```
{% include copy.html %}

要检索使用DQL匹配特定字段的文档，请指定该字段，例如：

```
superheroes: {hero-name: Superman}
```
{% include copy.html %}

要检索匹配多个字段的文档，请指定所有字段，例如：

```
superheroes: {hero-name: Superman} and superheroes: {hero-name: Batman}
```
{% include copy.html %}

您可以组合多个布尔值和范围查询以创建一个更精致的查询，例如：

```
superheroes: {hero-name: Superman and age < 50}
```
{% include copy.html %}

## 查询双嵌套对象

如果文档具有双嵌套对象（对象嵌套在其他对象中），请通过指定通往字段的完整路径来检索字段值。在以下示例文档中`superheroes` 对象嵌套在`justice-league` 目的：

```json
{
"justice-league": [
{
"superheroes":[
{
"hero-name": "Superman",
"real-identity": "Clark Kent",
"age": 28
},
{
"hero-name": "Batman",
"real-identity": "Bruce Wayne",
"age": 26
},
{
"hero-name": "Flash",
"real-identity": "Barry Allen",
"age": 28
},
{
"hero-name": "Robin",
"real-identity": "Dick Grayson",
"age": 15
}
]
}
]
}
```
{% include copy.html %}

下图显示了使用示例符号表示查询结果`justice-league.superheroes: {hero-name:Superman}`。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/dql-query-result.png" alt="DQL query result" width="1000">

