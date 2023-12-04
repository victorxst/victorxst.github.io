---
layout: default
title: 更新设置
parent: 索引API
nav_order: 75
redirect_from:
  - /opensearch/rest-api/index-apis/update-settings/
---

# 更新设置
**引入1.0**
{: .label .label-purple }

您可以使用更新设置API操作更新索引-级别设置。您可以随时更改动态索引设置，但是创建索引后无法更改静态设置。有关静态和动态索引设置的更多信息，请参见[创建索引]({{site.url}}{{site.baseurl}}/api-reference/index-apis/create-index/)。

除了静态和动态索引设置外，您还可以更新单个插件的设置。要获取可更新设置的完整列表，请运行`GET <target-index>/_settings?include_defaults=true`。

## 例子

```json
PUT /sample-index1/_settings
{
  "index.plugins.index_state_management.rollover_skip": true,
  "index": {
    "number_of_replicas": 4
  }
}
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
PUT /<target-index>/_settings
```

## 查询参数

所有更新设置参数都是可选的。

范围| 数据类型| 描述
:--- | :--- | :---
允许_no_indices| 布尔| 是否忽略不符合任何索引的通配符。默认为`true`。
Expand_WildCard| 细绳| 将通配符表达式扩展到不同的索引。将多个值与逗号相结合。可用值是`all` （匹配所有索引），`open` （匹配打开索引），`closed` （匹配封闭索引），`hidden` （匹配隐藏索引），`none` （请勿接受通配符表达式），必须与`open`，`closed`， 或两者。默认为`open`。
flat_settings| 布尔| 是否以平面形式返回设置，这可以提高可读性，尤其是对于重嵌套的设置。例如，“索引”的扁平形式：{“ creation_date”：“ 123456789”}是“ index.creation_date”：“ 123456789”。
ignore_unavailable| 布尔| 如果为true，则OpenSearch不会在响应中包括缺失或封闭索引。
preserve_existing| 布尔| 是否保留现有的索引设置。默认值为false。
cluster_manager_timeout| 时间| 等待连接到群集管理器节点多长时间。默认为`30s`。
暂停| 时间| 等待连接返回多长时间。默认为`30s`。

## 请求身体

请求主体必须要更新的所有索引设置。

```json
{
  "index.plugins.index_state_management.rollover_skip": true,
  "index": {
    "number_of_replicas": 4
  }
}
```

## 回复

```json
{
    "acknowledged": true
}
```

