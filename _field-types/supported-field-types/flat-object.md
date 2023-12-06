---
layout: default
title: 平坦对象
nav_order: 43
has_children: false
parent: Object字段类型
grand_parent: 支持的字段类型
redirect_from:
  - /field-types/flat-object/
---

# 平面Object字段类型

在OpenSearch中，您无需在索引文档之前指定映射。如果您不指定映射，则OpenSearch使用[动态映射]({{site.url}}{{site.baseurl}}/field-types/index#dynamic-mapping) 自动映射文档中的每个字段及其子字段。当您摄入诸如日志之类的文档时，您可能不知道每个字段的子字段名称并提前键入。在这种情况下，动态映射所有新的子字段都可以快速导致"mapping explosion," 越来越多的字段可能会降低集群的性能。

平面Object字段类型通过将整个JSON对象视为字符串来解决此问题。使用标准点路径符号可以访问JSON对象中的子字段，但没有为快速查找而索引。

点符号中的最大场值长度为2 <sup> 24 </sup>&minus;；1。
{:.note}

平坦Object字段类型提供以下好处：

- 有效的读取：获取性能类似于关键字字段的性能。
- 内存效率：将整个复杂的JSON对象存储在一个字段中而不索引其所有子字段会减少索引中的字段数。
- 空间效率：OpenSearch不会为平面对象中的子字段创建倒置索引，从而节省空间。
- 迁移的兼容性：您可以从支持类似的平面类型的系统中迁移数据。

当字段及其子字段被读取而不用作搜索条件时，将字段映射为平坦对象，因为子字段没有索引。平面对象对于具有大量字段的对象或您不预先知道键时有用。

平面对象支持有或没有点路径符号的精确匹配查询。有关支持的查询类型的完整列表，请参见[支持的查询](#supported-queries)。

在文档中搜索嵌套字段的特定值可能会效率低下，因为它可能需要对索引进行全面扫描，这可能是一个昂贵的操作。
{:.note}

平坦对象不支持：

- 类型-具体解析。
- 数值操作，例如数值比较或数值分类。
- 文本分析。
- 突出显示。
- 使用点表示法的汇总。
- 通过子场过滤。

## 支持的查询

平面Object字段类型支持以下查询：

- [学期]({{site.url}}{{site.baseurl}}/query-dsl/term/term/) 
- [术语]({{site.url}}{{site.baseurl}}/query-dsl/term/terms/) 
- [术语设置]({{site.url}}{{site.baseurl}}/query-dsl/term/terms-set/)  
- [字首]({{site.url}}{{site.baseurl}}/query-dsl/term/prefix/) 
- [范围]({{site.url}}{{site.baseurl}}/query-dsl/term/range/) 
- [匹配]({{site.url}}{{site.baseurl}}/query-dsl/full-text/match/) 
- [并发-匹配]({{site.url}}{{site.baseurl}}/query-dsl/full-text/multi-match/) 
- [请求参数]({{site.url}}{{site.baseurl}}/query-dsl/full-text/query-string/) 
- [简单查询字符串]({{site.url}}{{site.baseurl}}/query-dsl/full-text/simple-query-string/) 
- [存在]({{site.url}}{{site.baseurl}}/query-dsl/term/exists/) 

## 限制

以下限制适用于OpenSearch 2.7中的平面对象：

- 平面对象不支持开放参数。
- 对于检索子字段的值，不支持无痛的脚本和通配符查询。

该功能是为将来发布的。

## 使用平坦对象

以下示例说明将字段映射为平坦对象，用平面对象字段索引文档，并在这些文档中搜索平面对象的叶值。

首先，为您的索引创建映射，其中`issue` 是类型`flat_object`：

```json
PUT /test-index/
{
  "mappings": {
    "properties": {
      "issue": {
        "type": "flat_object"
      }
    }
  }
}
```
{% include copy-curl.html %}

接下来，索引两个文档，带有平面对象字段：

```json
PUT /test-index/_doc/1
{
  "issue": {
    "number": "123456",
    "labels": {
      "version": "2.1",
      "backport": [
        "2.0",
        "1.3"
      ],
      "category": {
        "type": "API",
        "level": "enhancement"
      }
    }
  }
}
```
{% include copy-curl.html %}

```json
PUT /test-index/_doc/2
{
  "issue": {
    "number": "123457",
    "labels": {
      "version": "2.2",
      "category": {
        "type": "API",
        "level": "bug"
      }
    }
  }
}
```
{% include copy-curl.html %}

要搜索平面对象的叶值，请使用get或post请求。即使您不知道字段名称，也可以在整个平面对象中搜索叶值。例如，以下请求搜索所有标记为错误的问题：

```json
GET /test-index/_search
{
  "query": {
    "match": {"issue": "bug"}
  }
}
```

另外，如果您知道要在其中搜索的子字段名称，请在DOT符号中提供该字段的路径：

```json
GET /test-index/_search
{
  "query": {
    "match": {"issue.labels.category.level": "bug"}
  }
}
```
{% include copy-curl.html %}

在这两种情况下，响应都是相同的，并且包含文件2：

```json
{
  "took": 1,
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
    "max_score": 1.0303539,
    "hits": [
      {
        "_index": "test-index",
        "_id": "2",
        "_score": 1.0303539,
        "_source": {
          "issue": {
            "number": "123457",
            "labels": {
              "version": "2.2",
              "category": {
                "type": "API",
                "level": "bug"
              }
            }
          }
        }
      }
    ]
  }
}
```

使用前缀查询，您可以搜索以从`2.`：

```json
GET /test-index/_search
{
  "query": {
    "prefix": {"issue.labels.version": "2."}
  }
}
```

使用范围查询，您可以搜索版本2.0的所有问题--2.1：

```json
GET /test-index/_search
{
  "query": {
    "range": {
      "issue": {
        "gte": "2.0",
        "lte": "2.1"
      }
    }
  }
}
```

## 将子场定义为平坦对象

您可以将JSON对象的子字段定义为平坦对象。例如，使用以下查询定义`issue.labels` 作为`flat_object`：

```json
PUT /test-index/
{
  "mappings": {
    "properties": {
      "issue": {
        "properties": {
          "number": {
            "type": "double"
          },
          "labels": {
            "type": "flat_object"
          }
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

因为`issue.number` 不是平面对象的一部分，您可以使用它来汇总和排序文档

