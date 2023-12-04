---
layout: default
title: Range字段类型
nav_order: 35
has_children: false
grand_parent: 支持的字段类型
redirect_from:
  - /opensearch/supported-field-types/range/
  - /field-types/range/
---

# Range字段类型

下表列出了OpenSearch支持的所有范围字段类型。

字段数据类型| 描述
：--- | ：---
`integer_range` | 范围[整数]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/numeric/) 值。
`long_range` | 范围[长的]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/numeric/) 值。
`double_range` | 范围[双倍的]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/numeric/) 值。
`float_range` | 范围[漂浮]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/numeric/) 值。
`ip_range` | 范围[IP地址]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/ip/) 在IPv4或IPv6格式中。启动和结束IP地址可能以不同的格式。
`date_range` | 范围[日期]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/date/) 值。开始和结束日期可能不同[格式]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/date/#formats)。在内部，所有日期都存储为未签名64-自时代以来代表毫秒的位整数。

## 例子

创建具有双重范围和日期范围的映射：

```json
PUT testindex 
{
  "mappings" : {
    "properties" :  {
      "gpa" : {
        "type" : "double_range"
      },
      "graduation_date" : {
        "type" : "date_range",
        "format" : "strict_year_month||strict_year_month_day"
      }
    }
  }
}
```
{% include copy-curl.html %}

索引具有双重范围和日期范围的文档：

```json
PUT testindex/_doc/1
{
  "gpa" : {
    "gte" : 1.0,
    "lte" : 4.0
  },
  "graduation_date" : {
    "gte" : "2019-05-01",
    "lte" : "2019-05-15"
  }
}
```
{% include copy-curl.html %}

## IP地址范围

您可以以两种格式指定IP地址范围：作为一个范围和[CIDR表示法](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing#CIDR_notation)。

创建具有IP地址范围的映射：

```json
PUT testindex 
{
  "mappings" : {
    "properties" :  {
      "ip_address_range" : {
        "type" : "ip_range" 
      },
      "ip_address_cidr" : {
        "type" : "ip_range" 
      }
    }
  }
}
```
{% include copy-curl.html %}

索引具有两种格式IP地址范围的文档：

```json
PUT testindex/_doc/2
{
  "ip_address_range" : {
    "gte" : "10.24.34.0",
    "lte" : "10.24.35.255"
  },
  "ip_address_cidr" : "10.24.34.0/24"
}
```
{% include copy-curl.html %}

## 查询范围字段

您可以使用[术语查询](#term-query) 或a[范围查询](#range-query) 搜索范围字段内的值。

### 术语查询

术语查询具有一个值，并匹配该值在范围内的所有范围字段。

以下查询将返回文档1，因为3.5在[1.0，4.0]范围内：

```json
GET testindex/_search
{
  "query" : {
    "term" : {
      "gpa" : {
        "value" : 3.5
      }
    }
  }
}
```
{% include copy-curl.html %}

### 范围查询

范围字段上的范围查询返回该范围内的文档。

查询2019年所有毕业日期，提供日期范围"MM/dd/yyyy" 格式：

```json
GET testindex1/_search
{
  "query": {
    "range": {
      "graduation_date": {
        "gte": "01/01/2019",
        "lte": "12/31/2019",
        "format": "MM/dd/yyyy",
        "relation" : "within"       
      }
    }
  }
}
```
{% include copy-curl.html %}

前面的查询将返回文件1`within` 和`intersects` 关系，但不会归还`contains` 关系。有关关系类型的更多信息，请参见[范围查询参数]({{site.url}}{{site.baseurl}}/query-dsl/term/range#parameters)。

## 参数

下表列出了按范围字段类型接受的参数。所有参数都是可选的。

范围| 描述
：--- | ：--- 
`boost` | 浮动-指定该字段对相关性分数的重量的点值。值高于1.0的值增加了该领域的相关性。0.0至1.0之间的值降低了该场的相关性。默认值为1.0。
`coerce` | 一个布尔值，该值向整数值截断了小数，并将字符串转换为数字值。默认为`true`。
`index` | 布尔值指定是否应搜索该字段。默认为`true`。
`store` | 布尔值指定是否应存储字段值，并且可以与_source字段分开检索。默认为`false`。

