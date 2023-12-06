---
layout: default
title: 删除管道
nav_order: 13
redirect_from:
  - /opensearch/rest-api/ingest-apis/delete-ingest/
  - /api-reference/ingest-apis/delete-ingest/
---

# 删除管道
**引入 1.0**
{: .label .label-purple }

使用以下请求删除管道。

若要删除特定管道，请将管道 ID 作为参数传递：

```json
DELETE /_ingest/pipeline/<pipeline-id>
```
{% include copy-curl.html %}

若要删除群集中的所有管道，请使用通配符（ `*`）：

```json
DELETE /_ingest/pipeline/*
```
{% include copy-curl.html %}
