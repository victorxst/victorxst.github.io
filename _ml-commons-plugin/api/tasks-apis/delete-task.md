---
layout: default
title: 删除任务
parent: 任务API
grand_parent: ML Commons API
nav_order: 20
---

# 删除任务

根据`task_id`。

运行删除请求时，ML Commons不会检查任务状态。在任务完成之前，可以删除当前正在运行的任务的风险。要检查任务的状态，请运行`GET /_plugins/_ml/tasks/<task_id>` 在任务删除之前。
{:.note}

### 路径和HTTP方法

```json
DELETE /_plugins/_ml/tasks/<task_id>
```

#### 示例请求

```json
DELETE /_plugins/_ml/tasks/xQRYLX8BydmmU1x6nuD3
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "_index" : ".plugins-ml-task",
  "_id" : "xQRYLX8BydmmU1x6nuD3",
  "_version" : 4,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 42,
  "_primary_term" : 7
}
```
