---
layout: default
title: 索引分析
nav_order: 20
---

# 索引分析

索引分析在索引时间指定，用于分析[文本]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/text/) 索引文档时字段。

## 确定要使用哪种索引分析

要确定在索引文档时将哪种分析仪用于字段，OpenSearch按顺序检查以下参数：

1. 这`analyzer` 该字段的映射参数
1. 这`analysis.analyzer.default` 索引设置
1. 这`standard` 分析仪（默认）

指定索引分析时，请记住，在大多数情况下，为每个分析仪指定一个分析仪`text` 索引中的字段最有效。使用相同分析仪分析文本字段（在索引时间）和查询字符串（在查询时间）确保搜索使用与索引中存储的术语相同的术语。
{: .important }

有关验证哪个分析仪与哪个字段相关的信息，请参见[验证分析仪设置]({{site.url}}{{site.baseurl}}/analyzers/index/#verifying-analyzer-settings)。

## 为字段指定索引分析

创建索引映射时，您可以提供`analyzer` 每个参数[文本]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/text/) 场地。例如，以下请求指定`simple` 分析仪`text_entry` 场地：

```json
PUT testindex
{
  "mappings": {
    "properties": {
      "text_entry": {
        "type": "text",
        "analyzer": "simple"
      }
    }
  }
}
```
{% include copy-curl.html %}

## 为索引指定默认索引分析

如果要在索引中使用同一分析仪对所有文本字段使用相同的分析仪，则可以在`analysis.analyzer.default` 设置如下：

```json
PUT testindex
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default": {
          "type": "simple"
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

如果您不指定默认分析仪，则`standard` 使用分析仪。
{:.note}


