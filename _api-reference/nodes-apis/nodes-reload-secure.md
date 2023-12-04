---
layout: default
title: 节点重新加载安全设置
parent: 节点API
nav_order: 50
---

# 节点重新加载安全设置
**引入1.0**
{: .label .label-purple }

节点重新加载安全设置端点使您可以在节点上更改安全设置，并在不重新启动节点的情况下重新加载安全设置。

## 路径和HTTP方法

```
POST _nodes/reload_secure_settings
POST _nodes/<nodeId>/reload_secure_settings
```

## 路径参数

您可以在请求中包含以下可选路径参数。

范围| 类型| 描述
:--- | :--- | :---
nodeid| 细绳| 逗号-用于过滤结果的节点的分离列表。支持[节点过滤器]({{site.url}}{{site.baseurl}}/api-reference/nodes-apis/index/#node-filters)。默认为`_all`。

## 请求字段

该请求可能包括一个可选对象，其中包含OpenSearch密钥库的密码。

```json
{
  "secure_settings_password": "keystore_password"
}
```

#### 示例请求

以下是一个示例API请求：

```
POST _nodes/reload_secure_settings
```
{% include copy-curl.html %}

#### 示例响应

以下是一个示例响应：

```json
{
  "_nodes" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "cluster_name" : "opensearch-cluster",
  "nodes" : {
    "t7uqHu4SSuWObK3ElkCRfw" : {
      "name" : "opensearch-node1"
    }
  }
}
```

## 所需的权限

如果使用安全插件，请确保设置以下权限：`cluster:manage/nodes`

