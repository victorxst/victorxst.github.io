---
layout: default
title: 已保存对象的多租户聚合视图
parent: OpenSearch 仪表板多租户
nav_order: 150
---

# OpenSearch仪表板多-保存对象的租赁汇总视图

这是OpenSearch 2.4中发布的实验功能，不建议在生产环境中使用。有关功能进度或要留下反馈的更新，请参阅[仪表板对象共享](https://github.com/opensearch-project/OpenSearch-Dashboards/issues/2249) Github问题。为了更全面地看待拟议的Multi的未来发展-租赁，请参阅[仪表板对象共享](https://github.com/opensearch-project/security/issues/1869) 问题。
{： 。警告}

保存对象的汇总视图允许可以访问多个租户的用户在一个视图中查看与这些租户关联的所有保存对象，而无需在租户之间切换。这包括用户创建的租户和与用户共享的租户。聚合视图在保存的对象表中引入了租户下拉菜单和列，该表使用户可以通过租户过滤并使其可见其关联的保存对象进行过滤。

一旦确定了保存的感兴趣对象，就可以切换到该租户与对象一起使用。

要访问保存的对象，请展开顶部菜单，然后选择**管理>仪表板管理>保存对象**。保存的对象窗口打开。默认情况下，用户拥有权限的所有租户都与与租户关联的所有保存对象一起显示。

作为实验功能，保存对象的聚合视图保留在功能标志后面，必须在`opensearch_dashboards.yml` 在提供该功能之前的文件。看[启用聚合视图](#enabling-aggregate-view-for-saved-objects) 了解更多信息。
{： 。笔记 }

### 功能优势

- 在一个屏幕上实现所有保存对象的汇总视图，您可以快速找到感兴趣的对象，并确定哪个租户与之相关。找到对象后，您可以选择适当的租户并与对象一起使用。
- 此功能还将租户下拉菜单添加到保存的对象表中，该菜单使您可以通过租户及其关联的保存对象过滤视图。

### 未来发展的计划

在随后的版本中，我们计划扩展此功能的功能，以包括直接从聚合视图执行操作并共享项目的能力，而无需首先选择特定的租户。从长远来看，OpenSearch计划进化多个-租赁使其成为在用户之间共享对象的更灵活的工具，并采用了一种更复杂的方式来分配有助于共享的角色和权限。要了解有关未来版本提出的功能的更多信息，请参阅GitHub问题[仪表板对象共享](https://github.com/opensearch-project/security/issues/1869)。

### 已知限制

在开发的第一个实验阶段，在启用该功能并在测试环境中使用它之前，应有一些限制：

*该功能只能在新集群中使用。目前，该功能不受已经使用的群集的支持。
*此外，该功能仅应在测试环境中而不是在生产中使用。
*最后，一旦启用了该功能并将其用于测试群集，就无法为群集禁用该功能。一旦使用该功能与租户合作并保存对象可能会导致损失保存的物体，并且可能会对租户产生影响-到-租户功能。当以三种方式中的任何一种禁用该功能时，可能会发生这种情况：使用[功能标志](#enabling-aggregate-view-for-saved-objects);禁用多重-传统的租赁[多-租赁配置]({{site.url}}{{site.baseurl}}/security/multi-tenancy/multi-tenancy-config/) 环境;或禁用多重-租赁[动态配置]({{site.url}}{{site.baseurl}}/security/multi-tenancy/dynamic-config/) 设置。

这些限制将在即将发布的版本中解决。

## 启用保存对象的汇总视图

默认情况下，禁用了保存对象表中的聚合视图。要启用该功能，请添加`opensearch_security.multitenancy.enable_aggregation_view` 标记到`opensearch_dashboards.yml` 提交并将其设置为`true`：

`opensearch_security.multitenancy.enable_aggregation_view: true`

启用功能后，您可以启动新集群，然后启动仪表板。

## 总体观点工作

选择**租户** 下拉箭头显示可用的租户列表。您可以在菜单打开时选择多个租户。每次您在菜单中选择一个租户时，保存对象的列表都会由该租户和其他任何其他租户都在其名称旁边有复选标记进行过滤。

<img src="{{site.url}}{{site.baseurl}}/images/Security/Tenant_column.png" alt="Dashboards Saved Objects view with emphasis on Tenants column" width="500">
   
完成指定租户后，请选择菜单外的任何地方崩溃。
*标题列显示可用保存的对象的名称。
*租户列显示与已保存对象相关的租户。
*此外，在租户下拉菜单标签旁边的一个红色框中显示了选择过滤的租户数量。

<img src="{{site.url}}{{site.baseurl}}/images/Security/ten-filter-results.png" alt="Dashboards Saved Objects tenant filtering" width="700">

使用**类型** 下拉菜单以按类型过滤保存的对象。行为**类型** 下拉菜单与行为相同**租户** 下拉式菜单。

### 选择并使用保存的对象

确定要使用的保存对象后，请按照以下步骤访问该对象：

1. 注意与租户列中对象相关的租户。
1. 在上部-窗口的右角，打开用户菜单，然后选择**切换租户**。
<br> <img src ="{{site.url}}{{site.baseurl}}/images/Security/switch_tenant.png" alt ="Switching tenants in the user menu" 宽度="425">
1. 在里面**选择您的租户** 窗口，选择全局或私人选项或自定义租户选项之一，以指定正确的租户。选择**确认** 按钮。租户变得活跃，并显示在用户菜单中。
1. 租户处于活动状态后，您可以使用“操作”列中的控件与与租户关联的保存对象一起使用。
<img src="{{site.url}}{{site.baseurl}}/images/Security/actions.png" alt="Actions column controls" width="700">

当租户不活跃时，您无法使用操作列控件与其关联对象一起使用。要与这些物体一起工作，请按照上述步骤使租户活跃起来。
{： 。笔记 }


