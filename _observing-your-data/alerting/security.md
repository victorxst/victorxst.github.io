---
layout: default
title: 警告安全性
nav_order: 10
parent: 警报
has_children: false
redirect_from:
  - /monitoring-plugins/alerting/security/
---

# 警告安全性

如果您将安全插件与警报一起使用，则可能需要将某些用户限制在某些操作中。例如，您可能希望某些用户只能查看和确认警报，而另一些用户可以修改监视器和目的地。


## 基本权限

安全插件具有三个构建-在涵盖大多数警报用例的角色中：`alerting_read_access`，，，，`alerting_ack_alerts`， 和`alerting_full_access`。对于每个描述，请参阅[预定义的角色]({{site.url}}{{site.baseurl}}/security/access-control/users-roles#predefined-roles)。

如果这些角色无法满足您的需求，请混合并匹配个人警报[权限]({{site.url}}{{site.baseurl}}/security/access-control/permissions/) 适合您的用例。每个动作对应于REST API中的操作。例如，`cluster:admin/opensearch/alerting/destination/delete` 许可使您可以删除目的地。


## 监视数据如何访问数据

监视器运行的是创建或最后修改的用户的权限。例如，考虑用户`jdoe`，在零售商店连锁店工作。`jdoe` 有两个角色。这两个角色一起允许阅读三个索引：`store1-returns`，，，，`store2-returns`， 和`store3-returns`。

`jdoe` 每当三个索引的收益次数超过每小时40时，就可以创建一个监视器，该显示器将电子邮件发送给管理层。

后来，用户`psantos` 想编辑监视器每两个小时运行一次，但是`psantos` 只能访问`store1-returns`。为了改变，`psantos` 有两个选择：

- 更新监视器，以便仅检查`store1-returns`。
- 要求管理员阅读对其他两个索引的访问。

进行更改后，监视器现在以与`psantos`，包括任何[文档-级别的安全性]({{site.url}}{{site.baseurl}}/security/access-control/document-level-security/) 查询，[排除的字段]({{site.url}}{{site.baseurl}}/security/access-control/field-level-security/)， 和[蒙面字段]({{site.url}}{{site.baseurl}}/security/access-control/field-masking/)。如果您使用提取查询来定义监视器，请使用**跑步** 按钮以确保响应包括您需要的字段。

创建监视器后，即使创建监视器的用户已删除了其权限，也将继续执行监视器。只有具有正确集群权限的用户可以手动禁用或删除监视器以阻止其执行：

- 禁用显示器：`cluster:admin/opendistro/alerting/monitor/write`
- 删除监视器：`cluster:admin/opendistro/alerting/monitor/delete`

如果您的显示器的触发器已配置了通知，则不管目的地类型如何，警报插件继续发送通知。要停止通知，用户必须在触发操作中手动删除它们。

### 警报和罚款的注释-粒度访问控制

当触发器生成警报时，监视器配置，警报本身以及发送到通道的任何通知都可能包括描述要查询的索引的元数据。根据设计，插件必须提取数据并将其存储为索引外的元数据。[文档-级别的安全性]({{site.url}}{{site.baseurl}}/security/access-control/document-level-security) （DLS）和[场地-级别的安全性]({{site.url}}{{site.baseurl}}/security/access-control/field-level-security) （FLS）访问控件旨在保护索引中的数据。但是，一旦将数据存储在索引之外作为元数据，则具有访问监视器配置，警报及其通知的用户将能够查看此元数据，并可能推断索引中的数据内容和质量由DLS和FLS访问控制。

为了减少查看可以描述索引的元数据的意外用户的机会，我们建议管理员启用角色-在为预期的用户组分配权限时，基于访问控制并牢记这些设计元素。看[限制后端角色访问](#advanced-limit-access-by-backend-role) 有关详细信息。

## （高级）限制后端角色访问

开箱即用，警报插件没有所有权的概念。例如，如果您有`cluster:admin/opensearch/alerting/monitor/write` 许可，您可以编辑 *所有 *监视器，无论您是否创建它们。如果少数值得信赖的用户管理您的监视器和目的地，那么缺乏所有权通常并不是问题。较大的组织可能需要通过后端角色进行细分访问。

首先，确保您的用户有适当的[后端角色]({{site.url}}{{site.baseurl}}/security/access-control/index/)。后端角色通常来自[LDAP服务器]({{site.url}}{{site.baseurl}}/security/configuration/ldap/) 或者[SAML提供商]({{site.url}}{{site.baseurl}}/security/configuration/saml/)。但是，如果使用内部用户数据库，则可以使用REST API使用创建用户操作手动添加它们。要在创建用户请求中添加后端角色，请按照[创建用户]({{site.url}}{{site.baseurl}}/security/access-control/api#create-user) 安全插件API文档中的说明。

接下来，启用以下设置：

```json
PUT _cluster/settings
{
  "transient": {
    "plugins.alerting.filter_by_backend_roles": "true"
  }
}
```

现在，当用户查看OpenSearch仪表板中的警报（或进行REST API调用）中时，他们只会看到由共享 *至少一个 *后端角色的用户创建的显示器和目的地。例如，考虑三个都可以完全访问警报的用户：`jdoe`，，，，`jroe`， 和`psantos`。

`jdoe` 和`jroe` 在同一团队中工作，都有`analyst` 后端角色。`psantos` 有`human-resources` 后端角色。

如果`jdoe` 创建一个显示器，`jroe` 可以看到并修改它，但是`psantos` 不能。如果该监视器生成警报，则情况相同：`jroe` 可以看到并承认，但是`psantos` 不能。如果`psantos` 创建目的地，`jdoe` 和`jroe` 看不到或修改它。

<！-- ## （高级）限制个人访问

如果您只希望用户能够查看和修改自己的监视器和目的地，请重复`alerting_full_access` 角色并添加以下内容[DLS查询]({{site.url}}{{site.baseurl}}/security/access-control/document-level-security/) 对此：

```json
{
  "bool": {
    "should": [{
      "match": {
        "monitor.created_by": "${user.name}"
      }
    }, {
      "match": {
        "destination.created_by": "${user.name}"
      }
    }]
  }
}
```

然后，为所有警报用户使用此新角色。-->

### 指定RBAC后端角色

您可以指定角色-使用警报API创建或更新监视器时，基于访问控制（RBAC）后端角色。

在创建监视器方案中，请遵循以下准则指定角色：

用户类型| 角色是由用户指定的（Y/N）| 如何使用RBAC角色
：--- | ：--- | ：---
管理用户| 是的| 使用所有指定的后端角色与监视器关联。
常规用户| 是的| 使用用户有权与监视器关联的后端角色列表中的所有指定的后端角色。
常规用户| 不| 复制用户的后端角色，并将其关联到显示器。

在更新监视器方案中，请遵循以下指南以指定角色：

用户类型| 角色是由用户指定的（Y/N）| 如何使用RBAC角色
：--- | ：--- | ：---
管理用户| 是的| 删除与监视器关联的所有后端角色，然后使用与监视器关联的所有指定的后端角色。
常规用户| 是的| 删除用户可以访问但未指定的监视器关联的后端角色。然后从用户有权使用的后端角色列表中添加所有其他指定的后端角色。
常规用户| 不| 不要更新监视器上的后端角色。

- 对于管理员用户，一个空列表被认为与删除用户拥有的所有权限相同。如果是非-管理员用户通过一个空列表中的传递，这会引发异常，因为非-管理用户。
- 如果用户试图关联他们没有权限使用的角色，则会引发例外。
{： 。笔记 }

要创建RBAC角色，请按照安全插件API文档中的说明进行操作[创建角色]({{site.url}}{{site.baseurl}}/security/access-control/api#create-role)。
### 创建带有RBAC角色的监视器

使用警报API创建监视器时，可以指定请求主体底部的RBAC角色。使用`rbac_roles` 范围。

以下样本显示了RBAC参数指定的RBAC角色：

```json
... 
  "rbac_roles": ["role1", "role2"]
```

要查看完整的请求示例，请参阅[创建一个查询-级别监视器]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/api/#create-a-query-level-monitor)。


