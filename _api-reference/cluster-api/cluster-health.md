---
layout: default
title: 集群健康
nav_order: 40
parent: 群集API
has_children: false
redirect_from: 
 - /api-reference/cluster-health/
 - /opensearch/rest-api/cluster-health/
---

# 簇健康
**引入1.0**
{: .label .label-purple }

最基本的集群健康请求返回了群集健康状况的简单状态。OpenSearch表达三种颜色的集群健康：绿色，黄色和红色。绿色状态意味着将所有主要碎片及其复制品分配给节点。黄色状态意味着所有主要碎片都分配给节点，但有些复制品不是。红色状态意味着至少一个主碎片未分配给任何节点。

要获得特定索引的状态，请提供索引名称。

## 例子

该请求等待50秒才能达到黄色状态或更高的状态：

```
GET _cluster/health?wait_for_status=yellow&timeout=50s
```
{% include copy-curl.html %}

如果群集健康在经过50秒之前变成黄色或绿色，则立即返回回应。否则，它会在超出超时后立即返回响应。

## 路径和HTTP方法

```
GET _cluster/health
GET _cluster/health/<index>
```

## 查询参数

下表列出了可用查询参数。所有查询参数都是可选的。

范围| 类型| 描述
:--- | :--- | :---
Expand_WildCard| 枚举| 将通配符表达式扩展到混凝土指数。将多个值与逗号相结合。支持的值是`all`，`open`，`closed`，`hidden`， 和`none`。默认为`open`。
等级| 枚举| 返回健康信息的细节级别。支持的值是`cluster`，`indices`，`shards`， 和`awareness_attributes`。默认为`cluster`。
vareness_attribute| 细绳| 意识属性的名称，为此返回集群健康（例如，`zone`）。仅适用于`level` 被设定为`awareness_attributes`。
当地的| 布尔| 是否仅从本地节点返回信息，而不是从群集管理器节点返回。默认值为false。
cluster_manager_timeout| 时间| 等待连接到群集管理器节点的时间。默认值为30秒。
暂停| 时间| 等待响应的时间。如果超时到期，请求失败。默认值为30秒。
wait_for_active_shards| 细绳| 等到指定数量的碎片处于活动状态，然后再返回响应。`all` 对于所有碎片。默认为`0`。
wait_for_nodes| 细绳| 等待n个节点数。使用`12` 对于确切的匹配，`>12` 和`<12` 对于范围。
wait_for_events| 枚举| 等待直到处理具有给定优先级的所有当前排队事件。支持的值是`immediate`，`urgent`，`high`，`normal`，`low`， 和`languid`。
wait_for_no_relocating_shards| 布尔| 是否要等到集群中没有重新定位碎片。默认值为false。
wait_for_no_initializing_shards| 布尔| 是否要等到集群中没有初始化的碎片。默认值为false。
wait_for_status| 枚举| 等到簇健康达到指定状态或更好的状态。支持的值是`green`，`yellow`， 和`red`。
权重| JSON对象| 将权重分配给PUT请求主体内的属性。重量可以以任何口粮设置，例如2：3：5。在2：3：5的比例中，有三个区域，每100个请求发送到集群，每个区域将以随机顺序接收20、30或50个搜索请求。分配重量`0`，该区域未收到任何搜索流量。

#### 示例请求

以下示例请求检索集群中所有索引的集群健康：

```json
GET _cluster/health
```
{% include copy-curl.html %}

#### 示例响应

响应包含群集健康信息：

```json
{
  "cluster_name" : "opensearch-cluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 2,
  "number_of_data_nodes" : 2,
  "discovered_master" : true,
  "active_primary_shards" : 6,
  "active_shards" : 12,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

## 响应字段

下表列出了所有响应字段。

|场地|数据类型|描述|
|:---	|:---	|：---|
|cluster_name| 细绳| 集群的名称。|
|地位| 细绳| 集群健康状况，代表集群中的分片分配状态。或许`green`，`yellow`， 或者`red`。|
|number_of_nodes| 整数| 集群中的节点数量。|
|number_of_data_nodes| 整数| 集群中的数据节点数量。|
|discusiged_cluster_manager| 布尔| 指定是否发现群集管理器。|
|Active_primary_shards| 整数|  主动主要碎片的数量。|
|Active_shards| 整数| 主动碎片的总数，包括主要和复制碎片。|
|rococating_shards| 整数| 重新定位碎片的数量。|
|initialized_shards| 整数| 初始化碎片的数量。|
|unsigned_shards| 整数| 未分配的碎片数量。|
|delayed_unassigned_shards| 整数| 延迟未分配的碎片的数量。|
|number_of_pending_tasks| 整数| 集群中的待处理任务数量。|
|number_of_in_flight_fetch| 整数| 未完成的获取数量。|
|task_max_waiting_in_queue_millis| 整数| 所有等待执行的任务的最大等待时间，以毫秒为单位。|
|Active_shards_percent_as_number| 双倍的| 集群中有源碎片的百分比。|
|vareness_attributes| 目的| 包含每个意识属性的集群健康信息。|

## 通过意识属性返回集群健康

要通过意识属性（例如区域或机架）检查群集健康，请指定`awareness_attributes` 在里面`level` 查询参数：

```json
GET _cluster/health?level=awareness_attributes
```
{% include copy-curl.html %}

响应包含通过意识属性分区的集群健康指标：

```json
{
  "cluster_name": "runTask",
  "status": "green",
  "timed_out": false,
  "number_of_nodes": 3,
  "number_of_data_nodes": 3,
  "discovered_master": true,
  "discovered_cluster_manager": true,
  "active_primary_shards": 0,
  "active_shards": 0,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 0,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 100,
  "awareness_attributes": {
    "zone": {
      "zone-3": {
        "active_shards": 0,
        "initializing_shards": 0,
        "relocating_shards": 0,
        "unassigned_shards": 0,
        "data_nodes": 1,
        "weight": 1
      },
      "zone-1": {
        "active_shards": 0,
        "initializing_shards": 0,
        "relocating_shards": 0,
        "unassigned_shards": 0,
        "data_nodes": 1,
        "weight": 1
      },
      "zone-2": {
        "active_shards": 0,
        "initializing_shards": 0,
        "relocating_shards": 0,
        "unassigned_shards": 0,
        "data_nodes": 1,
        "weight": 1
      }
    },
    "rack": {
      "rack-3": {
        "active_shards": 0,
        "initializing_shards": 0,
        "relocating_shards": 0,
        "unassigned_shards": 0,
        "data_nodes": 1,
        "weight": 1
      },
      "rack-1": {
        "active_shards": 0,
        "initializing_shards": 0,
        "relocating_shards": 0,
        "unassigned_shards": 0,
        "data_nodes": 1,
        "weight": 1
      },
      "rack-2": {
        "active_shards": 0,
        "initializing_shards": 0,
        "relocating_shards": 0,
        "unassigned_shards": 0,
        "data_nodes": 1,
        "weight": 1
      }
    }
  }
}
```

如果您对特定的意识属性感兴趣，则可以将意识属性的名称包括为查询参数：

```json
GET _cluster/health?level=awareness_attributes&awareness_attribute=zone
```
{% include copy-curl.html %}

为了响应上述请求，OpenSearch返回集群健康信息仅用于`zone` 意识属性。

仅当您[启用副本计数执行]({{site.url}}{{site.baseurl}}/opensearch/cluster#replica-count-enforcement) 和[配置强制意识]({{site.url}}{{site.baseurl}}/opensearch/cluster#forced-awareness) 对于群集开始或群集启动之后但在任何索引请求之前，要么在群集开始之前或之后的意识属性。如果在集群收到索引请求后启用复制执行，则未分配的碎片信息可能不准确。如果您不配置副本计数执行和强迫意识，则`unassigned_shards` 字段将包含-1。
{: .warning}

## 所需的权限

如果使用安全插件，请确保拥有适当的权限：
`cluster:monitor/health`。

