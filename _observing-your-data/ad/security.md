---
layout: default
title: 异常检测安全性
nav_order: 10
parent: 异常检测
has_children: false
redirect_from: 
  - /monitoring-plugins/ad/security/
---

# 异常检测安全性

您可以在OpenSearch中使用带有异常检测的安全插件来限制非-管理用户对特定操作。例如，您可能希望某些用户只能创建，更新或删除检测器，而另一些用户只能查看检测器。

所有异常检测指数均被保护为系统索引。只有具有TLS证书的超级管理用户或管理用户才能访问系统索引。有关更多信息，请参阅[系统索引]({{site.url}}{{site.baseurl}}/security/configuration/system-indices/)。


异常检测的安全性与[警报的安全性]({{site.url}}{{site.baseurl}}/monitoring-plugins/alerting/security/)。

## 基本权限

作为管理用户，您可以使用安全插件将特定权限分配给用户，以基于他们需要访问的API。有关支持API的列表，请参阅[异常检测API]({{site.url}}{{site.baseurl}}/monitoring-plugins/ad/api/)。

安全插件有两个构建-在涵盖大多数异常检测用例的角色中：`anomaly_full_access` 和`anomaly_read_access`。对于每个描述，请参阅[预定义的角色]({{site.url}}{{site.baseurl}}/security/access-control/users-roles#predefined-roles)。

如果这些角色无法满足您的需求，请混合并匹配单个异常检测[权限]({{site.url}}{{site.baseurl}}/security/access-control/permissions/) 适合您的用例。每个动作对应于REST API中的操作。例如，`cluster:admin/opensearch/ad/detector/delete` 许可使您可以删除检测器。

### 警报和罚款的注释-粒度访问控制

当触发器生成警报时，检测器和监视配置，警报本身以及发送到通道的任何通知都可能包括描述要查询的索引的元数据。根据设计，插件必须提取数据并将其存储为索引外的元数据。[文档-级别的安全性]({{site.url}}{{site.baseurl}}/security/access-control/document-level-security) （DLS）和[场地-级别的安全性]({{site.url}}{{site.baseurl}}/security/access-control/field-level-security) （FLS）访问控件旨在保护索引中的数据。但是，一旦将数据存储在索引外作为元数据之外，可以访问检测器和监视配置，警报及其通知的用户将能够查看此元数据，并可能推断索引中数据的内容和质量，否则，这将是被DLS和FLS访问控制所隐藏。

为了减少查看可以描述索引的元数据的意外用户的机会，我们建议管理员启用角色-在为预期的用户组分配权限时，基于访问控制并牢记这些设计元素。看[限制后端角色访问](#advanced-limit-access-by-backend-role) 有关详细信息。

## （高级）限制后端角色访问

使用后端角色来配置罚款-基于角色的粒状访问单个探测器。例如，组织中不同部门的用户可以查看自己部门拥有的检测器。

首先，确保您的用户有适当的[后端角色]({{site.url}}{{site.baseurl}}/security/access-control/index/)。后端角色通常来自[LDAP服务器]({{site.url}}{{site.baseurl}}/security/configuration/ldap/) 或者[SAML提供商]({{site.url}}{{site.baseurl}}/security/configuration/saml/)，但是如果使用内部用户数据库，则可以将REST API使用[手动添加它们]({{site.url}}{{site.baseurl}}/security/access-control/api#create-user)。

接下来，启用以下设置：

```json
PUT _cluster/settings
{
  "transient": {
    "plugins.anomaly_detection.filter_by_backend_roles": "true"
  }
}
```

现在，当用户在OpenSearch仪表板中查看异常检测资源（或进行REST API调用）时，他们只会看到由共享至少一个后端角色的用户创建的检测器。
例如，考虑两个用户：`alice` 和`bob`。

`alice` 有一个分析师的后端角色：

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

`bob` 有人类-资源后端角色：

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

两个都`alice` 和`bob` 可以完全访问异常检测：

```json
PUT _plugins/_security/api/rolesmapping/anomaly_full_access
{
  "backend_roles": [],
  "hosts": [],
  "users": [
    "alice",
    "bob"
  ]
}
```

因为他们有不同的后端角色，所以`alice` 和`bob` 无法查看彼此的探测器或结果。

如果用户没有后端角色，只要他们拥有，他们仍然可以查看其他用户的异常检测结果`anomaly_read_access`。对于拥有的用户来说，这是相同的`anomaly_full_access`，因为它包括所有权限`anomaly_read_access`。管理员应告知用户拥有`anomaly_read_access` 允许从集群中的任何检测器查看结果，包括他们无法直接访问的数据。为了限制对检测器结果的访问，管理员应在创建检测器时使用后端角色过滤器。这样可以确保只有具有匹配后端角色的用户才能访问这些特定检测器的结果

