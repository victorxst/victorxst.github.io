---
layout: default
title: 删除索引
parent: 索引API
nav_order: 35
redirect_from:
  - /opensearch/rest-api/index-apis/delete-index/
---

# 删除索引
**引入1.0**
{: .label .label-purple }

如果您不再需要索引，则可以使用删除索引API操作将其删除。

## 例子

```json
DELETE /sample-index
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
DELETE /<index-name>
```

## URL参数

所有参数都是可选的。

范围| 类型| 描述
:--- | :--- | :---
允许_no_indices| 布尔| 是否忽略不符合任何索引的通配符。默认是正确的。
Expand_WildCard| 细绳| 将通配符表达式扩展到不同的索引。将多个值与逗号相结合。可用的值全部（匹配所有索引），打开（匹配打开索引），封闭（匹配封闭索引），隐藏（匹配隐藏索引）和无（不接受通配符表达式），必须将其用于开放，封闭，封闭，封闭，或两者。默认值打开。
ignore_unavailable| 布尔| 如果为true，则OpenSearch不会在响应中包括缺失或封闭索引。
cluster_manager_timeout| 时间| 等待连接到群集管理器节点多长时间。默认为`30s`。
暂停| 时间| 等待响应返回多长时间。默认为`30s`。


## 回复
```json
{
  "acknowledged": true
}
```

