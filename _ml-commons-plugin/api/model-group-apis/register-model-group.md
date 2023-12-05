---
layout: default
title: 注册模型组
parent: 模型组API
grand_parent: ML Commons API
nav_order: 10
---

# 注册模型组

要注册模型组，请发送`POST` 请求`_register` 端点。您可以在`public`，`private`， 或者`restricted` 访问模式。

集群中的每个模型组名称必须在全球范围内唯一。
{： 。重要的}

有关更多信息，请参阅[模型访问控制]({{site.url}}{{site.baseurl}}/ml-commons-plugin/model-access-control/)。

## 路径和HTTP方法

```json
POST /_plugins/_ml/model_groups/_register
```

## 请求字段

下表列出了可用的请求字段。

场地|数据类型| 描述
:--- | :--- | :---
`name` | 细绳| 模型组名称。必需的。
`description` | 细绳| 模型组描述。选修的。
`access_mode` | 细绳| 此模型的访问模式。有效值是`public`，`private`， 和`restricted`。当此参数设置为`restricted`，您必须指定`backend_roles` 或者`add_all_backend_roles`，但不是两者。选修的。如果未指定任何安全参数（`access_mode`，`backend_roles`， 和`add_all_backend_roles`），默认`access_mode` 是`private`。
`backend_roles` | 大批| 模型所有者的后端角色列表要添加到模型中。只有在`access_mode` 是`restricted`。不能与`add_all_backend_roles`。选修的。
`add_all_backend_roles` | 布尔| 如果`true`，将模型所有者的所有后端角色添加到模型组中。默认为`false`。不能与`backend_roles`。管理员用户无法将此参数设置为`true`。选修的。

#### 示例请求

```json
POST /_plugins/_ml/model_groups/_register
{
    "name": "test_model_group_public",
    "description": "This is a public model group",
    "access_mode": "public"
}
```
{% include copy-curl.html %}

#### 示例响应

```json
{
    "model_group_id": "GDNmQ4gBYW0Qyy5ZcBcg",
    "status": "CREATED"
}
```

## 响应字段

下表列出了可用的响应字段。

场地|数据类型| 描述
:--- | :--- | :---
`model_group_id` | 细绳| 您可以使用的模型组ID访问此模型组。
`status` | 细绳| 操作状态。

## 注册公共模型组

如果您注册了一个模型组`public` 访问模式，任何具有访问群集的用户都可以访问此模型组中的任何模型。以下请求注册一个公共模型组：

```json
POST /_plugins/_ml/model_groups/_register
{
    "name": "test_model_group_public",
    "description": "This is a public model group",
    "access_mode": "public"
}
```
{% include copy-curl.html %}

## 注册限制模型组

要限制通过后端角色访问访问，您必须注册一个模型组`restricted` 访问模式。

注册模型组时，必须使用以下方法将一个或多个后端角色附加到模型上：
    - 在`backend_roles` 范围。
    - 设置`add_all_backend_roles` 参数为`true` 将所有后端角色添加到模型组。此选项不适合管理用户。

任何与模型组共享后端角色的用户都可以访问此模型组中的任何模型。这将用户授予用户角色所包含的权限，该权限映射到后端角色。

管理用户可以访问所有模型组，而不管其访问模式如何。
{: .note}

#### 示例请求：后端角色列表

以下请求注册一个受限制的模型组，只能由用户访问`IT` 后端角色：

```json
POST /_plugins/_ml/model_groups/_register
{
    "name": "model_group_test",
    "description": "This is an example description",
    "access_mode": "restricted",
    "backend_roles" : ["IT"]
}
```
{% include copy-curl.html %}

#### 示例请求：所有后端角色

以下请求注册一个受限制的模型组，将用户的所有后端角色添加到模型组：

```json
POST /_plugins/_ml/model_groups/_register
{
    "name": "model_group_test",
    "description": "This is an example description",
    "access_mode": "restricted",
    "add_all_backend_roles": "true"
}
```
{% include copy-curl.html %}

## 注册一个私人模型组

如果您注册了一个模型组`private` 访问模式，此模型组中的任何模型都将仅适用于您和管理员用户。以下请求注册一个私人模型组：

```json
POST /_plugins/_ml/model_groups/_register
{
    "name": "model_group_test",
    "description": "This is an example description",
    "access_mode": "private"
}
```
{% include copy-curl.html %}

如果您不指定任何`access_mode`，`backend_roles`， 或者`add_all_backend_roles`，该模型将有一个`private` 访问模式：

```json
POST /_plugins/_ml/model_groups/_register
{
    "name": "model_group_test",
    "description": "This is an example description"
}
```
{% include copy-curl.html %}

## 在禁用模型访问控制的集群中注册模型组

如果在群集上禁用模型访问控制（其中之一[先决条件](ml-commons-plugin/model-access-control/#model-access-control-prerequisites) 未满足），您可以在一个模型组中注册`name` 和`description` 但是无法指定任何访问参数（`model_access_name`，`backend_roles`， 或者`add_backend_roles`）。默认情况下，在这样的集群中，所有模型组都是公开的

