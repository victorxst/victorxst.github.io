---
layout: default
title: CAT 待定任务
parent: CAT API

nav_order: 45
has_children: false
redirect_from:
- /opensearch/rest-api/cat/cat-pending-tasks/
---

# 猫待定任务
**引入1.0**
{: .label .label-purple }

猫待定任务操作列出了所有待处理任务的进度，包括任务优先级和队列时间。

## 例子

```
GET _cat/pending_tasks?v
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET _cat/pending_tasks
```

## URL参数

所有CAT节点URL参数都是可选的。

除了[常见的URL参数]({{site.url}}{{site.baseurl}}/api-reference/cat/index)，您可以指定以下参数：

范围| 类型| 描述
:--- | :--- | :---
当地的| 布尔| 是否仅从本地节点返回信息，而不是从cluster_manager节点返回。默认值为false。
cluster_manager_timeout| 时间| 等待连接到群集管理器节点的时间。默认值为30秒。
时间| 时间| 指定时间的单位。例如，`5d` 或者`7h`。有关更多信息，请参阅[支持单位]({{site.url}}{{site.baseurl}}/opensearch/units/)。

## 回复

```json
insertOrder | timeInQueue | priority | source
  1786      |    1.8s     |  URGENT  | shard-started
```

