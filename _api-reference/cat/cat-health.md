---
layout: default
title: CAT 健康
parent: CAT API

nav_order: 20
has_children: false
redirect_from:
- /opensearch/rest-api/cat/cat-health/
---

# 猫健康
**引入1.0**
{: .label .label-purple }

CAT健康操作列出了集群的状态，群集已经增加了多长时间，节点的数量以及其他有用的信息，可帮助您分析集群的健康状况。

## 例子

```json
GET _cat/health?v
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET _cat/health?v
```
{% include copy-curl.html %}

## URL参数

所有CAT健康URL参数都是可选的。

范围| 类型| 描述
:--- | :--- | :---
时间| 时间| 指定时间的单位。例如，`5d` 或者`7h`。有关更多信息，请参阅[支持单位]({{site.url}}{{site.baseurl}}/opensearch/units/)。
TS| 布尔| 如果为true，请返回HH：MM：SS和UNIX Epoch时间戳。默认是正确的。

## 回复

```json
GET _cat/health?v&time=5d

epoch | timestamp | cluster | status | node.total | node.data | shards | pri | relo | init | unassign | pending_tasks | max_task_wait_time | active_shards_percent
1624248112 | 04:01:52 | odfe-cluster | green | 2 | 2 | 16 | 8 | 0 | 0 | 0 | 0 | - | 100.0%
```

