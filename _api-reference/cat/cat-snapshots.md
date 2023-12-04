---
layout: default
title: CAT 快照
parent: CAT API

nav_order: 65
has_children: false
redirect_from:
- /opensearch/rest-api/cat/cat-snapshots/
---

# 猫快照
**引入1.0**
{: .label .label-purple }

CAT快照操作列出了存储库的所有快照。

## 例子

```
GET _cat/snapshots?v
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET _cat/snapshots
```

## URL参数

所有CAT快照URL参数都是可选的。

除了[常见的URL参数]({{site.url}}{{site.baseurl}}/api-reference/cat/index)，您可以指定以下参数：

范围| 类型| 描述
:--- | :--- | :---
cluster_manager_timeout| 时间| 等待连接到群集管理器节点的时间。默认值为30秒。
时间| 时间| 指定时间的单位。例如，`5d` 或者`7h`。有关更多信息，请参阅[支持单位]({{site.url}}{{site.baseurl}}/opensearch/units/)。


## 回复

```json
index | shard | prirep | state   | docs | store | ip |       | node
plugins | 0   |   p    | STARTED |   0  |  208b | 172.18.0.4 | odfe-node1
plugins | 0   |   r    | STARTED |   0  |  208b | 172.18.0.3 |  odfe-node2          
```

