---
layout: default
title: 删除快照
parent: 快照API
nav_order: 7
---

## 删除快照
**引入1.0**
{: .label .label-purple }

从存储库中删除快照。

*要了解有关快照的更多信息，请参阅[快照]({{site.url}}{{site.baseurl}}/opensearch/snapshots/index)。

*要查看您的存储库列表，请参阅[猫存储库]({{site.url}}{{site.baseurl}}/api-reference/cat/cat-repositories)。

*要查看您的快照列表，请参阅[猫快照]({{site.url}}{{site.baseurl}}/api-reference/cat/cat-snapshots)。

## 路径参数

范围| 数据类型| 描述
:--- | :--- | :---
存储库| 细绳| 包含快照的转换。|
快照| 细绳| 快照要删除。|

#### 示例请求

以下请求删除了一个名为的快照`my-first-snapshot` 来自`my-opensearch-repo` 存储库：

```json
DELETE _snapshot/my-opensearch-repo/my-first-snapshot
```
{% include copy-curl.html %}

#### 示例响应

成功后，响应返回以下JSON对象：

```json
{
  "acknowledged": true
}
```

要验证快照已删除，请使用[获取快照]({{site.url}}{{site.baseurl}}/api-reference/snapshots/get-snapshot) API，将快照名称传递给`snapshot` 路径参数。
{:.note}

