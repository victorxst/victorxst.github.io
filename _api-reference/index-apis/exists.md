---
layout: default
title: 索引存在
parent: 索引API
nav_order: 50
redirect_from:
  - /opensearch/rest-api/index-apis/exists/
---

# 索引存在
**引入1.0**
{: .label .label-purple }

索引存在API操作返回是否已经存在索引。

## 例子

```json
HEAD /sample-index
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
HEAD /<index-name>
```

## URL参数

所有参数都是可选的。

范围| 类型| 描述
:--- | :--- | :---
允许_no_indices| 布尔| 是否忽略不符合任何索引的通配符。默认是正确的。
Expand_WildCard| 细绳| 将通配符表达式扩展到不同的索引。将多个值与逗号相结合。可用的值全部（匹配所有索引），打开（匹配打开索引），封闭（匹配封闭索引），隐藏（匹配隐藏索引）和无（不接受通配符表达式）。默认值打开。
flat_settings| 布尔| 是否以平面形式返回设置，这可以提高可读性，尤其是对于重嵌套的设置。例如，平坦的形式"index"：{"creation_date"："123456789" } 是"index.creation_date"："123456789"。
include_defaults| 布尔| 是否将默认设置作为响应的一部分。此参数可用于识别要更新的设置的名称和当前值。
ignore_unavailable| 布尔| 如果为true，则OpenSearch不会搜索缺失或封闭的索引。默认值为false。
当地的| 布尔| 是否仅从本地节点而不是从群集管理器节点返回信息。默认值为false。


## 回复

索引存在API操作仅返回两个可能的响应代码之一：`200` -- 该指数存在，`404` -- 该索引不存在。

