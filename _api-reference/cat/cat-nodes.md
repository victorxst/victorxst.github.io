---
layout: default
title: CAT 节点操作
parent: CAT API

nav_order: 40
has_children: false
redirect_from:
- /opensearch/rest-api/cat/cat-nodes/
---

# 猫节点
**引入1.0**
{: .label .label-purple }

猫节点操作列出节点-级别信息，包括节点角色和负载指标。

一些重要的节点指标是`pid`，`name`，`cluster_manager`，`ip`，`port`，`version`，`build`，`jdk`， 随着`disk`，`heap`，`ram`， 和`file_desc`。

## 例子

```
GET _cat/nodes?v
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET _cat/nodes
```

## URL参数

所有CAT节点URL参数都是可选的。

除了[常见的URL参数]({{site.url}}{{site.baseurl}}/api-reference/cat/index)，您可以指定以下参数：

范围| 类型| 描述
:--- | :--- | :---
字节| 字节大小| 指定字节大小的单元。例如，`7kb` 或者`6gb`。有关更多信息，请参阅[支持单位]({{site.url}}{{site.baseurl}}/opensearch/units/)。
full_id| 布尔| 如果为true，请返回完整的节点ID。如果false，请返回缩短的节点ID。默认为false。
当地的| 布尔| 是否仅从本地节点返回信息，而不是从cluster_manager节点返回。默认值为false。
cluster_manager_timeout| 时间| 等待连接到群集管理器节点的时间。默认值为30秒。
时间| 时间| 指定时间的单位。例如，`5d` 或者`7h`。有关更多信息，请参阅[支持单位]({{site.url}}{{site.baseurl}}/opensearch/units/)。
包括_unloaded_segments| 布尔| 是否包括未加载到内存中的段中的信息。默认值为false。


## 回复

```json
ip       |   heap.percent | ram.percent | cpu load_1m | load_5m | load_15m | node.role | node.roles |     cluster_manager |  name
10.11.1.225  |         31   |    32  | 0  |  0.00  |  0.00   | di  | data,ingest,ml  | - |  data-e5b89ad7
```

