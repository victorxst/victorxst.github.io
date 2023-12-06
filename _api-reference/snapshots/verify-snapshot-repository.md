---
layout: default
title: 验证快照存储库
parent: 快照API

nav_order: 4
---

# 验证快照存储库
**引入1.0**
{: .label .label-purple }

验证快照存储库是否有效。验证集群中每个节点上的存储库。

如果验证成功，则验证快照存储库API返回连接到快照存储库的节点的列表。如果验证失败，API将返回错误。

如果使用安全插件，则必须`manage cluster` 特权。
{: .note}

## 路径参数

路径参数是可选的。

| 范围| 数据类型| 描述| 
:--- | :--- | :---
| 存储库| 细绳| 验证存储库的名称。|

## 查询参数

| 范围| 数据类型| 描述| 
:--- | :--- | :---
| cluster_manager_timeout| 时间| 等待与主节点连接的时间。可选，默认为`30s`。|
| 暂停| 时间| 等待回应的时间。如果在超时值之前未收到响应，则请求失败并返回错误。默认为`30s`。|

#### 示例请求

以下请求验证了我的-OpenSearch-仓库功能很强：

````json
POST /_snapshot/my-opensearch-repo/_verify?timeout=0s&cluster_manager_timeout=50s
````

#### 示例响应

接下来的示例对应于上面的请求[示例请求](#example-request) 部分。

这`POST /_snapshot/my-opensearch-repo/_verify?timeout=0s&cluster_manager_timeout=50s` 请求返回以下字段：

````json
{
  "nodes" : {
    "by1kztwTRoeCyg4iGU5Y8A" : {
      "name" : "opensearch-node1"
    }
  }
}
````

在前面的样本中，一个节点连接到快照存储库。如果有更多连接，您会在响应中看到它们。例子：

````json
{
  "nodes" : {
    "lcfL6jv2jo6sMEtp4idMvg" : {
      "name" : "node-1"
    },
    "rEPtFT/B+cuuOHnQn0jy4s" : {
      "name" : "node-2"
  }
}
````

## 响应字段

| 场地| 数据类型| 描述| 
:--- | :--- | :---
| 节点| 目的| 连接到快照存储库的节点的列表（不是数组）。每个节点本身是一个属性，其中节点ID是密钥，并且名称具有ID（对象）和名称（字符串）。

