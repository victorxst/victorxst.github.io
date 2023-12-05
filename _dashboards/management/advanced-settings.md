---
layout: default
title: 高级设置
parent: 仪表板管理
nav_order: 40
---

# 高级设置
更新2.10
{: .label .label-purple }

使用**高级设置** 页面以修改控制OpenSearch仪表板行为的设置。这些设置可用于自定义应用程序的外观，更改某些功能的行为等等。接口的视图如下图所示。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/advanced-settings-ui.png" alt="Advanced settings interface" width="700"/>

访问**高级设置**， 去**仪表板管理** 并选择**高级设置**。该页面分为几个部分，每个部分包含一组相关设置。您可以通过编辑其字段来修改这些设置。进行更改后，选择**节省** 应用它们。

{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/icons/alert-icon.png" class ="inline-icon" alt ="alert icon"/> {:/}**笔记**<br>某些设置要求您修改[这`opensearch_dashboards.yml` 文件](https://github.com/opensearch-project/OpenSearch-Dashboards/blob/main/config/opensearch_dashboards.yml) 并重新启动OpenSearch仪表板。
{: .note}

## 所需的权限

要修改设置，您必须有权进行更改。看[并发-租赁配置](https://opensearch.org/docs/latest/security/multi-tenancy/multi-tenancy-config/#give-roles-access-to-tenants) 有关向租户分配角色访问的指导。

