---
layout: default
title: 索引管理安全
nav_order: 40
has_children: false
---

# 索引管理安全性

将安全插件与索引管理结合使用，可以将非管理员用户限制为某些操作。例如，你可能希望设置安全性，以便一组用户只能读取 ISM 策略，而其他用户可以创建、删除或更改策略。

所有索引管理数据都作为系统索引进行保护，只有超级管理员或具有传输层安全性（TLS）证书的管理员才能访问系统索引。有关详细信息，请参阅[系统索引]({{site.url}}{{site.baseurl}}/security/configuration/system-indices)。

## 基本权限

Security 插件附带一个角色，该角色提供对索引管理的完全访问权限： `index_management_full_access`.有关角色权限的说明，请参见[预定义角色]({{site.url}}{{site.baseurl}}/security/access-control/users-roles#predefined-roles)。

启用安全性后，用户不仅需要正确的索引管理权限，还需要对涉及的索引执行操作的权限。例如，如果用户想要使用 REST API 将执行汇总作业的策略附加到名为 `system-logs` 的索引，则需要附加策略和执行汇总作业的权限，以及对 `system-logs` 的访问权限。

最后，除了“创建策略”、“获取策略”和“删除策略”之外，用户还需要 `indices:admin/opensearch/ism/managedindex` 执行[ISM APIs]({{site.url}}{{site.baseurl}}/im-plugin/ism/api).

## （高级）按后端角色限制访问权限

你可以使用后端角色来配置对索引管理策略和操作的精细访问。例如，组织中不同部门的用户可能会根据分配给他们的角色和权限查看不同的策略。

首先，确保你的用户具有适当的[后端角色]({{site.url}}{{site.baseurl}}/security/access-control/index/).后端角色通常来自[LDAP 服务器]({{site.url}}{{site.baseurl}}/security/configuration/ldap/)或[SAML 提供程序]({{site.url}}{{site.baseurl}}/security/configuration/saml/)。但是，如果你使用内部用户数据库，则可以使用 REST API 执行以下操作[手动添加它们]({{site.url}}{{site.baseurl}}/security/access-control/api#create-user)。

使用 REST API 启用以下设置：

```json
PUT _cluster/settings
{
  "transient": {
    "plugins.index_management.filter_by_backend_roles": "true"
  }
}
```

启用安全性后，只有共享至少一个后端角色的用户才能查看和执行与其角色相关的策略和操作。

例如，考虑具有三个用户的方案： `John` 和 `Jill`，他们具有后端角色 `helpdesk_staff`，以及 `Jane`，谁具有后端角色 `phone_operator`。 `John` 想要创建一个策略，用于在名为 `airline_data` 的索引上执行汇总作业，因此 `John` 需要一个后端角色，该角色有权访问该索引、创建相关策略和执行相关操作，并且 `Jill` 能够访问同一索引，政策和工作。但是， `Jane` 无法访问或编辑这些资源或操作。
