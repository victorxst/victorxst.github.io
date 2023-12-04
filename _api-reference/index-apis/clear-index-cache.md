---
layout: default
title: 清除缓存
parent: 索引API
nav_order: 10
---

# 清除缓存
**引入1.0**
{: .label .label-purple }

清晰的缓存API操作清除了一个或多个索引的缓存。对于数据流，API清除了该流的背索索引的缓存。


如果使用安全插件，则必须`manage index` 特权。
{: .note}

## 路径参数

| 范围| 数据类型| 描述|
:--- | :--- | :---
| 目标| 细绳| 逗号-应用缓存清除的数据流，索引和索引别名的界定列表。通配符表达式（`*`支持）。要针对集群中的所有数据流和索引，请省略此参数或使用`_all` 或者`*`。选修的。|


## 查询参数

所有查询参数都是可选的。

| 范围| 数据类型| 描述|
:--- | :--- | :---
| 允许_no_indices| 布尔| 是忽略通配符，索引别名还是`_all` 目标 （`target` 路径参数）值不匹配任何索引。如果`false`，请求如果任何通配符表达式，索引别名或`_all` 目标值不匹配任何索引。如果请求目标包括其他开放索引，则此行为也适用。例如，目标是`fig*,app*` 如果索引开始`fig` 但是没有索引开头`app`。默认为`true`。|
| Expand_WildCard| 细绳| 确定通配符表达式可以扩展到的索引类型。接受由逗号分隔的多个值，例如`open,hidden`。有效值是：<br /> <br />`all` -- 扩展到打开，关闭和隐藏索引。<br /> <br />`open` -- 仅扩展到打开索引。<br /> <br />`closed` -- 仅扩展到封闭索引<br /> <br />`hidden` -- 扩展以包括隐藏索引。必须与`open`，`closed`， 或者`both`。<br /> <br />`none` -- 不接受扩展。<br /> <br />默认为`open`。|
| fieldData| 布尔| 如果`true`，清除字段缓存。使用`fields` 参数清除特定字段的缓存。默认为`true`。|
| 字段| 细绳| 与`fielddata` 范围。逗号-从缓存中清除的字段名称的界定列表。不支持对象或字段别名。默认为所有字段。|
| 文件| 布尔| 如果`true`，清除带有搜索角色的节点上的文件缓存的未使用条目。默认为`false`。|
| 指数| 细绳| 逗号-从缓存中清除的索引名称的划界列表。|
| ignore_unavailable| 布尔| 如果`true`，OpenSearch忽略缺失或封闭索引。默认为`false`。|
| 询问| 布尔| 如果`true`，清除查询缓存。默认为`true`。|
| 要求| 布尔| 如果`true`，清除请求缓存。默认为`true`。|

#### 示例请求

以下示例请求显示多个清晰的缓存API使用。

##### 清除特定的缓存

以下请求仅清除字段缓存：

```json
POST /my-index/_cache/clear?fielddata=true
```
{% include copy-curl.html %}

<hr />

以下请求仅清除查询缓存：

```json
POST /my-index/_cache/clear?query=true
```
{% include copy-curl.html %}

<hr />

以下请求仅清除请求缓存：

```json
POST /my-index/_cache/clear?request=true
```
{% include copy-curl.html %}

#### 清除特定字段的缓存

以下请求清除了字段缓存`fielda` 和`fieldb`：

```json
POST /my-index/_cache/clear?fields=fielda,fieldb
```
{% include copy-curl.html %}

#### 清除特定数据流或索引的缓存

以下请求清除了两个特定索引的缓存：

```json
POST /my-index,my-index2/_cache/clear
```
{% include copy-curl.html %}

#### 清除所有数据流和索引的缓存

以下请求清除了所有数据流和索引的缓存：

```json
POST /_cache/clear
```
{% include copy-curl.html %}

#### 清除搜索中缓存中的未使用的条目-有能力的节点

```json
POST /*/_cache/clear?file=true 
```
{% include copy-curl.html %}

#### 示例响应

这`POST /books,hockey/_cache/clear` 请求返回以下字段：

```json
{
  "_shards" : {
    "total" : 4,
    "successful" : 2,
    "failed" : 0
  }
}
```

## 响应字段

这`POST /books,hockey/_cache/clear` 请求返回以下响应字段：

| 场地| 数据类型| 描述| 
:--- | :--- | :---
| _沙尔德| 目的| 碎片信息。|
| 全部的| 整数| 碎片总数。|
| 成功的| 整数| 带有卡车的索引碎片的数量成功清除了。|
| 失败的| 整数| 带有缓存的索引碎片数量未能清除。|

