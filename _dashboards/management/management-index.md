---
layout: default
title: 仪表板管理
nav_order: 100
has_children: true
---

# 仪表板管理
引入2.10
{: .label .label-purple }

仪表板管理是根据您的需求定制OpenSearch仪表板的指挥中心。接口的视图如下图所示。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/dashboards-management-ui.png" alt="Dashboards Management interface" width="700"/>

{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/icons/alert-icon.png" class ="inline-icon" alt ="alert icon"/> {:/}**笔记**<br> OpenSearch and OpenSearch仪表板特权管理对单个功能的访问。如果您没有适当的访问权限，请咨询您的管理员。
{:.note}

## 申请

仪表板管理可用以下应用程序：

- **[索引模式]({{site.url}}{{site.baseurl}}/dashboards/management/index-patterns/)：** 要访问OpenSearch数据，您需要创建一个索引模式，以便可以选择要使用的数据并定义字段的属性。索引模式工具使您能够从UI内部创建索引模式。索引模式指向一个或多个索引，数据流或索引别名。
- **[数据源]({{site.url}}{{site.baseurl}}/dashboards/management/multi-data-sources/)：** 数据源工具用于配置和管理OpenSearch用来收集和分析数据的数据源。您可以使用该工具在您的副本中指定源配置[OpenSearch仪表板配置文件]({{site.url}}{{site.baseurl}}https://github.com/opensearch-project/OpenSearch-Dashboards/blob/main/config/opensearch_dashboards.yml)。
- **保存的对象：** 保存的对象工具可帮助您组织和管理保存的对象。保存的对象是存储数据的文件，例如仪表板，可视化和地图，供以后使用。
- **[高级设置]({{site.url}}{{site.baseurl}}/dashboards/management/advanced-settings/)：** 高级设置工具使您可以灵活地个性化OpenSearch仪表板的行为。该工具分为设置部分，例如通用，可访问性和通知，您可以使用它来自定义和优化许多仪表板设置。

