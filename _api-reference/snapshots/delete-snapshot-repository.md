---
layout: default
title: 删除快照存储库
parent: 快照API
nav_order: 3
---

# 删除快照存储库配置
**引入1.0**
{: .label .label-purple }

 删除快照存储库配置。
 
 OpenSearch中的存储库只是一种配置，该配置将存储库名称映射到类型（文件系统或S3存储库）以及其他信息，具体取决于类型。配置由文件系统位置或S3存储桶进行支持。调用API时，物理文件系统或S3存储桶本身不会删除。仅删除配置。

 要了解有关存储库的更多信息，请参阅[注册或更新快照存储库]({{site.url}}{{site.baseurl}}/api-reference/snapshots/create-repository)。

## 路径参数

范围| 数据类型| 描述
:--- | :--- | :---
存储库| 细绳| 删除存储库。|

#### 示例请求

以下请求删除了`my-opensearch-repo` 存储库：

````json
DELETE _snapshot/my-opensearch-repo
````
{% include copy-curl.html %}

#### 示例响应

成功后，响应返回以下JSON对象：

````json
{
  "acknowledged" : true
}
````

要验证存储库已删除，请使用[获取快照存储库]({{site.url}}{{site.baseurl}}/api-reference/snapshots/get-snapshot-repository) API，将存储库名称作为`repository` 路径参数。
{: .note}

