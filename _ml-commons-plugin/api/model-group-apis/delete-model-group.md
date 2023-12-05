---
layout: default
title: 删除模型组
parent: 模型组API
grand_parent: ML Commons API
nav_order: 40
---

# 删除模型组

您只能在不包含任何模型版本的情况下删除该模型组。
{： 。重要的}

如果在群集上启用了模型访问控制，则只有具有匹配后端角色的所有者或用户才能删除模型组。任何用户都可以删除任何公共模型组。

如果在群集上禁用模型访问控制，则用用户`delete model group API` 权限可以删除任何模型组。

管理用户可以删除任何模型组。
{: .note}

当您在模型组中删除最后一个模型版本时，该模型组将自动从索引中删除。
{： 。重要的}

有关更多信息，请参阅[模型访问控制]({{site.url}}{{site.baseurl}}/ml-commons-plugin/model-access-control/)。

#### 示例请求

```json
DELETE _plugins/_ml/model_groups/<model_group_id>
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "_index": ".plugins-ml-model-group",
  "_id": "l8nnQogByXnLJ-QNpEk2",
  "_version": 5,
  "result": "deleted",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 70,
  "_primary_term": 23
}
```

