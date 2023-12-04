---
layout: default
title: Count
nav_order: 21
redirect_from:
 - /opensearch/rest-api/count/
---

# 数数
**引入1.0**
{: .label .label-purple }

计数API使您可以快速访问与查询匹配的文档数量。
您也可以使用它来检查索引，数据流或群集的文档计数。

## 例子

要查看与查询匹配的文档数量：

```json
GET opensearch_dashboards_sample_data_logs/_count
{
  "query": {
    "term": {
      "response": "200"
    }
  }
}
```
{% include copy-curl.html %}

以下呼叫搜索API会产生同等的结果：

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "query": {
    "term": {
      "response": "200"
    }
  },
  "size": 0,
  "track_total_hits": true
}
```
{% include copy-curl.html %}

在索引中查看文档数量：

```json
GET opensearch_dashboards_sample_data_logs/_count
```
{% include copy-curl.html %}

要检查一个文档的数量[数据流]({{site.url}}{{site.baseurl}}/opensearch/data-streams/)，用数据流名称替换索引名称。

要查看群集中的文档数量：

```json
GET _count
```
{% include copy-curl.html %}

或者，您可以使用[猫索引]({{site.url}}{{site.baseurl}}/api-reference/cat/cat-indices/) 和[猫数]({{site.url}}{{site.baseurl}}/api-reference/cat/cat-count/) API查看每个索引或数据流的文档数量。
{: .note }


## 路径和HTTP方法

```
GET <target>/_count/<id>
POST <target>/_count/<id>
```


## URL参数

所有计数参数都是可选的。

范围| 类型| 描述
:--- | :--- | :---
`allow_no_indices` | 布尔| 如果false，请求将返回错误，如果任何通配符表达式或索引别名针对任何封闭或丢失的索引。默认值为false。
`analyzer` | 细绳| 用于查询字符串中的分析仪。
`analyze_wildcard` | 布尔| 指定是否分析通配符和前缀查询。默认值为false。
`default_operator` | 细绳| 指示字符串查询的默认运算符应该是和或或。默认为或。
`df` | 细绳| 如果查询字符串中未提供字段前缀，则默认字段。
`expand_wildcards` | 细绳| 指定通配符表达式可以匹配的索引类型。支持逗号-分离的值。有效值是`all` （匹配任何索引），`open` （匹配开放，非-隐藏索引），`closed` （匹配封闭，非-隐藏索引），`hidden` （匹配隐藏索引），`none` （拒绝通配符表达）。默认为`open`。
`ignore_unavailable` | 布尔| 指定响应中是否包括缺失或封闭索引。默认值为false。
`lenient` | 布尔| 指定如果查询具有格式错误，OpenSearch是否应接受请求（例如，查询文本字段中的整数）。默认值为false。
`min_score` | 漂浮|仅包括最低文档`_score` 结果的价值。
`routing` | 细绳| 用于将操作路由到特定碎片的值。
`preference` | 细绳| 指定哪些分片或节点OpenSearch应执行计数操作。
`terminate_after` | 整数| 文档数量的最大数量OpenSearch应在终止请求之前进行处理。

## 回复

```json
{
  "count" : 14074,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  }
}
```

