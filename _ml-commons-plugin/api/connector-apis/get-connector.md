---
layout: default
title: 获取连接器
parent: 连接器API
grand_parent: ML Commons API
nav_order: 20
---

# 获取连接器

使用`_search` 搜索连接器的端点。

要检索有关连接器的信息，您可以：

- [通过ID获取连接器](#get-a-connector-by-id)
- [搜索连接器](#search-for-a-connector)

# 通过ID获取连接器

该API通过其ID检索连接器。

## 路径和HTTP方法

```json
GET /_plugins/_ml/connectors/<connector_id>
```

#### 示例请求

```json
GET /_plugins/_ml/connectors/N8AE1osB0jLkkocYjz7D
```
{% include copy-curl.html %}

## 搜索连接器

此API使用查询搜索匹配的连接器。

## 路径和HTTP方法

```json
POST /_plugins/_ml/connectors/_search
GET /_plugins/_ml/connectors/_search
```

#### 示例请求

```json
POST /_plugins/_ml/connectors/_search
{
  "query": {
    "match_all": {}
  },
  "size": 1000
}
```
{% include copy-curl.html %}


