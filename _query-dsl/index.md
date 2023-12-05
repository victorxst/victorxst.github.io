---
layout: default
title: 查询DSL
nav_order: 2
has_children: true
nav_exclude: true
has_toc: false
redirect_from:
  - /opensearch/query-dsl/
  - /opensearch/query-dsl/index/
  - /docs/opensearch/query-dsl/
  - /query-dsl/query-dsl/
  - /query-dsl/
---

{％- 评论-％}这`/docs/opensearch/query-dsl/` 重定向专门支持OpenSearch仪表板1.0.0中的UI链接。{％- 终点-％}

# 查询DSL

OpenSearch提供了一种称为 *查询域的搜索语言-特定语言（DSL）*您可以用来搜索数据。查询DSL是一种具有JSON接口的灵活语言。

使用查询DSL，您需要在`query` 搜索的参数。OpenSearch中最简单的搜索之一使用`match_all` 查询，与索引中的所有文档匹配：

```json
GET testindex/_search
{
  "query": {
     "match_all": { 
     }
  }
}
```

查询可以由许多查询子句组成。您可以将查询条款结合在一起以产生复杂的查询。

从广义上讲，您可以将查询分为两个类别---*叶子查询*和*复合查询*：

- **叶子查询**：LEAF查询在某个字段或字段中搜索指定值。您可以自己使用叶子查询。它们包括以下查询类型：

    - [满的-文本查询]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/full-text/index/)：完整-文本查询以搜索文本文档。对于经过分析的文本字段搜索，完整-文本查询使用在字段被索引时使用的相同分析仪将查询字符串分为术语。对于确切的价值搜索，完整-文本查询在不应用文本分析的情况下查找指定值。

    - [学期-级查询]({{site.url}}{{site.baseurl}}/query-dsl/term/index/)：使用术语-级别查询以搜索确切术语的文档，例如ID或值范围。学期-级别查询不会通过相关得分分析搜索术语或排序结果。

    - [地理和XY查询]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/geo-and-xy/index/)：使用地理查询搜索包含地理数据的文档。使用XY查询搜索两个在两个中包含点和形状的文档-维坐标系。

    - 加入查询：使用加入查询搜索嵌套字段或返回与特定查询相匹配的父和子文档。加入查询类型包括`nested`，，，，`has_child`，，，，`has_parent`， 和`parent_id` 查询。

    - [跨度查询]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/span-query/)：使用跨度查询执行精确的位置搜索。跨度查询很低-级别，可以控制指定查询条款的顺序和接近性的特定查询。它们主要用于搜索法律文件。

    - [专门查询]({{site.url}}{{site.baseurl}}/query-dsl/specialized/index/)：专门查询包括所有其他查询类型（`distance_feature`，，，，`more_like_this`，，，，`percolate`，，，，`rank_feature`，，，，`script`，，，，`script_score`， 和`wrapper`）。

- **复合查询**：复合查询是多个叶子或复合条款的包装器，可以结合其结果或修改其行为。它们包括布尔值，最大分离，恒定得分，功能分数和提高查询类型。要了解更多，请参阅[复合查询]({{site.url}}{{site.baseurl}}/query-dsl/compound/index/)。

## 关于文本字段中Unicode特殊字符的注释

由于与Unicode特殊字符相关的单词边界，Unicode标准分析仪无法索引[文本字段类型]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/text/) 当包含这些特殊字符之一时，值为整体值。结果，标准分析仪将包含特殊字符的文本字段值解析为由特殊字符隔开的多个值，从而有效地将其两侧的不同元素示意。这可能导致对文档的无意过滤，并可能损害其访问权限的控制。

以下示例说明了包含特殊字符的值，标准分析仪将不正确地解析。在此示例中，该值中连字符/减号的存在阻止分析仪区分两个不同的用户`user.id` 并将它们解释为同一：

```json
{
  "bool": {
    "must": {
      "match": {
        "user.id": "User-1"
      }
    }
  }
}
```

```json
{
  "bool": {
    "must": {
      "match": {
        "user.id": "User-2"
      }
    }
  }
}
```

为了避免使用查询DSL或REST API时，您可以使用自定义分析仪或将字段映射为`keyword`，执行确切的-匹配搜索。看[关键字字段类型]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/keyword/) 对于后一个选项。

对于使用时应避免的字符列表`text` 现场类型，请参阅[单词边界](https://unicode.org/reports/tr29/#Word_Boundaries)。

## 昂贵的查询

昂贵的查询可以消耗大量内存，并导致群集性能下降。以下查询可能是资源消耗的：

- [`fuzzy`]({{site.url}}{{site.baseurl}}/query-dsl/term/fuzzy/) 查询
- [`prefix`]({{site.url}}{{site.baseurl}}/query-dsl/term/prefix/) 查询
- [`range`]({{site.url}}{{site.baseurl}}/query-dsl/term/range/) 查询[`text`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/text/) 和[`keyword`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/keyword/) 字段
- [`regexp`]({{site.url}}{{site.baseurl}}/query-dsl/term/regexp/) 查询
- [`wildcard`]({{site.url}}{{site.baseurl}}/query-dsl/term/wildcard/) 查询
- [`query_string`]({{site.url}}{{site.baseurl}}/query-dsl/full-text/query-string/) 内部转换为前缀查询的查询

为了禁止昂贵的查询，您可以禁用`search.allow_expensive_queries` 集群设置如下：

```json
PUT _cluster/settings
{
  "persistent": {
    "search.allow_expensive_queries": false
  }
}
```
{％包含副本-curl.html％}

要跟踪昂贵的查询，请启用[慢记录]({{site.url}}{{site.baseurl}}/monitoring-your-cluster/logs/#slow-logs)。
{： 。提示

