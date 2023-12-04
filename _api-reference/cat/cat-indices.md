---
layout: default
title: CAT 指数操作
parent: CAT API
nav_order: 25
has_children: false
redirect_from:
- /opensearch/rest-api/cat/cat-indices/
---

# 猫指数
**引入1.0**
{: .label .label-purple }

CAT索引操作列出了与索引有关的信息，即，他们使用的磁盘空间数量，拥有多少碎片，其健康状况等等。

## 例子

```
GET _cat/indices?v
```
{% include copy-curl.html %}

要将信息限制为特定索引，请在查询之后添加索引名称。

```
GET _cat/indices/<index>?v
```
{% include copy-curl.html %}

如果您想获得多个索引的信息，请将索引与逗号分开：

```json
GET _cat/indices/index1,index2,index3
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET _cat/indices/<index>
GET _cat/indices
```

## URL参数

所有CAT索引URL参数都是可选的。

除了[常见的URL参数]({{site.url}}{{site.baseurl}}/api-reference/cat/index)，您可以指定以下参数：

范围| 类型| 描述
:--- | :--- | :---
字节| 字节大小| 指定字节大小的单元。例如，`7kb` 或者`6gb`。有关更多信息，请参阅[支持单位]({{site.url}}{{site.baseurl}}/opensearch/units/)。
健康| 细绳| 根据其健康状况限制索引。支持的值是`green`，`yellow`， 和`red`。
包括_unloaded_segments| 布尔| 是否包括未加载到内存中的段中的信息。默认值为false。
cluster_manager_timeout| 时间| 等待连接到群集管理器节点的时间。默认值为30秒。
pri| 布尔| 是否仅从主要碎片中返回信息。默认值为false。
时间| 时间| 指定时间的单位。例如，`5d` 或者`7h`。有关更多信息，请参阅[支持单位]({{site.url}}{{site.baseurl}}/opensearch/units/)。
Expand_WildCard| 枚举| 将通配符表达式扩展到混凝土指数。将多个值与逗号相结合。支持的值是`all`，`open`，`closed`，`hidden`， 和`none`。默认为`open`。


## 回复

```json
health | status | index | uuid | pri | rep | docs.count | docs.deleted | store.size | pri.store.size
green  | open | movies | UZbpfERBQ1-3GSH2bnM3sg | 1 | 1 | 1 | 0 | 7.7kb | 3.8kb
```

