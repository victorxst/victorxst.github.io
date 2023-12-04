---
layout: default
title: CAT 碎片
parent: CAT API

nav_order: 60
has_children: false
redirect_from:
- /opensearch/rest-api/cat/cat-shards/
---

# 猫碎片
**引入1.0**
{: .label .label-purple }

猫碎片操作列出了所有主要和复制碎片的状态以及它们的分布方式。

## 例子

```
GET _cat/shards?v
```
{% include copy-curl.html %}

要仅查看有关特定索引碎片的信息，请在查询后添加索引名称。

```
GET _cat/shards/<index>?v
```
{% include copy-curl.html %}

如果您想获得多个索引的信息，请将索引与逗号分开：

```
GET _cat/shards/index1,index2,index3
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET _cat/shards
```

## URL参数

所有猫碎片URL参数都是可选的。

除了[常见的URL参数]({{site.url}}{{site.baseurl}}/api-reference/cat/index)，您可以指定以下参数：

范围| 类型| 描述
:--- | :--- | :---
字节| 字节大小| 指定字节大小的单元。例如，`7kb` 或者`6gb`。有关更多信息，请参阅[支持单位]({{site.url}}{{site.baseurl}}/opensearch/units/)。
当地的| 布尔| 是否仅从本地节点返回信息，而不是从cluster_manager节点返回。默认值为false。
cluster_manager_timeout| 时间| 等待连接到cluster_manager节点的时间。默认值为30秒。
时间| 时间| 指定时间的单位。例如，`5d` 或者`7h`。有关更多信息，请参阅[支持单位]({{site.url}}{{site.baseurl}}/opensearch/units/)。


## 回复

```json
index | shard | prirep | state   | docs | store | ip |       | node
plugins | 0   |   p    | STARTED |   0  |  208b | 172.18.0.4 | odfe-node1
plugins | 0   |   r    | STARTED |   0  |  208b | 172.18.0.3 |  odfe-node2          
```

