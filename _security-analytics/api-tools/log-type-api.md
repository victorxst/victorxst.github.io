---
layout: default
title: 日志类型API
parent: API工具
nav_order: 56
---

# 日志类型API

日志类型API允许您创建自定义日志类型，搜索自定义日志类型，更新自定义日志类型以及删除自定义日志类型。


## 创建日志类型

创建新的自定义日志类型涉及输入名称和描述，并将源指定为`Custom`。


### 示例请求

```json
POST /_plugins/_security_analytics/logtype
{
  "description": "custom-log-type-desc",
  "name": "custom-log-type4",
  "source": "Custom"
}
```
{% include copy-curl.html %}


### 示例响应

```json
{
    "_id": "m98uk4kBlb9cbROIpEj2",
    "_version": 1,
    "logType": {
        "name": "custom-log-type4",
        "description": "custom-log-type-desc",
        "source": "Custom",
        "tags": {
            "correlation_id": 27
        }
    }
}
```


## 搜索自定义日志类型

此API允许您在系统中搜索日志类型。


### 示例请求

```json
POST /_plugins/_security_analytics/logtype/_search
{
    "query": {
        "match_all": {}
    }
}
```
{% include copy-curl.html %}


### 示例响应

```json
{
    "took": 3,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 26,
            "relation": "eq"
        },
        "max_score": 2.0,
        "hits": [
            {
                "_index": ".opensearch-sap-log-types-config",
                "_id": "s3",
                "_score": 2.0,
                "_source": {
                    "name": "s3",
                    "description": "Windows logs",
                    "source": "Sigma",
                    "tags": {
                        "correlation_id": 21
                    }
                }
            },
            {
                "_index": ".opensearch-sap-log-types-config",
                "_id": "others_compliance",
                "_score": 2.0,
                "_source": {
                    "name": "others_compliance",
                    "description": "Compliance logs",
                    "source": "Sigma",
                    "tags": {
                        "correlation_id": 4
                    }
                }
            },
            {
                "_index": ".opensearch-sap-log-types-config",
                "_id": "github",
                "_score": 2.0,
                "_source": {
                    "name": "github",
                    "description": "Sys logs",
                    "source": "Sigma",
                    "tags": {
                        "correlation_id": 16
                    }
                }
            },
            {
                "_index": ".opensearch-sap-log-types-config",
                "_id": "others_application",
                "_score": 2.0,
                "_source": {
                    "name": "others_application",
                    "description": "Application logs",
                    "source": "Sigma",
                    "tags": {
                        "correlation_id": 0
                    }
                }
            },
            {
                "_index": ".opensearch-sap-log-types-config",
                "_id": "dns",
                "_score": 2.0,
                "_source": {
                    "name": "dns",
                    "description": "Compliance logs",
                    "source": "Sigma",
                    "tags": {
                        "correlation_id": 15
                    }
                }
            },
            {
                "_index": ".opensearch-sap-log-types-config",
                "_id": "m98uk4kBlb9cbROIpEj2",
                "_score": 2.0,
                "_source": {
                    "name": "custom-log-type-updated4",
                    "description": "custom-log-type-updated-desc",
                    "source": "Custom",
                    "tags": null
                }
            }
        ]
    }
}
```


## 更新自定义日志类型

此API允许您更新现有的自定义日志类型。使用路由中的日志类型的ID来指定日志类型，如以下示例所示：

```json
PUT /_plugins/_security_analytics/logtype/<log_type_id>
```


### 示例请求

```json
PUT /_plugins/_security_analytics/logtype/m98uk4kBlb9cbROIpEj2
{
  "name": "custom-log-type4",
  "description": "custom-log-type-updated-desc",
  "source": "Custom"
}
```
{% include copy-curl.html %}


### 示例响应

```json
{
    "_id": "m98uk4kBlb9cbROIpEj2",
    "_version": 1,
    "logType": {
        "name": "custom-log-type4",
        "description": "custom-log-type-updated-desc",
        "source": "Custom",
        "tags": {
            "correlation_id": 27
        }
    }
}
```


## 删除自定义日志类型

此API用于删除自定义日志类型。在路由中指定日志类型的ID以运行操作：

```json
DELETE /_plugins/_security_analytics/logtype/<log_type_id>
```


### 示例请求

```json
DELETE /_plugins/_security_analytics/logtype/m98uk4kBlb9cbROIpEj2
```
{% include copy-curl.html %}


### 示例响应

```json
200 OK
{
    "_id": "m98uk4kBlb9cbROIpEj2",
    "_version": 1
}
```

只能删除自定义日志类型。试图删除标准搜索-定义的日志类型导致错误。
{: .note}


