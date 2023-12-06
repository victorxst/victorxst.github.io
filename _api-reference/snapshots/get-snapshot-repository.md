---
layout: default
title: 获取快照存储库
parent: 快照API
nav_order: 2
---

# 获取快照存储库。
**引入1.0**
{: .label .label-purple }

检索有关快照存储库的信息。

要了解有关存储库的更多信息，请参阅[注册存储库]({{site.url}}{{site.baseurl}}/opensearch/snapshots/snapshot-restore#register-repository)。

您还可以在快照创建期间和之后获取有关快照的详细信息。看[获取快照状态]({{site.url}}{{site.baseurl}}/api-reference/snapshots/get-snapshot-status/)。
{:.note}

## 路径参数

| 范围| 数据类型| 描述|
| :--- | :--- | :--- |
| 存储库| 细绳| 逗号-分开的快照存储库名称列表要检索。通配符`*`支持表达式，包括将通配符与排除模式结合起来`-`。|

## 查询参数

| 范围| 数据类型| 描述| 
:--- | :--- | :---
| 当地的| 布尔| 是否从本地节点获取信息。可选，默认为`false`。|
| cluster_manager_timeout| 时间| 等待与主节点连接的时间。可选，默认为30秒。|

#### 示例请求

以下请求检索信息`my-opensearch-repo` 存储库：

````json
GET /_snapshot/my-opensearch-repo
````
{% include copy-curl.html %}

#### 示例响应

成功后，响应返回存储信息。这个样本是针对的`s3` 存储库类型。

````json
{
  "my-opensearch-repo" : {
    "type" : "s3",
    "settings" : {
      "bucket" : "my-open-search-bucket",
      "base_path" : "snapshots"
    }
  }
}
````

## 响应字段

| 场地| 数据类型| 描述|
| :--- | :--- | :--- | 
| 类型| 细绳| 桶类型：`fs` （文件系统）或`s3` （S3桶）|
| 桶| 细绳| S3存储桶名。|
| base_path| 细绳| 存储快照的存储夹中的文件夹。

