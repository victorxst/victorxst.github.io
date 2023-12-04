---
layout: default
title: 悬挂索引
parent: index-apis
nav_order: 30
---

# 悬挂索引API
**引入1.0**
{: .label .label-purple }

节点加入群集后，如果在节点的本地目录中存在任何碎片，则会发生悬挂索引，这些碎片在集群中尚未存在。可以列出，删除或进口悬挂索引。

## 路径和HTTP方法

清单悬挂索引：

```
GET /_dangling
```

导入一个悬空指数：

```
POST /_dangling/<index-uuid>
```

删除一个悬挂索引：

```
DELETE /_dangling/<index-uuid>
```

## 路径参数

需要路径参数。

路径参数| 描述
:--- | :---
指数-UUID| 索引的UUID。

## 查询参数

查询参数是可选的。

查询参数| 数据类型| 描述
:--- | :--- | :---
accept_data_loss| 布尔| 必须设置为`true` 为`import` 或者`delete` 因为Opensearch不知道悬挂索引数据的来源。
暂停| 时间单元| 等待响应的时间。如果在定义的时间段内未收到任何响应，则会返回错误。默认为`30` 秒。
cluster_manager_timeout| 时间单元| 等待与集群管理器连接的时间。如果在定义的时间段内未收到任何响应，则会返回错误。默认为`30` 秒。

## 例子

以下是示例请求和示例响应。

#### 样本列表

````bash
GET /_dangling
````
{% include copy-curl.html %}

#### 样本导入

````bash
POST /_dangling/msdjernajxAT23RT-BupMB?accept_data_loss=true
````
{% include copy-curl.html %}
 
#### 样品删除

````bash
DELETE /_dangling/msdjernajxAT23RT-BupMB?accept_data_loss=true
````

#### 示例响应主体

````json
{
    "_nodes": {
        "total": 1,
        "successful": 1,
        "failed": 0
    },
    "cluster_name": "opensearch-cluster",
    "dangling_indices": [msdjernajxAT23RT-BupMB]
}
````
