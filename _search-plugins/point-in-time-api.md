---
layout: default
title: 时间点API
nav_order: 59
has_children: false
parent: 时间点
redirect_from:
  - /opensearch/point-in-time-api/
---

# 时间点API

使用[时间点（坑）]({{site.url}}{{site.baseurl}}/opensearch/point-in-time/) API管理坑。

---

#### Table of contents
- TOC
{:toc}

---

## 创建一个坑
引入2.4
{: .label .label-purple }

创建一个坑。这`keep_alive` 需要查询参数；它指定了要保持坑的时间。

### 路径和HTTP方法

```json
POST /<target_indexes>/_search/point_in_time?keep_alive=1h&routing=&expand_wildcards=&preference= 
```

### 路径参数

范围| 数据类型| 描述
:--- | :--- | :---
target_indexes| 细绳| 坑的目标索引的名称。可能包含一个逗号-分开列表或通配符索引模式。

### 查询参数

范围| 数据类型| 描述
:--- | :--- | :---
活着| 时间|  保持坑的时间。每当您使用搜索API访问坑时，坑寿命都会延长到等于`keep_alive` 范围。必需的。
偏爱| 细绳| 节点或用于执行搜索的碎片。选修的。默认值是随机的。
路由| 细绳| 指定将搜索请求路由到特定的碎片。选修的。默认是文档的`_id`。
Expand_WildCard| 细绳| 可以匹配通配符模式的索引类型。支持逗号-分离的值。有效值如下：<br>- `all`：匹配任何索引或数据流，包括隐藏的索引流。<br>- `open`：匹配打开，非-隐藏索引或非-隐藏的数据流。<br>- `closed`：比赛关闭，非-隐藏索引或非-隐藏的数据流。<br>- `hidden`：匹配隐藏索引或数据流。必须与`open`，`closed` 或两者`open` 和`closed`。<br>- `none`：不接受通配符模式。<br>可选。默认为`open`。
laster_partial_pit_creation| 布尔| 指定是否要创建部分失败。选修的。默认为`true`。

#### 示例请求

```json
POST /my-index-1/_search/point_in_time?keep_alive=100m
```

#### 示例响应

```json
{
    "pit_id": "o463QQEPbXktaW5kZXgtMDAwMDAxFnNOWU43ckt3U3IyaFVpbGE1UWEtMncAFjFyeXBsRGJmVFM2RTB6eVg1aVVqQncAAAAAAAAAAAIWcDVrM3ZIX0pRNS1XejE5YXRPRFhzUQEWc05ZTjdyS3dTcjJoVWlsYTVRYS0ydwAA",
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "creation_time": 1658146050064
}
```

### 响应字段

场地| 数据类型| 描述
:--- | :--- | :---  
pit_id| [BASE64编码二进制]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/binary) | 坑ID。
creation_time| 长的| 自时代以来，矿坑的创建时间是在毫秒内创建的。

## 延长进站

您可以通过提供一个`keep_alive` 参数`pit` 当您执行搜索时对象：

```json
GET /_search
{
  "size": 10000,
  "query": {
    "match" : {
      "user.id" : "elkbee"
    }
  },
  "pit": {
    "id":  "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA==", 
    "keep_alive": "100m"
  },
  "sort": [ 
    {"@timestamp": {"order": "asc"}}
  ],
  "search_after": [  
    "2021-05-20T05:30:04.832Z"
  ]
}
```

这`keep_alive` 搜索请求中的参数是可选的。它指定了延长时间以保持坑的时间的数量。
{:.note}

## 列出所有坑
引入2.4
{: .label .label-purple }

返回OpenSearch集群中的所有坑。

### 叉-聚类行为

列表所有坑API仅返回本地坑或混合坑（在本地和远程簇中创建的坑）。它不会返回完全远程凹坑。

#### 示例请求

```json
GET /_search/point_in_time/_all
```

#### 示例响应

```json
{
    "pits": [
        {
            "pit_id": "o463QQEPbXktaW5kZXgtMDAwMDAxFnNOWU43ckt3U3IyaFVpbGE1UWEtMncAFjFyeXBsRGJmVFM2RTB6eVg1aVVqQncAAAAAAAAAAAEWcDVrM3ZIX0pRNS1XejE5YXRPRFhzUQEWc05ZTjdyS3dTcjJoVWlsYTVRYS0ydwAA",
            "creation_time": 1658146048666,
            "keep_alive": 6000000
        },
        {
            "pit_id": "o463QQEPbXktaW5kZXgtMDAwMDAxFnNOWU43ckt3U3IyaFVpbGE1UWEtMncAFjFyeXBsRGJmVFM2RTB6eVg1aVVqQncAAAAAAAAAAAIWcDVrM3ZIX0pRNS1XejE5YXRPRFhzUQEWc05ZTjdyS3dTcjJoVWlsYTVRYS0ydwAA",
            "creation_time": 1658146050064,
            "keep_alive": 6000000
        }
    ]
}
```

### 响应字段

场地| 数据类型| 描述
:--- | :--- | :---  
坑| JSON对象的数组| 所有坑的列表。

每个坑对象都包含以下字段。

场地| 数据类型| 描述
:--- | :--- | :---  
pit_id| [BASE64编码二进制]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/binary) | 坑ID。
creation_time| 长的| 自时代以来，矿坑的创建时间是在毫秒内创建的。
活着| 长的|  将坑保持在毫秒内的时间。

## 删除坑
引入2.4
{: .label .label-purple }

删除一个，几个或所有坑。当坑被自动删除时`keep_alive` 时间段。但是，要处理资源，您可以使用DELETE PIT API删除坑。删除PIT API支持通过ID或一次删除所有坑来删除坑列表。

### 叉-聚类行为

ID API的删除坑完全支持删除十字架-集群坑。

删除所有凹坑API仅删除本地坑或混合坑（在本地和远程簇中创建的凹坑）。它不会删除完全远程的凹坑。

#### 示例请求：删除所有坑

```json
DELETE /_search/point_in_time/_all
```

如果要删除一个或几个坑，请在请求正文中指定其坑ID。

### 请求字段

场地| 数据类型| 描述
:--- | :--- | :---
pit_id| [BASE64编码二进制]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/binary) 或一系列二进制| 要删除的坑的坑ID。必需的。

#### 示例请求：通过ID删除坑

```json
DELETE /_search/point_in_time

{
    "pit_id": [
        "o463QQEPbXktaW5kZXgtMDAwMDAxFkhGN09fMVlPUkVPLXh6MUExZ1hpaEEAFjBGbmVEZHdGU1EtaFhhUFc4ZkR5cWcAAAAAAAAAAAEWaXBPNVJtZEhTZDZXTWFFR05waXdWZwEWSEY3T18xWU9SRU8teHoxQTFnWGloQQAA",
        "o463QQEPbXktaW5kZXgtMDAwMDAxFkhGN09fMVlPUkVPLXh6MUExZ1hpaEEAFjBGbmVEZHdGU1EtaFhhUFc4ZkR5cWcAAAAAAAAAAAIWaXBPNVJtZEhTZDZXTWFFR05waXdWZwEWSEY3T18xWU9SRU8teHoxQTFnWGloQQAA"
    ]
}
```

#### 示例响应

对于每个坑，响应包含一个带有坑ID和一个的JSON对象`successful` 指定删除是否成功的字段。部分失败被视为失败。

```json
{
    "pits": [
        {
            "successful": true,
            "pit_id": "o463QQEPbXktaW5kZXgtMDAwMDAxFkhGN09fMVlPUkVPLXh6MUExZ1hpaEEAFjBGbmVEZHdGU1EtaFhhUFc4ZkR5cWcAAAAAAAAAAAEWaXBPNVJtZEhTZDZXTWFFR05waXdWZwEWSEY3T18xWU9SRU8teHoxQTFnWGloQQAA"
        },
        {
            "successful": false,
            "pit_id": "o463QQEPbXktaW5kZXgtMDAwMDAxFkhGN09fMVlPUkVPLXh6MUExZ1hpaEEAFjBGbmVEZHdGU1EtaFhhUFc4ZkR5cWcAAAAAAAAAAAIWaXBPNVJtZEhTZDZXTWFFR05waXdWZwEWSEY3T18xWU9SRU8teHoxQTFnWGloQQAA"
        }
    ]
}
```

### 响应字段

场地| 数据类型| 描述
:--- | :--- | :---
成功的| 布尔| 删除操作是否成功。
pit_id| [BASE64编码二进制]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/binary)  | 要删除的坑的坑ID。

## 坑段
引入2.4
{: .label .label-purple }

类似于[猫节API]({{site.url}}{{site.baseurl}}/api-reference/cat/cat-segments)，坑段API可提供低-通过描述其Lucene段的磁盘利用磁盘利用的水平信息。PIT段API支持列出特定坑的段信息ID或所有凹坑的信息。

#### 示例请求：所有坑的坑段

```json
GET /_cat/pit_segments/_all
```

如果要列出一个或几个坑的细分，请在请求正文中指定其坑ID。

### 请求字段

场地| 数据类型| 描述
:--- | :--- | :---
pit_id| [BASE64编码二进制]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/binary) 或一系列二进制| 将要列出的坑的坑ID。必需的。

#### 示例请求：ID的坑段

```json
GET /_cat/pit_segments

{
    "pit_id": [
        "o463QQEPbXktaW5kZXgtMDAwMDAxFkhGN09fMVlPUkVPLXh6MUExZ1hpaEEAFjBGbmVEZHdGU1EtaFhhUFc4ZkR5cWcAAAAAAAAAAAEWaXBPNVJtZEhTZDZXTWFFR05waXdWZwEWSEY3T18xWU9SRU8teHoxQTFnWGloQQAA",
        "o463QQEPbXktaW5kZXgtMDAwMDAxFkhGN09fMVlPUkVPLXh6MUExZ1hpaEEAFjBGbmVEZHdGU1EtaFhhUFc4ZkR5cWcAAAAAAAAAAAIWaXBPNVJtZEhTZDZXTWFFR05waXdWZwEWSEY3T18xWU9SRU8teHoxQTFnWGloQQAA"
    ]
}
```

#### 示例响应

```json
index  shard prirep ip            segment generation docs.count docs.deleted  size size.memory committed searchable version compound
index1 0     r      10.212.36.190 _0               0          4            0 3.8kb        1364 false     true       8.8.2   true
index1 1     p      10.212.36.190 _0               0          3            0 3.7kb        1364 false     true       8.8.2   true
index1 2     r      10.212.74.139 _0               0          2            0 3.6kb        1364 false     true       8.8.2   true
```

## 维修区设置

您可以为坑指定以下设置。

环境| 描述| 默认
:--- | :--- | :---
point_in_time.max_keep_alive| 集群-指定最大值的级别设置`keep_alive` 范围。| 24H
search.max_open_pit_context| 节点-指定节点的最大开放端上下文数量的级别设置。| 30

