---
layout: default
title: OpenSearch 仪表板多租户
nav_order: 140
has_children: true
has_toc: false
redirect_from:
  - /security/multi-tenancy/
---

# OpenSearch 仪表板多租户

*租户*在OpenSearch仪表板中是用于保存索引模式，可视化，仪表板和其他OpenSearch仪表板对象的空间。OpenSearch允许用户创建多个用途的多个租户。租户对于与其他OpenSearch仪表板用户安全共享您的工作很有用。您可以控制哪些角色可以访问租户，以及这些角色是否已读取或写入访问权限。默认情况下，所有OpenSearch仪表板用户都可以访问两个独立的租户：全球租户和一个私人租户。多-租赁还提供了创建自定义租户的选项。

- **全球的** -- 每个OpenSearch仪表板用户之间共享此租户。它确实允许在可以访问该对象的用户之间共享对象。
- **私人的** -- 该租户是每个用户独有的，无法共享。它不允许您访问用户全局租户创建的路由或索引模式。
- **风俗** -- 管理员可以创建自定义租户并将其分配给特定角色。一旦创建，这些租户就可以为特定用户组提供空间。

从某种意义上说，全球租户不是 *主要 *租户，它在私人租户中复制其内容。相反，如果您对全球租户进行更改，您将不会看到您的私人租户发生的变化。一些示例更改包括以下内容：

- 更改高级设置
- 创建可视化
- 创建索引模式

为了提供一个实用的例子，您可能会使用私人租户进行探索性工作，并在您的团队中创建详细的可视化`analysts` 租户，并在公司领导中维护公司领导板的摘要仪表板`executive` 租户。

如果您与某人共享可视化或仪表板，则可以看到URL包括租户：

```
http://<opensearch_dashboards_host>:5601/app/opensearch-dashboards?security_tenant=analysts#/visualize/edit/c501fa50-7e52-11e9-ae4e-b5d69947d32e?_g=()
```

## 下一步

要开始租户，请参阅[多-租赁配置]({{site.url}}{{site.baseurl}}/security/multi-tenancy/multi-tenancy-config/) 有关启用Multi的信息-租赁，增加租户，并为租户分配角色。

有关对多人进行动态更改的信息-租赁配置，请参阅[OpenSearch仪表板中的动态配置]({{site.url}}{{site.baseurl}}/security/multi-tenancy/dynamic-config/)。


