---
layout: default
title: 集群管理器任务节流
nav_order: 10
has_children: false
---

# 集群管理器任务节流

对于许多集群状态更新，例如定义映射或创建索引，节点将任务提交给群集管理器。集群管理器维护这些任务的待处理任务队列，并将其运行-螺纹环境。当节点发送数以万计的资源时-强化任务，例如`put-mapping` 或快照任务，这些任务可以在队列中堆积并淹没集群管理器。这会影响集群管理器的性能，进而可能影响整个集群的可用性。

第一道防线是在呼叫者节点中实现机制，以避免群集管理器上的任务超载。但是，即使有了这些机制，集群管理器也需要一个构建的-在保护自己的方式中：集群管理器任务节流。

要打开群集管理器任务节流，您需要设置节流限制。群集管理器使用节流限制来确定是否拒绝任务。

集群管理器根据其类型拒绝任务。对于任何传入任务，群集管理器评估待处理任务队列中同一类型的任务总数。如果此数字超过此任务类型的阈值，则群集管理器拒绝传入的任务。拒绝任务不会影响其他类型的任务。例如，如果集群管理器拒绝`put-mapping` 任务，它仍然可以接受随后的`create-index` 任务。

当群集管理器拒绝任务时，节点会以指数向后进行重试，以将任务重新提交为群集管理器。如果在超时期内重新验证不成功，则OpenSearch返回集群超时错误。

## 设置节流限制

您可以通过在`cluster_manager.throttling.thresholds` 对象并更新[OpenSearch集群设置]({{site.url}}{{site.baseurl}}/api-reference/cluster-settings)。设置是动态的，因此您可以在不重新启动群集的情况下更改此功能的行为。

默认情况下，所有任务类型都禁用节流。
{: .note}

该请求具有以下格式：

```json
PUT _cluster/settings
{
  "persistent": {
    "cluster_manager.throttling.thresholds" : {
      "<task-type>" : {
          "value" : <threshold limit>
      }
    }
  }
}
```

下表描述了`cluster_manager.throttling.thresholds` 目的。

字段名称| 描述
:--- | :---
任务-类型| 任务类型。看[支持的任务类型](#supported-task-types) 对于有效值列表。
价值| 最大任务数量`task-type` 输入群集管理器的待处理任务队列。默认为`-1` （没有任务节流）。

## 支持的任务类型

支持以下任务类型：

- `create-index` 
- `update-settings` 
- `cluster-update-settings` 
- `auto-create` 
- `delete-index` 
- `delete-dangling-index` 
- `create-data-stream` 
- `remove-data-stream` 
- `rollover-index` 
- `index-aliases` 
- `put-mapping` 
- `create-index-template` 
- `remove-index-template` 
- `create-component-template` 
- `remove-component-template` 
- `create-index-template-v2` 
- `remove-index-template-v2` 
- `put-pipeline` 
- `delete-pipeline` 
- `create-persistent-task` 
- `finish-persistent-task` 
- `remove-persistent-task` 
- `update-task-state` 
- `put-script` 
- `delete-script` 
- `put-repository` 
- `delete-repository` 
- `create-snapshot` 
- `delete-snapshot` 
- `update-snapshot-state` 
- `restore-snapshot` 
- `cluster-reroute-api`

#### 示例请求

以下请求设置了节流门槛`put-mapping` 任务类型至100：

```json
PUT _cluster/settings
{
  "persistent": {
    "cluster_manager.throttling.thresholds": {
      "put-mapping": {
        "value": 100
      }
    }
  }
}
```

将阈值设置为`-1` 为任务类型禁用节流。
{: .note}




