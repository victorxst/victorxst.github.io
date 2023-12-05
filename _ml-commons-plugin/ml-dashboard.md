---
layout: default
title: 在OpenSearch仪表板中管理ML模型
nav_order: 120
redirect_from:
  - /ml-commons-plugin/ml-dashbaord/
---

ML仪表板被从实验状态中取出，并在OpenSearch 2.9中通常可用。
{: .note}

机器学习（ML）群集的管理员可以使用OpenSearch仪表板来管理和检查集群中运行的ML模型的状态。这可以帮助ML开发人员提供节点，以确保其模型有效运行。

您只能使用API注册和部署模型。有关更多信息，请参阅[在OpenSearch中使用ML模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/model-serving-framework/)。

## 在仪表板中启用ML

在OpenSearch 2.6中，默认情况下禁用ML功能。要启用它，您需要在`opensearch_dashboards.yml` 然后重新启动您的群集。

启用该功能：

1. 在您的OpenSearch集群中，导航到仪表板主目录；例如，在Docker中，`/usr/share/opensearch-dashboards`。
2. 打开仪表板配置文件的本地副本`opensearch_dashboards.yml`。如果您没有副本，请从Github获取一份：[`opensearch_dashboards.yml`](https://github.com/opensearch-project/OpenSearch-Dashboards/blob/main/config/opensearch_dashboards.yml)。
3. 添加设置`ml_commons_dashboards.enabled:`  到`opensearch_dashboards.yml`。然后，将其设置为`ml_commons_dashboards.enabled: true` 并保存配置文件。
4. 重新启动仪表板容器。
5. 通过启动OpenSearch仪表板来验证功能配置设置是创建并正确配置的。机器学习部分应在**OpenSearch插件**。

## 访问仪表板中的ML功能

要访问OpenSearch仪表板中的ML功能，请选择**OpenSearch插件** >**机器学习**。

<img src="{{site.url}}{{site.baseurl}}/images/ml/ml-dashboard/ml-dashboard.png" alt="Machine Learning section in OpenSearch dashboards">

在机器学习部分中，您现在可以访问**部署的模型** 仪表板。

## 部署的型号仪表板

部署的型号仪表板使Admins能够检查OpenSearch集群中存储的任何型号的状态。

<img src="{{site.url}}{{site.baseurl}}/images/ml/ml-dashboard/deployed-models.png" alt="The deployed models view.">

仪表板包括有关该模型的以下信息：

- **姓名**：上传时给出的模型名称。
- **地位**：模型响应的节点数量。
   - 当所有节点响应迅速时，状态为**绿色的**。
   - 当某些节点响应迅速时，状态为**黄色的**。
   - 当所有节点无反应时，状态为**红色的**。
- **模型ID**：模型ID。
- **行动**：您可以采取哪些动作。

从OpenSearch 2.6开始，唯一可用的操作是**查看状态详细信息**，如下图所示。

<img src="{{site.url}}{{site.baseurl}}/images/ml/ml-dashboard/view-status-details.png" alt="You can view status details under actions.">

选择时，将出现状态详细信息面板。

该面板在面板内提供以下详细信息：

- **模型ID**
- **模型状态通过节点**：该模型响应的节点数量。

一个节点列表可让您视图模型正在运行的每个节点的视图，包括每个节点**节点ID** 和状态，如下图所示。如果您想使用节点的**节点ID** 确定为什么节点没有反应。

<img src="{{site.url}}{{site.baseurl}}/images/ml/ml-dashboard/model-node-details.png" alt="The status of each node running the model.">

## 下一步

有关如何在OpenSearch中管理ML模型的更多信息，请参见[在OpenSearch中使用ML模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/model-serving-framework/)。

