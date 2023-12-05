---
layout: default
title: 创建连接器
parent: 连接器API
grand_parent: ML Commons API
nav_order: 10
---

# 创建一个连接器

创建一个独立的连接器。有关更多信息，请参阅[连接器]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/connectors/)。

## 路径和HTTP方法

```json
POST /_plugins/_ml/connectors/_create
```

#### 示例请求

要创建独立的连接器，请将请求发送到`connectors/_create` 端点并提供所有描述的参数[连接器蓝图]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/blueprints/)：

```json
POST /_plugins/_ml/connectors/_create
{
    "name": "OpenAI Chat Connector",
    "description": "The connector to public OpenAI model service for GPT 3.5",
    "version": 1,
    "protocol": "http",
    "parameters": {
        "endpoint": "api.openai.com",
        "model": "gpt-3.5-turbo"
    },
    "credential": {
        "openAI_key": "..."
    },
    "actions": [
        {
            "action_type": "predict",
            "method": "POST",
            "url": "https://${parameters.endpoint}/v1/chat/completions",
            "headers": {
                "Authorization": "Bearer ${credential.openAI_key}"
            },
            "request_body": "{ \"model\": \"${parameters.model}\", \"messages\": ${parameters.messages} }"
        }
    ]
}
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "connector_id": "a1eMb4kBJ1eYAeTMAljY"
}
```
