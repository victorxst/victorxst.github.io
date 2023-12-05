---
layout: default
title: 部署模型
parent: 模型API
grand_parent: ML Commons API
nav_order: 30
---

# 部署模型

部署模型操作从模型索引中读取模型的块，然后创建模型的实例，以缓存到内存中。此操作需要`model_id`。

有关此API的用户访问的信息，请参见[模型访问控制注意事项]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/model-apis/index/#model-access-control-considerations)。

## 路径和HTTP方法

```json
POST /_plugins/_ml/models/<model_id>/_deploy
```

#### 示例请求：部署到所有可用的ML节点

在此示例请求中，OpenSearch将模型部署到任何可用的OpenSearch ML节点：

```json
POST /_plugins/_ml/models/WWQI44MBbzI2oUKAvNUt/_deploy
```
{% include copy-curl.html %}

#### 示例请求：部署到特定节点

如果要在群集中保留其他ML节点的内存，则可以通过指定该模型将模型部署到特定节点`node_ids` 在请求主体中：

```json
POST /_plugins/_ml/models/WWQI44MBbzI2oUKAvNUt/_deploy
{
    "node_ids": ["4PLK7KJWReyX0oWKnBA8nA"]
}
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "task_id" : "hA8P44MBhyWuIwnfvTKP",
  "status" : "DEPLOYING"
}
```

## 检查模型部署的状态

要查看模型部署的状态并检索为新模型版本创建的模型ID，请通过`task_id` 作为任务API的路径参数：

```json
GET /_plugins/_ml/tasks/hA8P44MBhyWuIwnfvTKP
```
{% include copy-curl.html %}

响应包含模型版本的模型ID：

```json
{
  "model_id": "Qr1YbogBYOqeeqR7sI9L",
  "task_type": "DEPLOY_MODEL",
  "function_name": "TEXT_EMBEDDING",
  "state": "COMPLETED",
  "worker_node": [
    "N77RInqjTSq_UaLh1k0BUg"
  ],
  "create_time": 1685478486057,
  "last_update_time": 1685478491090,
  "is_async": true
}
```
