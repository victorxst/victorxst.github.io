---
layout: default
title: 将Amazon S3连接到OpenSearch
parent: 数据源
nav_order: 15
has_children: true
---

# 将Amazon S3连接到OpenSearch
引入2.11
{: .label .label-purple }

从OpenSearch 2.11开始，您可以使用OpenSearch Dashboards UI连接OpenSearch与Amazon Simple Storage Service（Amazon S3）数据源。然后，您可以查询该数据，优化查询性能，定义表并将S3数据集成到单个UI中。
从OpenSearch 2.11开始，您可以使用OpenSearch仪表板用户界面（UI）将OpenSearch连接到Amazon S3数据源。然后，您可以查询该数据，优化查询性能，定义表并从单个UI集成S3数据。

## 先决条件

要使用OpenSearch仪表板将来自Amazon S3的数据连接到OpenSearch，您必须有：

- 访问Amazon S3和[AWS胶水数据目录](https://github.com/opensearch-project/sql/blob/main/docs/user/ppl/admin/connectors/s3glue_connector.rst#id2)。
- 访问OpenSearch和OpenSearch仪表板。
- 对OpenSearch数据源和连接器概念的理解。看到[开发人员文档](https://github.com/opensearch-project/sql/blob/main/docs/user/ppl/admin/datasources.rst#introduction) 有关这些概念的信息。

## 连接您的Amazon S3数据源

要连接您的Amazon S3数据源，请执行以下步骤：

1. 在OpenSearch仪表板主菜单中，选择**管理** >**数据源**。
2. 在**数据源** 页面，选择**新数据源** >**S3**。下图显示了一个示例UI。

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/data-sources-UI.png" alt ="Amazon S3 data sources UI" 宽度="700"/>

3. 在**配置Amazon S3数据源** 页面，输入所需的**数据源详细信息**，，，，**AWS胶认证细节**，，，，**AWS GLUE索引商店详细信息**， 和**查询权限**。下图显示了一个示例UI。

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/S3-config-UI.png" alt ="Amazon S3 configuration UI" 宽度="700"/>

4. 选择**查看配置** 按钮并验证详细信息。
5. 选择**连接到Amazon S3** 按钮。

## 管理您的Amazon S3数据源

连接Amazon S3数据源后，您可以通过**管理数据源** 标签。以下步骤指导您使用此功能：

1. 在**管理数据源** 选项卡，从列表中选择一个日期源。
2. 在该数据源的页面上，您可以管理数据源，选择用例并管理访问控件和配置。下图显示了一个示例UI。

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/manage-data-source-UI.png" alt ="Manage data sources UI" 宽度="700"/>

3. （可选）探索Amazon S3用例，包括查询数据并优化查询性能。去**下一步** 要了解有关每种用例的更多信息。

## 限制

此功能仍在开发中，包括数据集成功能。真正的-时间更新，请参阅[GitHub上的开发人员文档](https://github.com/opensearch-project/opensearch-spark/blob/main/docs/index.md#limitations)。

## 下一步

- 学习关于[在数据资源管理器中查询您的数据]({{site.url}}{{site.baseurl}}/dashboards/management/query-data-source/) 通过OpenSearch仪表板。
- 了解方法[优化外部数据源的查询性能]({{site.url}}{{site.baseurl}}/dashboards/management/accelerate-external-data/)，例如Amazon S3，通过查询工作台。
- 学习关于[Amazon S3和AWS胶水数据目录](https://github.com/opensearch-project/sql/blob/main/docs/user/ppl/admin/connectors/s3glue_connector.rst) 以及与Amazon S3数据源一起使用的API，包括配置设置和查询示例。
- 学习关于[管理您的索引]({{site.url}}{{site.baseurl}}/dashboards/im-dashboards/index/) 通过OpenSearch仪表板。
  

