---
layout: default
title: term
parent: 术语级查询
grand_parent: 查询DSL
nav_order: 70
---

# term查询

使用`term` 查询在字段中搜索确切的术语。例如，以下查询搜索具有确切行号的行：

```json
GET shakespeare/_search
{
  "query": {
    "term": {
      "line_id": {
        "value": "61809"
      }
    }
  }
}
```
{% include copy-curl.html %}

当索引文档时`text` 字段是[分析]({{site.url}}{{site.baseurl}}/analyzers/index/)。分析包括标记和降低文本并删除标点符号。与众不同`match` 查询，分析查询文本，`term` 查询仅与确切的术语匹配，因此可能无法返回相关结果。避免使用`term` 查询`text` 字段。有关更多信息，请参阅[学期-水平和饱满-比较文本查询]({{site.url}}{{site.baseurl}}/query-dsl/term-vs-full-text/)。

您可以指定查询在`case_insensitive` 范围：

```json
GET shakespeare/_search
{
  "query": {
    "term": {
      "speaker": {
        "value": "HAMLET",
        "case_insensitive": true
      }
    }
  }
}
```
{% include copy-curl.html %}

尽管有任何差异，但响应包含匹配文档：

```json
"hits": {
  "total": {
    "value": 1582,
    "relation": "eq"
  },
  "max_score": 2,
  "hits": [
    {
      "_index": "shakespeare",
      "_id": "32700",
      "_score": 2,
      "_source": {
        "type": "line",
        "line_id": 32701,
        "play_name": "Hamlet",
        "speech_number": 9,
        "line_number": "1.2.66",
        "speaker": "HAMLET",
        "text_entry": "[Aside]  A little more than kin, and less than kind."
      }
    },
  ...
}
```

## 参数

查询接受字段的名称（`<field>`）作为顶部-级别参数：

```json
GET _search
{
  "query": {
    "term": {
      "<field>": {
        "value": "sample",
        ... 
      }
    }
  }
}
```
{% include copy-curl.html %}

这`<field>` 接受以下参数。除所有参数外`value` 是可选的。

范围| 数据类型| 描述
:--- | :--- | :---
`value` | 细绳| 在指定的字段中搜索的术语`<field>`。仅当结果的字段值与正确的间距和资本化完全匹配时，仅在结果中返回文档。
`boost` | 漂浮的-观点| 通过给定的乘数增强查询。对于包含多个查询的搜索很有用。[0，1）中的值降低了相关性，并且值大于1的相关性。默认为`1`。
`case_insensitive` | 布尔| 如果`true`，允许案例-该值与索引字段值的不敏感匹配。默认为`false` （案例灵敏度由字段的映射确定）

