---
layout: default
title: 异步搜索安全性
nav_order: 2
parent: 异步搜索
has_children: false
redirect_from:
 - /search-plugins/async/security/
---

# 异步搜索安全性

您可以使用异步搜索的安全插件来限制非-管理用户对特定操作。例如，您可能希望某些用户只能提交或删除异步搜索，而您可能希望其他人仅查看结果。

所有异步搜索索引都被保护为系统索引。只有具有传输层安全性（TLS）证书的超级管理用户或管理用户才能访问系统索引。有关更多信息，请参阅[系统索引]({{site.url}}{{site.baseurl}}/security/configuration/system-indices/)。

## 基本权限

作为管理用户，您可以使用安全插件将特定权限分配给用户，以根据他们需要访问的API操作。有关支持的API操作列表，请参见[异步搜索]({{site.url}}{{site.baseurl}}/)。

安全插件有两个构建-在涵盖大多数异步搜索用例的角色中：`asynchronous_search_full_access` 和`asynchronous_search_read_access`。对于每个描述，请参阅[预定义的角色]({{site.url}}{{site.baseurl}}/security/access-control/users-roles#predefined-roles)。

如果这些角色无法满足您的需求，请混合并匹配单独的异步搜索许可，以适合您的用例。每个动作对应于REST API中的操作。例如，`cluster:admin/opensearch/asynchronous_search/delete` 许可使您可以删除先前提交的异步搜索。

### 有关异步搜索和罚款的注释-粒度访问控制

通过设计，异步搜索插件从目标索引中提取数据，并将数据存储在单独的索引中，以使搜索结果可为用户提供适当的权限。虽然一个用户`asynchronous_search_read_access` 或者`cluster:admin/opensearch/asynchronous_search/get` 权限无法提交异步搜索请求本身，该用户可以使用关联的搜索ID获取和查看搜索结果。[文档-级别的安全性]({{site.url}}{{site.baseurl}}/security/access-control/document-level-security) （DLS）和[场地-级别的安全性]({{site.url}}{{site.baseurl}}/security/access-control/field-level-security) （FLS）访问控件旨在保护目标索引中的数据。但是，一旦将数据存储在此索引之外，具有这些访问权限的用户可以使用搜索ID来获取和查看异步搜索结果，该结果可能包括DLS和FLS访问权限控制在目标索引中的数据。

为了减少查看可以描述索引的搜索结果的意外用户的机会，我们建议管理员启用角色-在为预期的用户组分配权限时，基于访问控制并牢记这些设计元素。看[限制后端角色访问](#advanced-limit-access-by-backend-role) 有关详细信息。

## （高级）限制后端角色访问

使用后端角色来配置罚款-基于角色的粒状访问异步搜索。例如，组织中不同部门的用户可以查看自己部门拥有的异步搜索。

首先，确保您的用户有适当的[后端角色]({{site.url}}{{site.baseurl}}/security/access-control/index/)。后端角色通常来自[LDAP服务器]({{site.url}}{{site.baseurl}}/security/configuration/ldap/) 或者[SAML提供商]({{site.url}}{{site.baseurl}}/security/configuration/saml/)。但是，如果使用内部用户数据库，则可以将REST API使用[手动添加它们]({{site.url}}{{site.baseurl}}/security/access-control/api#create-user)。

现在，当用户在OpenSearch仪表板中查看异步搜索资源（或进行REST API调用）时，他们只会看到具有后端角色子集的用户提交的异步搜索。
例如，考虑两个用户：`judy` 和`elon`。

`judy` 具有IT后端角色：

```json
PUT _plugins/_security/api/internalusers/judy
{
  "password": "judy",
  "backend_roles": [
    "IT"
  ],
  "attributes": {}
}
```

`elon` 具有管理员的后端角色：

```json
PUT _plugins/_security/api/internalusers/elon
{
  "password": "elon",
  "backend_roles": [
    "admin"
  ],
  "attributes": {}
}
```

两个都`judy` 和`elon` 可以完全访问异步搜索：

```json
PUT _plugins/_security/api/rolesmapping/async_full_access
{
  "backend_roles": [],
  "hosts": [],
  "users": [
    "judy",
    "elon"
  ]
}
```

因为它们具有不同的后端角色，所以提交的异步搜索`judy` 看不到`elon` 反之亦然。

`judy` 至少需要具有所有角色的超集`elon` 必须看到`elon`的异步搜索。

例如，如果`judy` 有五个后端角色`elon` 具有这些角色之一，然后`judy` 可以看到由`elon`， 但`elon` 看不到提交的异步搜索`judy`。这意味着`judy` 可以在提交的异步搜索上执行和删除操作`elon`，但不是相反。

如果没有一个用户担任任何后端角色，则三个用户都将能够看到其他搜索。

例如，考虑三个用户：`judy`，`elon`， 和`jack`。

`judy`，`elon`， 和`jack` 没有设置的后端角色：

```json
PUT _plugins/_security/api/internalusers/judy
{
  "password": "judy",
  "backend_roles": [],
  "attributes": {}
}
```

```json
PUT _plugins/_security/api/internalusers/elon
{
  "password": "elon",
  "backend_roles": [],
  "attributes": {}
}
```

```json
PUT _plugins/_security/api/internalusers/jack
{
  "password": "jack",
  "backend_roles": [],
  "attributes": {}
}
```

两个都`judy` 和`elon` 可以完全访问异步搜索：

```json
PUT _plugins/_security/api/rolesmapping/async_full_access
{
  "backend_roles": [],
  "hosts": [],
  "users": ["judy","elon"]
}
```

`jack` 已阅读访问异步搜索结果：

```json
PUT _plugins/_security/api/rolesmapping/async_read_access
{
  "backend_roles": [],
  "hosts": [],
  "users": ["jack"]
}
```

由于没有用户担任后端角色，因此他们将能够看到彼此的异步搜索。因此，如果`judy` 提交异步搜索，`elon`，谁拥有完全访问权限，将能够看到该搜索。`jack`，谁已阅读访问，也将能够看到`judy`的异步搜索

