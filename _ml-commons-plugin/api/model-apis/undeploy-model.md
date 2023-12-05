---
layout: default
title: Uneploy模型
parent: 模型API
grand_parent: ML Commons API
nav_order: 40
---

# Uneploy A模型

为了使内存中的模型不乏力，请使用Undeploy操作。

有关此API的用户访问的信息，请参见[模型访问控制注意事项]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/model-apis/index/#model-access-control-considerations)。

### 路径和HTTP方法

```json
POST /_plugins/_ml/models/<model_id>/_undeploy
```

#### 示例请求：不启用所有ML节点的模型

```json
POST /_plugins/_ml/models/MGqJhYMBbbh0ushjm8p_/_undeploy
```
{% include copy-curl.html %}

#### 示例请求：从特定节点中不剥削特定模型

```json
POST /_plugins/_ml/models/_undeploy
{
  "node_ids": ["sv7-3CbwQW-4PiIsDOfLxQ"],
  "model_ids": ["KDo2ZYQB-v9VEDwdjkZ4"]
}
```
{% include copy-curl.html %}

#### 示例请求：所有节点的特定模型都不张紧

```json
{
  "model_ids": ["KDo2ZYQB-v9VEDwdjkZ4"]
}
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "sv7-3CbwQW-4PiIsDOfLxQ" : {
    "stats" : {
      "KDo2ZYQB-v9VEDwdjkZ4" : "UNDEPLOYED"
    }
  }
}
```

