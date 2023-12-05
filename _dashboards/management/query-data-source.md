---
layout: default
title: 查询和可视化Amazon S3数据
parent: 将 Amazon S3 连接到 OpenSearch
grand_parent: 数据源
nav_order: 10
has_children: false
---

# 查询和可视化Amazon S3数据
引入2.11
{: .label .label-purple }

该教程指导您使用**查询数据** 使用OpenSearch仪表板查询和可视化Amazon简单存储服务（Amazon S3）数据的用例。

## 先决条件

您必须使用`opensearch-security` 插件并具有适当的角色权限。请与您的IT管理员联系以分配必要的权限。

## 开始查询

要开始，请按照以下步骤：

1. 在**管理数据源** 页面，从列表中选择您的数据源。
2. 在数据源的详细信息页面上，选择**查询数据** 卡片。此选项将您带到**可观察性** >**日志** 页面，如下图所示。

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/observability-logs-UI.png" alt ="Observability Logs UI" 宽度="700">

3. 选择**活动资源管理器** 按钮。此选项可创建并保存经常搜索的查询和可视化[管道处理语言（PPL）]({{site.url}}{{site.baseurl}}/search-plugins/sql/ppl/index/) 或者[SQL]({{site.url}}{{site.baseurl}}/search-plugins/sql/index/)，连接到Spark SQL。
4. 从上拉菜单中的下拉菜单中选择Amazon S3数据源-左角。下图显示了一个示例。

     <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/query-data-sources-UI-2.png" alt ="Observability Logs Amazon S3 dropdown menu" 宽度="700">

5. 输入查询**输入PPL查询** 场地。请注意，默认语言是SQL。要更改语言，请从下拉菜单中选择PPL。
6. 选择**搜索** 按钮。这**查询处理** 显示消息，确认您的查询正在处理。
7. 查看结果，这些结果在**事件** 标签。在此页面上，详细信息（诸如可用字段，源和时间）以表格格式显示。
8. （可选）创建数据可视化。

## 创建亚马逊S3数据的可视化

要创建可视化，请执行以下步骤：

1. 在**资源管理器** 页面，选择**可视化** 标签。下图显示了一个示例。

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/explorer-S3viz-UI.png" alt ="Explorer Amazon S3 visualizations UI" 宽度="700">

2. 选择**索引数据可视化**。此选项当前仅创建[加速索引]({{site.url}}{{site.baseurl}}/dashboards/management/accelerate-external-data/)，从**可视化** 标签。要创建Amazon S3数据的可视化，请转到**发现**。看到[发现文档]({{site.url}}{{site.baseurl}}/dashboards/discover/index-discover/) 有关信息和教程。

## 使用Amazon S3数据源使用查询工作台

[查询工作台]({{site.url}}{{site.baseurl}}/search-plugins/sql/workbench/) 运行-需求SQL查询，将SQL转换为其休息等效物，并将结果保存为文本，JSON，JDBC或CSV。

要与您的Amazon S3数据一起使用查询工作台，请按照以下步骤：

1. 在OpenSearch仪表板主菜单中，选择**OpenSearch插件** >**查询工作台**。
2. 来自**数据源** 上下拉菜单-左角，选择您的Amazon S3数据源。您的数据开始加载数据源的一部分数据库。下图显示了一个示例。

     <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/query-workbench-S3.png" alt ="Query Workbench Amazon S3 data loading UI" 宽度="700">

3. 查看左侧列出的数据库-侧导航菜单，然后选择一个数据库以查看其详细信息。有关加速索引的任何信息列出了**加速索引目的地**。
4. 选择**描述索引** 按钮以了解有关如何将数据存储在该特定索引中的更多信息。
5. 选择**下降索引** 按钮删除和清除OpenSearch索引和刷新数据的Amazon S3 Spark作业。
6. 输入您的SQL查询并选择**跑步**。
## 下一步

- 学习关于[加速您的外部数据源的查询性能]({{site.url}}{{site.baseurl}}/dashboards/management/accelerate-external-data/)。

