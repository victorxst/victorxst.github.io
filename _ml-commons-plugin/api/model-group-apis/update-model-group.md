---
layout: default
title: 更新模型组
parent: 模型组API
grand_parent: ML Commons API
nav_order: 20
---

# 更新模型组

要更新模型组，请发送`PUT` 请求`model_groups` 端点并提供要更新的模型组的ID。

更新模型组时，适用以下限制：

- 模型所有者或管理员用户可以更新所有字段。任何与模型组共享一个或多个后端角色的用户都可以更新`name` 和`description` 仅字段。
- 更新时`access_mode` 到`restricted`，您必须指定`backend_roles` 或者`add_all_backend_roles` 但不是两者。
- 更新时`name`，确保该名称在集群中是全球唯一的。

有关更多信息，请参阅[模型访问控制]({{site.url}}{{site.baseurl}}/ml-commons-plugin/model-access-control/)。

## 路径和HTTP方法

```json
PUT /_plugins/_ml/model_groups/<model_group_id>
```

## 请求字段

参考[请求字段](#request-fields) 用于请求字段说明。

#### 示例请求

```json
PUT /_plugins/_ml/model_groups/<model_group_id>
{
    "name": "model_group_test",
    "description": "This is the updated description",
    "add_all_backend_roles": true
}
```
{% include copy-curl.html %}

## 在禁用模型访问控制的集群中更新模型组

如果在群集上禁用模型访问控制（其中之一[先决条件](ml-commons-plugin/model-access-control/#model-access-control-prerequisites) 未满足），您只能更新`name` 和`description` 模型组但无法更新任何访问参数（`model_access_name`，`backend_roles`， 或者`add_backend_roles`）。

