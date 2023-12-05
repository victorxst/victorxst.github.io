---
layout: default
title: 用于安全分析的 OpenSearch Security
nav_order: 2
has_children: false
---

# 用于安全分析的 OpenSearch Security

您可以使用Security Analytics使用OpenSearch Security来分配用户权限，并管理用户可以和无法执行的操作。例如，您可能希望一组用户能够创建，更新或删除检测器，而另一组用户只能查看检测器。您可能还希望另一个小组能够接收和确认警报，但要阻止执行其他任务。OpenSearch安全框架使您可以控制用户对安全分析功能的访问级别。

---
## Security Analytics system indexes

安全分析索引被保护为系统索引，对群集中的其他索引的处理方式不同。系统索引存储配置和其他系统设置，因此，无法使用REST API或OpenSearch仪表板接口进行修改。只有具有TLS [admin证书]的用户({{site.url}}) {{site.baseurl}}/security/configuration/tls/tls/tls/tls/tls/tls/tls/#配置-行政-证书）可以访问系统索引。有关使用此类型索引的更多信息，请参见[System Indexes]({{site.url}}) {{site.baseurl}}/security/configuration/configuration/system-指数/）。

---
## 基本权限

作为管理员，您可以使用OpenSearch仪表板或安全性REST API来根据所需访问的特定API为用户分配特定的权限。有关支持API的列表，请参阅[API工具]({{site.url}}{{site.baseurl}}/security-analytics/api-tools/index/)。

OpenSearch Security已建立了三个-在涵盖大多数安全分析用例的角色中：`security_analytics_full_access`，，，，`security_analytics_read_access`， 和`security_analytics_ack_alerts`。有关这些角色和其他角色的描述，请参阅[预定义的角色]({{site.url}}{{site.baseurl}}/security/access-control/users-roles#predefined-roles)。

如果这些角色无法满足您的需求，请混合并匹配个体安全分析[权限]({{site.url}}{{site.baseurl}}/security/access-control/permissions/#security-analytics-permissions) 适合您的用例。每个动作对应于REST API中的操作。例如，`cluster:admin/opensearch/securityanalytics/detector/delete` 许可使您可以删除检测器。

---
## (Advanced) Limit access by backend role

您可以使用后端角色来配置罚款-基于角色的粒状访问单个探测器。例如，可以将后端角色分配给在组织的不同部门工作的用户，以便他们只能查看其工作部门拥有的那些检测器。

首先，确保您的用户具有适当的[后端角色]（{{site.url}}} {{site.baseurl}}/security/access-控制/索引/）。后端角色通常来自[ldap Server]（{{site.url}}} {{site.baseurl}}/security/configuration/ldap/）或[saml provider]（{site.url}}} {{site。baseurl}}/security/configuration/saml/）。但是，如果您使用内部用户数据库，则可以使用REST API进行[手动添加它们]（{{site.url}} {site.baseurl}}/security/security/access-控制/API#创造-用户）。

接下来，启用以下设置：

```json
put /_ cluster /设置
{
  "transient"：{
    "plugins.security_analytics.filter_by_backend_roles"："true"
  }
}
```
{% include copy-curl.html %}

现在，当用户在OpenSearch仪表板中查看Security Analytics资源（或进行REST API调用）时，他们只会看到由至少共享一个后端角色的用户创建的检测器。
例如，考虑两个用户：`alice` 和`bob`。

以下示例分配用户`alice` 这`analyst` 后端角色：

```json
put/_plugins/_security/api/internalusers/alice
{
  "password"："alice"，，，，
  "backend_roles"：[[
    "analyst"
  ]，，
  "attributes"：{}
}
```
{% include copy-curl.html %}

下一个示例分配用户`bob` 这`human-resources` 后端角色：

```json
put/_plugins/_ security/api/internestusers/bob
{
  "password"："bob"，，，，
  "backend_roles"：[[
    "human-resources"
  ]，，
  "attributes"：{}
}
```
{% include copy-curl.html %}

最后，最后一个示例分配了`alice` 和`bob` 使他们完全访问安全分析的角色：

```json
put/_plugins/_security/api/rolesmapping/security_analytics_full_access
{
  "backend_roles"：[]，，
  "hosts"：[]，，
  "users"：[[
    "alice"，，，，
    "bob"
  这是给出的
}
```
{% include copy-curl.html %}

但是，由于它们具有不同的后端角色，所以`alice` 和`bob` 无法查看彼此的探测器或结果。

---
## 关于使用罚款的注释-使用插件的粒度访问控制

当触发器生成警报时，检测器配置，警报本身以及发送到通道的任何通知都可能包括描述要查询的索引的元数据。根据设计，插件必须提取数据并将其存储为索引外的元数据。[文档-级别的安全性]({{site.url}}{{site.baseurl}}/security/access-control/document-level-security) （DLS）和[场地-级别的安全性]({{site.url}}{{site.baseurl}}/security/access-control/field-level-security) （FLS）访问控件旨在保护索引中的数据。但是，一旦将数据存储在索引外作为元数据之外，可以访问检测器和监视配置，警报及其通知的用户将能够查看此元数据，并可能推断索引中数据的内容和质量，否则，这将是被DLS和FLS访问控制所隐藏。

为了减少查看可以描述索引的元数据的意外用户的机会，我们建议管理员启用角色-在为预期的用户组分配权限时，基于访问控制并牢记这些设计元素。看[限制后端角色访问](#advanced-limit-access-by-backend-role) 了解更多信息。

