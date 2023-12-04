---
layout: default
title: CAT 线程池
parent: CAT API
nav_order: 75
has_children: false
redirect_from:
- /opensearch/rest-api/cat/cat-thread-pool/
---

# 猫线池
**引入1.0**
{: .label .label-purple }

CAT线程池操作列出了每个节点上不同线程池的活动，排队和拒绝的线程。

## 例子

```
GET _cat/thread_pool?v
```
{% include copy-curl.html %}

如果要获取多个线程池的信息，请将线程池名称与逗号分开：

```
GET _cat/thread_pool/thread_pool_name_1,thread_pool_name_2,thread_pool_name_3
```
{% include copy-curl.html %}

如果要将信息限制到特定的线程池中，请在查询之后添加线程池名称：

```
GET _cat/thread_pool/<thread_pool_name>?v
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET _cat/thread_pool
```

## URL参数

所有CAT线程池URL参数都是可选的。

除了[常见的URL参数]({{site.url}}{{site.baseurl}}/api-reference/cat/index)，您可以指定以下参数：

范围| 类型| 描述
:--- | :--- | :---
当地的| 布尔| 是否仅从本地节点返回信息，而不是从cluster_manager节点返回。默认值为false。
cluster_manager_timeout| 时间| 等待连接到cluster_manager节点的时间。默认值为30秒。


## 回复

```json
node_name  name                      active queue rejected
odfe-node2 ad-batch-task-threadpool    0     0        0
odfe-node2 ad-threadpool               0     0        0
odfe-node2 analyze                     0     0        0s
```

