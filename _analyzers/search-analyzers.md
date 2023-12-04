---
layout: default
title: 搜索分析
nav_order: 30
---

# 搜索分析

在查询时间指定搜索分析，并在运行完整时用于分析查询字符串-文字查询[文本]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/text/) 场地。

## 确定要使用哪种搜索分析

要确定在查询时间用于查询字符串的分析仪，OpenSearch按顺序检查以下参数：

1. 这`analyzer` 查询的参数
1. 这`search_analyzer` 该字段的映射参数
1. 这`analysis.analyzer.default_search` 索引设置
1. 这`analyzer` 该字段的映射参数
1. 这`standard` 分析仪（默认）

在大多数情况下，指定与索引分析仪不同的搜索分析不是必需的，并且可能对搜索结果相关性产生负面影响或导致意外搜索结果。
{: .warning}

有关验证哪个分析仪与哪个字段相关的信息，请参见[验证分析仪设置]({{site.url}}{{site.baseurl}}/analyzers/index/#verifying-analyzer-settings)。

## 为查询字符串指定搜索分析

在查询时间中指定要在查询时间使用的分析仪的名称`analyzer` 场地：

```json
GET shakespeare/_search
{
  "query": {
    "match": {
      "text_entry": {
        "query": "speak the truth",
        "analyzer": "english"
      }
    }
  }
}
```
{% include copy-curl.html %}

有效值[建造-在分析仪中]({{site.url}}{{site.baseurl}}/analyzers/index#built-in-analyzers) 是`standard`，，，，`simple`，，，，`whitespace`，，，，`stop`，，，，`keyword`，，，，`pattern`，，，，`fingerprint`，或任何受支持的[语言分析仪]({{site.url}}{{site.baseurl}}/analyzers/language-analyzers/)。

## 为字段指定搜索分析

创建索引映射时，您可以提供`search_analyzer` 每个参数[文本]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/text/) 场地。提供`search_analyzer`，您还必须提供`analyzer` 参数，指定[索引分析仪]({{site.url}}{{site.baseurl}}/analyzers/index-analyzers/) 在索引时间使用。

例如，以下请求指定`simple` 分析仪作为索引分析仪和`whitespace` 分析仪作为搜索分析`text_entry` 场地：

```json
PUT testindex
{
  "mappings": {
    "properties": {
      "text_entry": {
        "type": "text",
        "analyzer": "simple",
        "search_analyzer": "whitespace"
      }
    }
  }
}
```
{% include copy-curl.html %}

## 为索引指定默认搜索分析

如果您想在搜索时间分析所有查询字符串，则可以在`analysis.analyzer.default_search` 环境。提供`analysis.analyzer.default_search`，您还必须提供`analysis.analyzer.default` 参数，指定[索引分析仪]({{site.url}}{{site.baseurl}}/analyzers/index-analyzers/) 在索引时间使用。

例如，以下请求指定`simple` 分析仪作为索引分析仪和`whitespace` 分析仪作为搜索分析`testindex` 指数：

```json
PUT testindex
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default": {
          "type": "simple"
        },
        "default_search": {
          "type": "whitespace"
        }
      }
    }
  }
}

```
{% include copy-curl.html %}

