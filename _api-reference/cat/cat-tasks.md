---
layout: default
title: CAT 任务
parent: CAT API

nav_order: 70
has_children: false
redirect_from:
- /opensearch/rest-api/cat/cat-tasks/
---

# 猫任务
**引入1.0**
{: .label .label-purple }

CAT任务操作列出了当前在群集上运行的所有任务的进度。

## 例子

```
GET _cat/tasks?v
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET _cat/tasks
```

## URL参数

所有CAT任务URL参数都是可选的。

除了[常见的URL参数]({{site.url}}{{site.baseurl}}/api-reference/cat/index)，您可以指定以下参数：

范围| 类型| 描述
:--- | :--- | :---
节点| 列表| 逗号-分开的节点ID列表或名称以限制返回的信息。使用`_local` 要返回您要连接的节点的信息，请指定节点名称以从特定节点获取信息，或将参数保留为空以从所有节点中获取信息。
详细的| 布尔| 返回详细的任务信息。（默认：false）
parent_task_id| 细绳| 使用指定的父任务ID（node_id：task_number）返回任务。保持空或设置为-1返回全部。
时间| 时间| 指定时间的单位。例如，`5d` 或者`7h`。有关更多信息，请参阅[支持单位]({{site.url}}{{site.baseurl}}/opensearch/units/)。


## 回复

```json
action | task_id | parent_task_id | type | start_time | timestamp | running_time | ip | node
cluster:monitor/tasks/lists | 1vo54NuxSxOrbPEYdkSF0w:168062 | - | transport | 1624337809471 | 04:56:49 | 489.5ms | 172.18.0.4 | odfe-node1     
```

