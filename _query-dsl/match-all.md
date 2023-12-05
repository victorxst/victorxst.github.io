---
layout: default
title: 匹配所有查询
nav_order: 65
---

# 匹配所有查询

这`match_all` 查询返回所有文档。如果您需要返回整个集合，则此查询可用于测试大型文档集。

```json
GET _search
{
  "query": {
    "match_all": {}
  }
}
```
{％包含副本-curl.html％}

这`match_all` 查询有一个`match_none` 对应物，很少有用：

```json
GET _search
{
  "query": {
    "match_none": {}
  }
}
```
{％包含副本-curl.html％

