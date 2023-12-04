---
layout: default
title: CAT 恢复
parent: CAT API

nav_order: 50
has_children: false
redirect_from:
- /opensearch/rest-api/cat/cat-recovery/
---

# 猫恢复
**引入1.0**
{: .label .label-purple }

CAT恢复操作列出了所有已完成的索引和碎片恢复。

## 例子

```
GET _cat/recovery?v
```
{% include copy-curl.html %}

要仅查看特定索引的恢复，请在查询之后添加索引名称。

```
GET _cat/recovery/<index>?v
```
{% include copy-curl.html %}

如果您想获得多个索引的信息，请将索引与逗号分开：

```json
GET _cat/recovery/index1,index2,index3
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET _cat/recovery
```

## URL参数

所有CAT恢复URL参数都是可选的。

除了[常见的URL参数]({{site.url}}{{site.baseurl}}/api-reference/cat/index)，您可以指定以下参数：

范围| 类型| 描述
:--- | :--- | :---
Active_only| 布尔| 是否仅包括正在进行的碎片回收。默认值为false。
字节| 字节大小| 指定字节大小的单元。例如，`7kb` 或者`6gb`。有关更多信息，请参阅[支持单位]({{site.url}}{{site.baseurl}}/opensearch/units/)。
详细的| 布尔| 是否包括有关碎片回收的详细信息。默认值为false。
时间| 时间| 指定时间的单位。例如，`5d` 或者`7h`。有关更多信息，请参阅[支持单位]({{site.url}}{{site.baseurl}}/opensearch/units/)。

## 回复

```json
index | shard | time | type | stage | source_host | source_node | target_host | target_node | repository | snapshot | files | files_recovered | files_percent | files_total | bytes | bytes_recovered | bytes_percent | bytes_total | translog_ops | translog_ops_recovered | translog_ops_percent
movies | 0 | 117ms | empty_store | done | n/a | n/a | 172.18.0.4 | odfe-node1 | n/a | n/a | 0 | 0 | 0.0% | 0 | 0 | 0 | 0.0% | 0 | 0 | 0 | 100.0%
movies | 0 | 382ms | peer | done | 172.18.0.4 | odfe-node1 | 172.18.0.3 | odfe-node2 | n/a | n/a | 1 | 1 |  100.0% | 1 | 208 | 208 | 100.0% | 208 | 1 | 1 | 100.0%
```

