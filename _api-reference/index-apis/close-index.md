---
layout: default
title: 关闭索引
parent: 索引API
nav_order: 20
redirect_from:
  - /opensearch/rest-api/index-apis/close-index/
---

# 关闭索引
**引入1.0**
{: .label .label-purple }

关闭索引API操作关闭索引。关闭索引后，您将无法向其添加数据或搜索索引中的任何数据。

#### 例子

```json
POST /sample-index/_close
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
POST /<index-name>/_close
```

## URL参数

所有参数都是可选的。

范围| 类型| 描述
:--- | :--- | :---
＆lt;索引-名称＆gt;| 细绳| 关闭的索引。可以是逗号-多个索引名称的分开列表。使用`_all` 或 *关闭所有索引。
允许_no_indices| 布尔| 是否忽略不符合任何索引的通配符。默认是正确的。
Expand_WildCard| 细绳| 将通配符表达式扩展到不同的索引。将多个值与逗号相结合。可用的值全部（匹配所有索引），打开（匹配打开索引），封闭（匹配封闭索引），隐藏（匹配隐藏索引）和无（不接受通配符表达式）。默认值打开。
ignore_unavailable| 布尔| 如果为true，则OpenSearch不会搜索缺失或封闭的索引。默认值为false。
wait_for_active_shards| 细绳| 指定在OpenSearch处理请求之前必须可用的活动碎片数。默认值为1（仅是主要碎片）。设置为全部或正整数。大于1的值需要复制品。例如，如果指定一个值为3的值，则该索引必须在两个其他节点上分布两个副本才能成功。
cluster_manager_timeout| 时间| 等待连接到群集管理器节点多长时间。默认为`30s`。
暂停| 时间| 等待群集的响应多长时间。默认为`30s`。


## 回复
```json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "indices": {
    "sample-index1": {
      "closed": true
    }
  }
}
```

