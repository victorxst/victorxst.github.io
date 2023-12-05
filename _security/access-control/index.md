---
layout: default
title: 访问控制
nav_order: 75
has_children: true
has_toc: false
redirect_from:
  - /security-plugin/access-control/index/
---

# 访问控制

您先请[配置安全插件]({{site.url}}{{site.baseurl}}/security/configuration/index/) 要使用自己的证书和首选的身份验证后端，您可以开始添加用户，创建角色并将角色映射到用户。

文档的这一部分涵盖了成功身份验证后允许用户看到和执行的操作。


## 概念

学期| 描述
：--- | ：---
允许| 单个行动，例如创建索引（例如`indices:admin/create`）。有关完整列表，请参阅[权限]({{site.url}}{{site.baseurl}}/security/access-control/permissions/)。
行动小组| 一组权限。例如，预定义`SEARCH` 行动小组授权角色使用`_search` 和`_msearch` 蜜蜂。
角色| 安全角色定义了权限或操作组的范围：集群，索引，文档或字段。例如，一个命名的角色`delivery_analyst` 可能没有集群权限，`READ` 所有匹配的索引的行动组`delivery-data-*` 模式，访问这些索引中的所有文档类型，以及访问所有字段以外的所有字段`delivery_driver_name`。
后端角色| （可选）您指定的任意字符串来自外部身份验证系统（例如，LDAP/Active Directory）。后端角色可以帮助简化角色映射过程。您可以将角色映射到所有100个用户共享的单个后端角色。
用户| 用户向OpenSearch群集提出请求。用户具有凭据（例如用户名和密码），零或更多后端角色以及零或更多自定义属性。
角色映射| 用户成功身份验证后扮演角色。角色映射将角色映射到用户（或后端角色）。例如，映射`kibana_user` （角色）`jdoe` （用户）表示约翰·杜（John Doe）获得了所有权限`kibana_user` 经过身份验证。同样，映射`all_access` （角色）`admin` （后端角色）意味着任何具有后端角色的用户`admin` 获得的所有权限`all_access` 经过身份验证。您可以将每个角色映射到多个用户和/或后端角色。

安全插件带有许多[预定义的动作组]({{site.url}}{{site.baseurl}}/security/access-control/default-action-groups/)，角色，映射和用户。这些实体是明智的默认值，是如何使用插件的好示例。

