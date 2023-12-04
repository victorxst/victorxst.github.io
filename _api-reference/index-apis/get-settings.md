---
layout: default
title: 获取设置
parent: 索引API
nav_order: 45
redirect_from:
  - /opensearch/rest-api/index-apis/get-index/
---

# 获取设置
**引入1.0**
{: .label .label-purple }

GET设置API操作返回索引中的所有设置。

## 例子

```json
GET /sample-index1/_settings
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET /_settings
GET /<target-index>/_settings
GET /<target-index>/_settings/<setting>
```

## URL参数

所有获取设置参数都是可选的。

范围| 数据类型| 描述
:--- | :--- | :---
＆lt;目标-索引＆gt;| 细绳| 从中获得设置的索引。可以是逗号-分开列表以获取多个索引的设置，或使用`_all` 要从集群中的所有索引返回设置。
＆lt;设置＆gt;| 细绳| 过滤以返回特定的设置。
允许_no_indices| 布尔| 是否忽略不符合任何索引的通配符。默认为`true`。
Expand_WildCard| 细绳| 将通配符表达式扩展到不同的索引。将多个值与逗号相结合。可用值是`all` （匹配所有索引），`open` （匹配打开索引），`closed` （匹配封闭索引），`hidden` （匹配隐藏索引），`none` （请勿接受通配符表达式），必须与`open`，`closed`， 或两者。默认为`open`。
flat_settings| 布尔| 是否以平面形式返回设置，这可以提高可读性，尤其是对于重嵌套的设置。例如，“索引”的扁平形式：{“ creation_date”：“ 123456789”}是“ index.creation_date”：“ 123456789”。
include_defaults| 细绳| 是否在响应中包括默认设置，包括OpenSearch插件中使用的设置。默认值为false。
ignore_unavailable| 布尔| 如果为true，则OpenSearch不会在响应中包括缺失或封闭索引。
当地的| 布尔| 是否仅从本地节点而不是群集管理节点返回信息。默认值为false。
cluster_manager_timeout| 时间| 等待连接到群集管理器节点多长时间。默认为`30s`。

## 回复

```json
{
  "sample-index1": {
    "settings": {
      "index": {
        "creation_date": "1622672553417",
        "number_of_shards": "1",
        "number_of_replicas": "1",
        "uuid": "GMEA0_TkSaamrnJSzNLzwg",
        "version": {
          "created": "135217827",
          "upgraded": "135238227"
        },
        "provided_name": "sample-index1"
      }
    }
  }
}
```

