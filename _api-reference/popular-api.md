---
layout: default
title: 流行的API
nav_order: 96
redirect_from:
  - /opensearch/popular-api/
---

# 流行的API
**引入1.0**
{: .label .label-purple }

此页面包含示例请求流行的OpenSearch操作。


---

#### Table of contents
1. TOC
{:toc}


---

## 用非-默认设置

```json
PUT my-logs
{
  "settings": {
    "number_of_shards": 4,
    "number_of_replicas": 2
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "year": {
        "type": "integer"
      }
    }
  }
}
```


## 索引带有随机ID的文档

```json
POST my-logs/_doc
{
  "title": "Your Name",
  "year": "2016"
}
```


## 索引具有特定ID的文档

```json
PUT my-logs/_doc/1
{
  "title": "Weathering with You",
  "year": "2019"
}
```


## 一次索引几个文档

要求主体末尾的空白线。如果您省略了`_id` 字段，OpenSearch生成一个随机ID。

```json
POST _bulk
{ "index": { "_index": "my-logs", "_id": "2" } }
{ "title": "The Garden of Words", "year": 2013 }
{ "index" : { "_index": "my-logs", "_id" : "3" } }
{ "title": "5 Centimeters Per Second", "year": 2007 }

```


## 列出所有索引

```
GET _cat/indices?v&expand_wildcards=all
```


## 打开或关闭与模式匹配的所有索引

```
POST my-logs*/_open
POST my-logs*/_close
```


## 删除与模式匹配的所有索引

```
DELETE my-logs*
```


## 创建索引别名

此请求创建别名`my-logs-today` 对于索引`my-logs-2019-11-13`。

```
PUT my-logs-2019-11-13/_alias/my-logs-today
```


## 列出所有别名

```
GET _cat/aliases?v
```


## 搜索与模式匹配的索引或所有索引

```
GET my-logs/_search?q=test
GET my-logs*/_search?q=test
```


## 获取集群设置，包括默认设置

```
GET _cluster/settings?include_defaults=true
```


## 更改磁盘水印（或其他群集设置）

```json
PUT _cluster/settings
{
  "transient": {
    "cluster.routing.allocation.disk.watermark.low": "80%",
    "cluster.routing.allocation.disk.watermark.high": "85%"
  }
}
```


## 获得群集健康

```
GET _cluster/health
```


## 在集群中列出节点

```
GET _cat/nodes?v
```


## 获取节点统计

```
GET _nodes/stats
```


## 在存储库中获取快照

```
GET _snapshot/my-repository/_all
```


## 拍摄快照

```
PUT _snapshot/my-repository/my-snapshot
```


## 还原快照

```json
POST _snapshot/my-repository/my-snapshot/_restore
{
  "indices": "-.opendistro_security",
  "include_global_state": false
}
```

