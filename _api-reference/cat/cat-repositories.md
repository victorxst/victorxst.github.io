---
layout: default
title: CAT 存储库
parent: CAT API

nav_order: 52
has_children: false
redirect_from:
 - /opensearch/rest-api/cat/cat-repositories/
---

# 猫存储库
**引入1.0**
{: .label .label-purple }

CAT存储库操作列出了群集的所有快照存储库。

## 例子

```
GET _cat/repositories?v
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET _cat/repositories
```

## URL参数

所有CAT存储库URL参数都是可选的。

除了[常见的URL参数]({{site.url}}{{site.baseurl}}/api-reference/cat/index)，您可以指定以下参数：

范围| 类型| 描述
:--- | :--- | :---
当地的| 布尔| 是否仅从本地节点返回信息，而不是从cluster_manager节点返回。默认值为false。
cluster_manager_timeout| 时间| 等待连接到cluster_manager节点的时间。默认值为30秒。


## 回复

```json
id    type
repo1   fs
repo2   s3
```

