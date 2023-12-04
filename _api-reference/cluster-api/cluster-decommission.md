---
layout: default
title: 集群退役 
nav_order: 30
parent: 群集API
has_children: false
redirect_from: 
  - /api-reference/cluster-decommission/
  - /opensearch/rest-api/cluster-decommission/
---

# 集群退役
**引入1.0**
{: .label .label-purple }

集群退役操作增加了基于意识的支持退役。这极大地有益于多人-区域部署，意识属性，例如`zones`，可以帮助以受控方式将新升级应用于集群。这在中断期间特别有用，在这种情况下，您可以退役不健康的区域，以防止复制请求停止，并防止您的请求积压变得太大。

有关分配意识的更多信息，请参阅[碎片分配意识]({{site.url}}{{site.baseurl}}//opensearch/cluster/#shard-allocation-awareness)。


## HTTP和路径方法

```
PUT  /_cluster/decommission/awareness/{awareness_attribute_name}/{awareness_attribute_value}
GET  /_cluster/decommission/awareness/{awareness_attribute_name}/_status
DELETE /_cluster/decommission/awareness
```

## URL参数

范围| 类型| 描述
:--- | :--- | :---
vareness_attribute_name| 细绳| 意识属性的名称，通常`zone`。
vareness_attribute_value| 细绳| 意识属性的价值。例如，如果您分配了两个不同区域的碎片，则可以给每个区域一个值的值`zone-a` 或者`zoneb`。群集退役操作该方法中列出的区域。


## 示例：退役和推荐区域

您可以使用以下示例请求将其退役和推荐一个区域：

#### 要求

以下示例请求退役`zone-a`：

```json
PUT /_cluster/decommission/awareness/<zone>/<zone-a>
```
{% include copy-curl.html %}

如果您想推荐退役区，可以使用`DELETE` 方法：

```json
DELETE /_cluster/decommission/awareness
```
{% include copy-curl.html %}

#### 回复


```json
{
      "acknowledged": true
}
```

## 示例：获得区域退役状态

以下示例请求返回所有区域的退役状态。

#### 要求

```json
GET /_cluster/decommission/awareness/zone/_status
```
{% include copy-curl.html %}

#### 回复

```json
{
     "zone-1": "INIT | DRAINING | IN_PROGRESS | SUCCESSFUL | FAILED"
}
```


## 下一步

- 有关区域意识和体重的更多信息，请参阅[集群意识]({{site.url}}{{site.baseurl}}/api-reference/cluster-awareness/)。
- 有关分配意识的更多信息，请参阅[集群形成]({{site.url}}{{site.baseurl}}/opensearch/cluster/#advanced-step-6-configure-shard-allocation-awareness-or-forced-awareness)。

