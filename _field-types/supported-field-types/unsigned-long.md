---
layout: default
title: 未签名长
parent: Numeric字段类型
grand_grand_parent: 支持的字段类型
nav_order: 15
has_children: false
---

# 未签名的长场类型

这`unsigned_long` 字段类型是代表无符号的数字字段类型64-最小值为0的位整数，最大值为2 <sup> 64 </sup>＆minus;1.在以下示例中，`counter` 被映射为`unsigned_long` 场地：


```json
PUT testindex 
{
  "mappings" : {
    "properties" :  {
      "counter" : {
        "type" : "unsigned_long"
      }
    }
  }
}
```
{% include copy-curl.html %}

## 索引

用一个文档索引`unsigned_long` 价值，使用以下请求：

```json
PUT testindex/_doc/1 
{
  "counter" : 10223372036854775807
}
```
{% include copy-curl.html %}

或者，您可以使用[散装API]({{site.url}}{{site.baseurl}}/api-reference/document-apis/bulk/) 如下：

```json
POST _bulk
{ "index": { "_index": "testindex", "_id": "1" } }
{ "counter": 10223372036854775807 }
```
{% include copy-curl.html %}

如果类型字段`unsigned_long` 有`store` 参数设置为`true` （也就是说，该字段是一个存储的字段），将其存储并返回为字符串。`unsigned_long` 值不支持小数部分，因此，如果提供，小数部分将被截断。
{: .note}}

## 查询

`unsigned_long` 字段支持其他数字类型支持的大多数查询。例如，您可以在`unsigned_long` 字段：

```json
POST _search
{
  "query": {
    "term": {
      "counter": {
        "value": 10223372036854775807
      }
    }
  }
}
```
{% include copy-curl.html %}

您也可以使用范围查询：

```json
POST _search
{
  "query": {
    "range": {
      "counter": {
        "gte": 10223372036854775807
      }
    }
  }
}
```
{% include copy-curl.html %}

## 排序

您可以使用`sort` 带有`unsigned_long` 订购搜索结果的字段，例如：

```json
POST _search
{
  "sort" : [
    { 
      "counter" : { 
        "order" : "asc" 
      } 
    }
  ],
  "query": {
    "range": {
      "counter": {
        "gte": 10223372036854775807
      }
    }
  }
}
```
{% include copy-curl.html %}


一个`unsigned_long` 字段不能用作索引排序字段（在`sort.field` 索引设置）。
{： 。警告}

## 聚合

像其他数字字段一样`unsigned_long` 字段支持聚合。为了`terms` 和`multi_terms` 聚合，`unsigned_long` 值按原样使用，但对于其他聚合类型，值将转换为`double` 类型（可能会丢失精度）。以下是`terms` 聚合：

```json
POST _search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "counters": {
      "terms": { 
         "field": "counter" 
      }
    }
  }
}
```
{% include copy-curl.html %}

## 脚本

在脚本中，`unsigned_long` 返回字段作为实例`BigInteger` 班级：

```json
POST _search
{
  "query": {
    "bool": {
      "filter": {
        "script": {
          "script": "BigInteger amount = doc['counter'].value; return amount.compareTo(BigInteger.ZERO) > 0;"
        }
      }
    }
  }
}
```
{% include copy-curl.html %}


## 限制

请注意以下局限`unsigned_long` 字段类型：

- 当跨不同数字类型执行聚合时，其中一种类型是`unsigned_long`，值转换为`double` 类型和`double` 使用算术，具有很高的精确损失。

- 一个`unsigned_long` 字段不能用作索引排序字段（在`sort.field` 索引设置）。当在多个索引上执行搜索时，此限制也适用`unsigned_long` 输入至少一个索引，但在其他索引中输入不同的数字类型或类型。

