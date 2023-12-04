---
layout: default
title: 节点API
has_children: true
nav_order: 50
---

# 节点API
**引入1.0**
{: .label .label-purple }

节点API使得可以检索群集中有关单个节点的信息。

## 节点过滤器

使用`<node-filters>` 参数以滤除API响应中的节点的目标集。

<样式>
表Th：第一个-的-类型 {
    宽度：25％；
}
表Th：nth-的-类型（2）{
    宽度：10％；
}
表Th：nth-的-类型（3）{
    宽度：65％；
}
</style>

范围| 类型| 描述
：--- |：-------| ：---
`<node-filters>` | 细绳| 逗号-OpenSearch用来识别群集节点的分辨率机制的分离列表。

节点过滤器支持几种节点分辨率机制：

- 预定义常数：`_local`，`_cluster_manager`， 或者`_all`。
- 确切的匹配`nodeID`
- 一个简单的情况-敏感的通配符模式匹配`node-name`，`host-name`， 或者`host-IP-address`。
- 节点角色，其中`<bool>` 值设置为`true` 或者`false`：
  - `cluster_manager:<bool>` 指的是所有集群管理器-合格的节点。
  - `data:<bool>` 指的是所有数据节点。
  - `ingest:<bool>` 指的是所有摄入节点。
  - `voting_only:<bool>` 指的是所有投票-只有节点。
  - `ml:<bool>` 指的是所有机器学习（ML）节点。
  - `coordinating_only:<bool>` 指的是所有协调-只有节点。
- 一个简单的情况-敏感的通配符模式匹配节点属性：`<node attribute*>:<attribute value*>`。通配符匹配模式可以同时使用键和值。

分辨率机制按客户端指定的顺序依次应用。每个机制规范可以添加或删除节点。

要从当选的集群管理器节点中获取统计信息，请使用以下查询：

```json
GET /_nodes/_cluster_manager/stats
```
{% include copy-curl.html %}

从节点获取数据的统计信息-只有节点，请使用以下查询：

```json
GET /_nodes/data:true/stats
```
{% include copy-curl.html %}

### 解决方案机制

分辨机制的顺序被依次应用，每个分辨率机制都可以添加或删除节点。以下示例产生不同的结果。

要从除集群管理器节点以外的所有节点中获取统计信息，请使用以下查询：

```json
GET /_nodes/_all,cluster_manager:false/stats
```
{% include copy-curl.html %}

但是，如果切换分辨率机制，结果将包括所有群集节点，包括群集管理器节点：

```json
GET /_nodes/cluster_manager:false,_all/stats
```
{% include copy-curl.html %}

