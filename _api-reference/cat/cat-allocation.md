---
layout: default
title: CAT 分配
parent: CAT API
redirect_from:
- /opensearch/rest-api/cat/cat-allocation/
nav_order: 5
has_children: false
---

# 猫分配
**引入1.0**
{: .label .label-purple }

CAT分配操作列出了索引磁盘空间的分配以及每个节点上的碎片数。

## 例子

```json
GET _cat/allocation?v
```
{% include copy-curl.html %}

要将信息限制为特定节点，请在查询之后添加节点名称：

```json
GET _cat/allocation/<node_name>
```
{% include copy-curl.html %}

如果要获取多个节点的信息，请将节点名称与逗号分开：

```json
GET _cat/allocation/node_name_1,node_name_2,node_name_3
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET _cat/allocation?v
GET _cat/allocation/<node_name>
```

## URL参数

所有CAT分配URL参数都是可选的。

除了[常见的URL参数]({{site.url}}{{site.baseurl}}/api-reference/cat/index)，您可以指定以下参数：

范围| 类型| 描述
:--- | :--- | :---
字节| 字节大小| 指定字节大小的单元。例如，`7kb` 或者`6gb`。有关更多信息，请参阅[支持单位]({{site.url}}{{site.baseurl}}/opensearch/units/)。
当地的| 布尔| 是否仅从本地节点返回信息，而不是从cluster_manager节点返回。默认值为false。
cluster_manager_timeout| 时间| 等待连接到cluster_manager节点的时间。默认值为30秒。

## 回复

以下响应表明，将八个碎片分配给了可用的两个节点：

```json
shards | disk.indices | disk.used | disk.avail | disk.total | disk.percent host | ip          | node
  8    |   989.4kb    |   25.9gb  |   32.4gb   |   58.4gb   |   44 172.18.0.4   | 172.18.0.4  | odfe-node1
  8    |   962.4kb    |   25.9gb  |   32.4gb   |   58.4gb   |   44 172.18.0.3   | 172.18.0.3  | odfe-node2
```

