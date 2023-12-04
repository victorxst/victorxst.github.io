---
layout: default
title: CAT nodeattrs
parent: CAT API
nav_order: 35
has_children: false
redirect_from:
- /opensearch/rest-api/cat/cat-nodeattrs/
---

# 猫nodeattrs
**引入1.0**
{: .label .label-purple }

CAT NodeAttrs操作列出了自定义节点的属性。

## 例子

```
GET _cat/nodeattrs?v
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET _cat/nodeattrs
```

## URL参数

所有CAT NodeAttrs URL参数都是可选的。

除了[常见的URL参数]({{site.url}}{{site.baseurl}}/api-reference/cat/index)，您可以指定以下参数：

范围| 类型| 描述
:--- | :--- | :---
当地的| 布尔| 是否仅从本地节点返回信息，而不是从cluster_manager节点返回。默认值为false。
cluster_manager_timeout| 时间| 等待连接到群集管理器节点的时间。默认值为30秒。


## 回复

```json
node | host | ip | attr | value
odfe-node2 | 172.18.0.3 | 172.18.0.3 | testattr | test
```

