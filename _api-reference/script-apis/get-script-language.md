---
layout: default
title: 获取脚本语言
parent: 脚本API
nav_order: 6
---

# 获取脚本语言
**引入1.0**
{: .label .label-purple }

GET脚本语言API操作检索所有支持的脚本语言及其上下文。

#### 示例请求

```json
GET _script_language
```
{% include copy-curl.html %}

#### 示例响应

这`GET _script_language` 请求返回每种语言的可用上下文：

```json
{
  "types_allowed" : [
    "inline",
    "stored"
  ],
  "language_contexts" : [
    {
      "language" : "expression",
      "contexts" : [
        "aggregation_selector",
        "aggs",
        "bucket_aggregation",
        "field",
        "filter",
        "number_sort",
        "score",
        "terms_set"
      ]
    },
    {
      "language" : "mustache",
      "contexts" : [
        "template"
      ]
    },
    {
      "language" : "opensearch_query_expression",
      "contexts" : [
        "aggs",
        "filter"
      ]
    },
    {
      "language" : "painless",
      "contexts" : [
        "aggregation_selector",
        "aggs",
        "aggs_combine",
        "aggs_init",
        "aggs_map",
        "aggs_reduce",
        "analysis",
        "bucket_aggregation",
        "field",
        "filter",
        "ingest",
        "interval",
        "moving-function",
        "number_sort",
        "painless_test",
        "processor_conditional",
        "score",
        "script_heuristic",
        "similarity",
        "similarity_weight",
        "string_sort",
        "template",
        "terms_set",
        "trigger",
        "update"
      ]
    }
  ]
}
```

## 响应字段

该请求包含以下响应字段。

场地| 数据类型| 描述| 
:--- | :--- | :---
types_lowered| 字符串列表| 启用的脚本类型，由`script.allowed_types` 环境。可能含有`inline` 和/或`stored`。
Laging_contexts| 对象列表| 一个对象列表，每个对象将支持的语言映射到其可用上下文中。
language_contexts.language| 细绳| 注册脚本语言的名称。
Laging_contexts.contexts| 字符串列表| 该语言的所有上下文列表，由`script.allowed_contexts` 环境。

