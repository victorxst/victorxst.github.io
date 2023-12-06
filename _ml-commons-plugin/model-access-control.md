---
layout: default
title: 模型访问控制
has_children: false
nav_order: 20
---

# 模型访问控制
**引入2.9**
{: .label .label-purple }

您可以将带有ML Commons的安全插件用于管理对非特定型号的访问-管理用户。例如，组织中的一个部门可能希望限制其他部门的用户访问其模型。

为此，将用户分配一个或多个[_backend角色_]({{site.url}}{{site.baseurl}}/security/access-control/index/)。后备角色不是在用户配置期间为单个用户分配各个用户的角色，而是通过在用户登录时将后端角色分配给角色来映射一组用户的方法。例如，可以为用户分配一个用户`IT` 后端角色包括`ml_full_access` 角色，并具有完全访问所有ML Commons功能。或者，其他用户可以分配`HR` 后端角色包括`ml_readonly_access` 角色并限制阅读-仅访问机器学习（ML）功能。鉴于这种灵活性，后端角色可以提供更精细的-粒度访问模型，并使将多个用户分配到角色而不是单独映射角色和角色更容易。

## ML Commons角色

ML CONSONS插件具有两个保留角色：

- `ml_full_access`：授予对所有ML功能的完全访问权限，包括启动新的ML任务以及阅读或删除模型。
- `ml_readonly_access`：赠款阅读-仅访问与模型集群相关的ML任务，训练有素的模型和统计信息。不授予启动或删除ML任务或模型的权限。

## 模型组

对于访问控制，模型被组织成_ model groups _---特定模型的版本集合。像用户一样，模型组可以分配一个或多个后端角色。同一型号的所有版本共享相同的模型名称，并具有相同的后端角色或角色。

创建一个新的模型组时，您将被视为模型_owner_。即使其他用户将模型注册到此模型组，您仍然是模型及其所有版本的所有者。当模型所有者创建模型组时，所有者可以为此模型组指定以下_access模式：

- `public`：所有可以访问集群的用户都可以访问此模型组。
- `private`：只有模型所有者或管理员用户才能访问此模型组。
- `restricted`：所有者，管理用户或共享模型组的后端角色之一的任何用户都可以访问此模型组中的任何模型。创建一个`restricted` 模型组，所有者必须将一个或多个所有者的后端角色附加到模型上。

管理员可以在集群中访问所有模型组，而不管其访问模式如何。
{:.note}

## 模型访问控制先决条件

在使用模型访问控制之前，您必须满足以下先决条件：

1. 在群集上启用安全插件。有关更多信息，请参阅[OpenSearch中的安全性]({{site.url}}{{site.baseurl}}/security/)。
2. 为了`restricted` 模型组，确保管理员具有[分配给用户的后端角色](#assigning-backend-roles-to-users)。
3. [启用模型访问控制](#enabling-model-access-control) 在你的集群上。

如果未满足任何先决条件，则集群中的所有模型均为`public` 任何有访问群集的用户都可以访问。
{:.note}

## 将后端角色分配给用户

创建适当的后端角色并将这些角色分配给用户。后端角色通常来自[LDAP服务器]({{site.url}}{{site.baseurl}}/security/configuration/ldap/) 或者[SAML提供商]({{site.url}}{{site.baseurl}}/security/configuration/saml/)，但是如果使用内部用户数据库，则可以将REST API使用[手动添加它们]({{site.url}}{{site.baseurl}}/security/access-control/api#create-user)。

只有管理员用户才能为用户分配后端角色。
{:.note}

分配后端角色时，请考虑两个用户的以下示例：`alice` 和`bob`。

以下请求分配用户`alice` 这`analyst` 后端角色：

```json
PUT _plugins/_security/api/internalusers/alice
{
  "password": "alice",
  "backend_roles": [
    "analyst"
  ],
  "attributes": {}
}
```

下一个请求分配用户`bob` 这`human-resources` 后端角色：

```json
PUT _plugins/_security/api/internalusers/bob
{
  "password": "bob",
  "backend_roles": [
    "human-resources"
  ],
  "attributes": {}
}
```

最后，最后一个请求都分配了`alice` 和`bob` 使他们完全访问ML Commons的角色：

```json
PUT _plugins/_security/api/rolesmapping/ml_full_access
{
  "backend_roles": [],
  "hosts": [],
  "users": [
    "alice",
    "bob"
  ]
}
```

如果`alice` 创建一个模型组并分配`analyst` 后端角色，`bob` 无法访问此模型。

## 启用模型访问控制

您可以动态启用模型访问控制，如下所示：

```json
PUT _cluster/settings
{
  "transient": {
    "plugins.ml_commons.model_access_control_enabled": "true"
  }
}
```
{% include copy-curl.html %}

## 模型访问控制API

模型访问控制是通过模型组API实现的。这些API包括寄存器，搜索，更新和删除模型组操作。

有关模型访问控制API的信息，请参阅[模型组API]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/model-group-apis/index/)

