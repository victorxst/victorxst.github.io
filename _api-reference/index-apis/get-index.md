---
layout: default
title: 获取索引
parent: 索引API
nav_order: 40
redirect_from:
  - /opensearch/rest-api/index-apis/get-index/
---

# 获取索引
**引入1.0**
{: .label .label-purple }

您可以使用GET索引API操作返回有关索引的信息。

## 例子

```json
GET /sample-index
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET /<index-name>
```

## URL参数

所有参数都是可选的。

范围| 类型| 描述
:--- | :--- | :---
允许_no_indices| 布尔| 是否忽略不符合任何索引的通配符。默认是正确的。
Expand_WildCard| 细绳| 将通配符表达式扩展到不同的索引。将多个值与逗号相结合。可用的值全部（匹配所有索引），打开（匹配打开索引），封闭（匹配封闭索引），隐藏（匹配隐藏索引）和无（不接受通配符表达式），必须将其用于开放，封闭，封闭，封闭，或两者。默认值打开。
flat_settings| 布尔| 是否以平面形式返回设置，这可以提高可读性，尤其是对于重嵌套的设置。例如，平坦的形式"index"：{"creation_date"："123456789" } 是"index.creation_date"："123456789"。
include_defaults| 布尔| 是否将默认设置作为响应的一部分。此参数可用于识别要更新的设置的名称和当前值。
ignore_unavailable| 布尔| 如果为true，则OpenSearch不会在响应中包括缺失或封闭索引。
当地的| 布尔| 是否仅从本地节点而不是从群集管理器节点返回信息。默认值为false。
cluster_manager_timeout| 时间| 等待连接到群集管理器节点多长时间。默认为`30s`。


## 回复
```json
{
  "sample-index1": {
    "aliases": {},
    "mappings": {},
    "settings": {
      "index": {
        "creation_date": "1633044652108",
        "number_of_shards": "2",
        "number_of_replicas": "1",
        "uuid": "XcXA0aZ5S0aiqx3i1Ce95w",
        "version": {
          "created": "135217827"
        },
        "provided_name": "sample-index1"
      }
    }
  }
}
```

## 响应身体场

场地| 描述
:--- | :---
别名| 与索引相关的任何别名。
映射| 索引中的任何映射。
设置| 索引的设置
创建日期| 创建索引的时间的Unix时期时间。
number_of_shards| 该指数有多少碎片。
number_of_replicas| 该索引有多少个复制品。
UUID| 索引的uuid。
创建| 创建索引时OpenSearch的版本。
提供_name| 索引的名称。

