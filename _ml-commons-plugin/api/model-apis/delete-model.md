---
layout: default
title: 删除模型
parent: 模型API
grand_parent: ML Commons API
nav_order: 50
---

# 删除模型

删除基于`model_id`。

当您在模型组中删除最后一个模型版本时，该模型组将自动从索引中删除。
{： 。重要的}

有关此API的用户访问的信息，请参见[模型访问控制注意事项]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/model-apis/index/#model-access-control-considerations)。

## 路径和HTTP方法

```json
DELETE /_plugins/_ml/models/<model_id>
```

#### 示例请求

```json
DELETE /_plugins/_ml/models/MzcIJX8BA7mbufL6DOwl
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "_index" : ".plugins-ml-model",
  "_id" : "MzcIJX8BA7mbufL6DOwl",
  "_version" : 2,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 27,
  "_primary_term" : 18
}
```
