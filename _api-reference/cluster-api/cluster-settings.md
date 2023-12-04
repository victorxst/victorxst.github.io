---
layout: default
title: 集群设置
nav_order: 50
parent: 群集API
redirect_from:
  - /api-reference/cluster-settings/
  - /opensearch/rest-api/cluster-settings/
---

# 集群设置
**引入1.0**
{: .label .label-purple }

群集设置操作使您可以检查当前设置中的群集，查看默认设置并更改设置。当您使用API更新设置时，OpenSearch将其应用于集群中的所有节点。

## 路径和HTTP方法

```
GET _cluster/settings
PUT _cluster/settings
```

## 路径参数

所有群集设置参数都是可选的。

范围| 数据类型| 描述
:--- | :--- | :---
flat_settings| 布尔| 是否以平面形式返回设置，这可以提高可读性，尤其是对于重嵌套的设置。例如，平坦的形式`"cluster": { "max_shards_per_node": 500 }` 是`"cluster.max_shards_per_node": "500"`。
包括_defaults（仅获取）| 布尔| 是否将默认设置作为响应的一部分。此参数可用于识别要更新的设置的名称和当前值。
cluster_manager_timeout| 时间单元| 等待群集管理器节点响应的时间。默认为`30 seconds`。
超时（仅放置）| 时间单元| 等待集群响应的时间。默认为`30 seconds`。


#### 示例请求

```json
GET _cluster/settings?include_defaults=true
```
{% include copy-curl.html %}

#### 示例响应

```json
PUT _cluster/settings
{
   "persistent":{
      "action.auto_create_index": false
   }
}
```

## 请求字段

GET操作没有请求主体选项。所有群集设置字段参数都是可选的。

并非所有群集设置都可以使用群集设置API更新。您将收到错误消息`"setting [cluster.some.setting], not dynamically updateable"` 尝试通过API配置这些设置时。
{: .note }

有关所有集群设置的列表，请参见[配置OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)。

#### 示例请求

为了进行POT操作，请求主体必须包含`transient` 或者`persistent`，以及您要更新的设置：

```json
PUT _cluster/settings
{
   "persistent":{
      "cluster.max_shards_per_node": 500
   }
}
```
{% include copy-curl.html %}

有关瞬态设置，持久设置和优先级的更多信息，请参见[OpenSearch配置]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/)。

#### 示例响应

```json
{
   "acknowledged":true,
   "persistent":{
      "cluster":{
         "max_shards_per_node":"500"
      }
   },
   "transient":{}
}
```

