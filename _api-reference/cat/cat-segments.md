---
layout: default
title: CAT 段
parent: CAT API

nav_order: 55
has_children: false
redirect_from:
- /opensearch/rest-api/cat/cat-segments/
---

# 猫段
**引入1.0**
{: .label .label-purple }

猫段操作列出了Lucene段-每个索引的级别信息。

## 例子

```
GET _cat/segments?v
```
{% include copy-curl.html %}

要仅查看有关特定索引段的信息，请在查询后添加索引名称。

```
GET _cat/segments/<index>?v
```
{% include copy-curl.html %}

如果您想获得多个索引的信息，请将索引与逗号分开：

```
GET _cat/segments/index1,index2,index3
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET _cat/segments
```

## URL参数

所有CAT段URL参数都是可选的。

除了[常见的URL参数]({{site.url}}{{site.baseurl}}/api-reference/cat/index)，您可以指定以下参数：

范围| 类型| 描述
:--- | :--- | :---
字节| 字节大小| 指定字节大小的单元。例如，`7kb` 或者`6gb`。有关更多信息，请参阅[支持单位]({{site.url}}{{site.baseurl}}/opensearch/units/)..
cluster_manager_timeout| 时间| 等待连接到群集管理器节点的时间。默认值为30秒。


## 回复

```json
index | shard | prirep | ip | segment | generation | docs.count | docs.deleted | size | size.memory | committed | searchable | version | compound
movies | 0 | p | 172.18.0.4 | _0 | 0 | 1 | 0 | 3.5kb | 1364 | true | true | 8.7.0 | true
movies | 0 | r | 172.18.0.3 | _0 | 0 | 1 | 0 | 3.5kb | 1364 | true | true | 8.7.0 | true
```

