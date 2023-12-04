---
layout: default
title: CAT 集群管理器
parent: CAT API
redirect_from:
 - /opensearch/rest-api/cat/cat-master/
nav_order: 30
has_children: false
---

# CAT CLUSTER_MANAGER
**引入1.0**
{: .label .label-purple }

CAT群集管理器操作列出了有助于识别当选集群管理器节点的信息。

## 例子

```
GET _cat/cluster_manager?v
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET _cat/cluster_manager
```

## URL参数

所有CAT群集管理器URL参数都是可选的。

除了[常见的URL参数]({{site.url}}{{site.baseurl}}/api-reference/cat/index)，您可以指定以下参数：

范围| 类型| 描述
:--- | :--- | :---
cluster_manager_timeout| 时间| 等待连接到群集管理器节点的时间。默认值为30秒。

## 回复

```json
id                     |   host     |     ip     |   node
ZaIkkUd4TEiAihqJGkp5CA | 172.18.0.3 | 172.18.0.3 | opensearch-node2
```

