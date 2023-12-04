---
layout: default
title: CAT 插件
parent: CAT API

nav_order: 50
has_children: false
redirect_from:
- /opensearch/rest-api/cat/cat-plugins/
---

# 猫插件
**引入1.0**
{: .label .label-purple }

CAT插件操作列出了已安装插件的名称，组件和版本。

## 例子

```
GET _cat/plugins?v
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET _cat/plugins
```

## URL参数

所有CAT插件URL参数都是可选的。

除了[常见的URL参数]({{site.url}}{{site.baseurl}}/api-reference/cat/index)，您可以指定以下参数：

范围| 类型| 描述
:--- | :--- | :---
当地的| 布尔| 是否仅从本地节点返回信息，而不是从cluster_manager节点返回。默认值为false。
cluster_manager_timeout| 时间| 等待连接到cluster_manager节点的时间。默认值为30秒。

## 回复

```json
name       component                       version
odfe-node2 opendistro-alerting             1.13.1.0
odfe-node2 opendistro-anomaly-detection    1.13.0.0
odfe-node2 opendistro-asynchronous-search  1.13.0.1
odfe-node2 opendistro-index-management     1.13.2.0
odfe-node2 opendistro-job-scheduler        1.13.0.0
odfe-node2 opendistro-knn                  1.13.0.0
odfe-node2 opendistro-performance-analyzer 1.13.0.0
odfe-node2 opendistro-reports-scheduler    1.13.0.0
odfe-node2 opendistro-sql                  1.13.2.0
odfe-node2 opendistro_security             1.13.1.0
odfe-node1 opendistro-alerting             1.13.1.0
odfe-node1 opendistro-anomaly-detection    1.13.0.0
odfe-node1 opendistro-asynchronous-search  1.13.0.1
odfe-node1 opendistro-index-management     1.13.2.0
odfe-node1 opendistro-job-scheduler        1.13.0.0
odfe-node1 opendistro-knn                  1.13.0.0
odfe-node1 opendistro-performance-analyzer 1.13.0.0
odfe-node1 opendistro-reports-scheduler    1.13.0.0
odfe-node1 opendistro-sql                  1.13.2.0
odfe-node1 opendistro_security             1.13.1.0
```

