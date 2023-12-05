---
layout: default
title: 统计 
parent: ML Commons API
nav_order: 50
---

# 统计

获取与任务数量相关的统计信息。

## 路径和HTTP方法

```json
GET /_plugins/_ml/stats
GET /_plugins/_ml/stats/<stat>
GET /_plugins/_ml/<nodeId>/stats/
GET /_plugins/_ml/<nodeId>/stats/<stat>
```

#### 示例请求：获取所有节点的所有统计信息

```json
GET /_plugins/_ml/stats
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "zbduvgCCSOeu6cfbQhTpnQ" : {
    "ml_executing_task_count" : 0
  },
  "54xOe0w8Qjyze00UuLDfdA" : {
    "ml_executing_task_count" : 0
  },
  "UJiykI7bTKiCpR-rqLYHyw" : {
    "ml_executing_task_count" : 0
  },
  "zj2_NgIbTP-StNlGZJlxdg" : {
    "ml_executing_task_count" : 0
  },
  "jjqFrlW7QWmni1tRnb_7Dg" : {
    "ml_executing_task_count" : 0
  },
  "3pSSjl5PSVqzv5-hBdFqyA" : {
    "ml_executing_task_count" : 0
  },
  "A_IiqoloTDK01uZvCjREaA" : {
    "ml_executing_task_count" : 0
  }
}
```

#### 示例请求：获取特定节点的所有统计信息

```json
GET /_plugins/_ml/<nodeId>/stats/
```
{% include copy-curl.html %}

#### 示例请求：获取特定节点的指定统计信息

```json
GET /_plugins/_ml/<nodeId>/stats/<stat>
```
{% include copy-curl.html %}

#### 示例请求：获取所有节点的指定统计信息

```json
GET /_plugins/_ml/stats/<stat>
```
{% include copy-curl.html %}





