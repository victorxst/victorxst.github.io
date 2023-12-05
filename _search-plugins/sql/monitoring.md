---
layout: default
title: 监视
parent: SQL和PPL
nav_order: 95
redirect_from:
  - /search-plugins/sql/monitoring/
---

# 监视

通过统计信息，您可以收集插件的指标
在间隔内。请注意，只有节点级统计信息是
现在实施。换句话说，您只能获得
您要访问的节点。集群级统计尚未
实施的。

## 节点统计

### 描述

响应中字段的含义如下：

|                 字段名称|                                                    描述|
| ------------------------- | ------------------------------------------------------------- |
|              request_total|                                         请求总数|
|              request_count|                     间隔内请求的总数|
|FAILED_REQUEST_COUNT_SYSERR|由于系统错误在间隔内由于系统错误而导致的次数计数|
|FAILED_REQUEST_COUNT_CUSERR| 在间隔内由于不良请求而导致的不良请求计数|
|    FAILED_REQUEST_COUNT_CB| 指示插件是否在间隔内断开|


### 例子

SQL查询：

```console
>> curl -H 'Content-Type: application/json' -X GET localhost:9200/_plugins/_sql/stats
```

结果集：

```json
{
  "failed_request_count_cb": 0,
  "failed_request_count_cuserr": 0,
  "circuit_breaker": 0,
  "request_total": 0,
  "request_count": 0,
  "failed_request_count_syserr": 0
}
```

