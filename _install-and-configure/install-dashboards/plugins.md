---
layout: default
title: 管理 OpenSearch 控制面板插件
nav_order: 100
redirect_from: 
  - /dashboards/install/plugins/
  - /install-and-configure/install-dashboards/plugins/
---

# 管理 OpenSearch 控制面板插件

OpenSearch 控制面板提供了一个命令行工具，用于 `opensearch-plugin` 管理插件。此工具允许你：

- 列出已安装的插件。
- 安装插件。
- 删除已安装的插件。

## 插件兼容性

主要、次要和补丁插件版本必须与 OpenSearch 主要版本、次要版本和补丁版本匹配才能兼容。例如，插件版本 2.3.0.x 仅适用于 OpenSearch 2.3.0. {：.warning}

## 先决条件

- 兼容的 OpenSearch 集群
- 相应的 OpenSearch 插件[安装在该集群上]({{site.url}}{{site.baseurl}}/opensearch/install/plugins/)
- 的相应版本[OpenSearch 控制面板]({{site.url}}{{site.baseurl}}/)（例如，OpenSearch 控制面板 2.3.0 可与 OpenSearch 2.3.0 配合使用）

## 可用的插件

下表列出了可用的 OpenSearch 控制面板插件。

| Plugin Name | Repository |最早的可用版本|
| :--- | :--- | :--- |
| Alerting Dashboards |[alerting-dashboards-plugin](https://github.com/opensearch-project/alerting-dashboards-plugin)| 1.0.0 |
| Anomaly Detection Dashboards |[异常检测仪表板插件](https://github.com/opensearch-project/anomaly-detection-dashboards-plugin)| 1.0.0 |
| Custom Import Maps Dashboards |[仪表板-地图](https://github.com/opensearch-project/dashboards-maps)| 2.2.0 |
| Search Relevance Dashboards |[仪表板-搜索-相关性](https://github.com/opensearch-project/dashboards-search-relevance)| 2.4.0 |
| Gantt Chart Dashboards |[甘特图](https://github.com/opensearch-project/dashboards-visualizations/tree/main/gantt-chart)| 1.0.0 |
| Index Management Dashboards |[index-management-dashboards-plugin](https://github.com/opensearch-project/index-management-dashboards-plugin)| 1.0.0 |
| Notebooks Dashboards |[仪表板-笔记本](https://github.com/opensearch-project/dashboards-notebooks)| 1.0.0 |
| Notifications Dashboards |[仪表板-通知](https://github.com/opensearch-project/dashboards-notifications)| 2.0.0 |
| Observability Dashboards |[仪表板 - 可观测性](https://github.com/opensearch-project/dashboards-observability)| 2.0.0 |
| Query Workbench Dashboards |[查询工作台](https://github.com/opensearch-project/dashboards-query-workbench)| 1.0.0 |
| Reports Dashboards |[仪表板报告](https://github.com/opensearch-project/dashboards-reporting)| 1.0.0 |
| Security Analytics Dashboards |[安全-分析-仪表板-插件](https://github.com/opensearch-project/security-analytics-dashboards-plugin)| 2.4.0 |
| Security Dashboards |[安全仪表板插件](https://github.com/opensearch-project/security-dashboards-plugin)| 1.0.0 |

## 安装

导航到 OpenSearch 控制面板主目录（例如， `/usr/share/opensearch-dashboards`），然后为每个插件运行 install 命令。

{% comment %}

#### 安全性 OpenSearch 控制面板

```bash
sudo bin/opensearch-dashboards-plugin install https://d3g5vo6xdbdb9a.cloudfront.net/downloads/opensearch-dashboards-plugins/opensearch-security/opensearchSecurityOpenSearch Dashboards-{{site.opensearch_major_minor_version}}.0.1.zip
```

此插件提供用于管理用户、角色、映射、操作组和租户的用户界面。

#### 向 OpenSearch 控制面板发出警报

```bash
sudo bin/opensearch-dashboards-plugin install https://d3g5vo6xdbdb9a.cloudfront.net/downloads/opensearch-dashboards-plugins/opensearch-alerting/opensearchAlertingOpenSearch Dashboards-{{site.opensearch_major_minor_version}}.0.0.zip
```

此插件提供了用于创建监视器和管理警报的用户界面。

#### 索引状态管理 OpenSearch 控制面板

```bash
sudo bin/opensearch-dashboards-plugin install https://d3g5vo6xdbdb9a.cloudfront.net/downloads/opensearch-dashboards-plugins/opensearch-index-management/opensearchIndexManagementOpenSearch Dashboards-{{site.opensearch_major_minor_version}}.0.1.zip
```

此插件提供用于管理策略的用户界面。

#### 异常检测 OpenSearch 控制面板

```bash
sudo bin/opensearch-dashboards-plugin install https://d3g5vo6xdbdb9a.cloudfront.net/downloads/opensearch-dashboards-plugins/opensearch-anomaly-detection/opensearchAnomalyDetectionOpenSearch Dashboards-{{site.opensearch_major_minor_version}}.0.0.zip
```

该插件提供了用于添加检测器的用户界面。

#### Query Workbench OpenSearch 控制面板

```bash
sudo bin/opensearch-dashboards-plugin install https://d3g5vo6xdbdb9a.cloudfront.net/downloads/opensearch-dashboards-plugins/opensearch-query-workbench/opensearchQueryWorkbenchOpenSearch Dashboards-{{site.opensearch_major_minor_version}}.0.0.zip
```

此插件提供了一个用户界面，用于使用 SQL 查询来浏览数据。

#### 跟踪分析

```bash
sudo bin/opensearch-dashboards-plugin install https://d3g5vo6xdbdb9a.cloudfront.net/downloads/opensearch-dashboards-plugins/opensearch-trace-analytics/opensearchTraceAnalyticsOpenSearch Dashboards-{{site.opensearch_major_minor_version}}.2.0.zip
```

此插件使用分布式跟踪数据（使用 Data Prepper 在 OpenSearch 中编制索引）来显示延迟趋势、错误率等。

#### 笔记本 OpenSearch 控制面板

```bash
sudo bin/opensearch-dashboards-plugin install https://d3g5vo6xdbdb9a.cloudfront.net/downloads/opensearch-dashboards-plugins/opensearch-notebooks/opensearchNotebooksOpenSearch Dashboards-{{site.opensearch_major_minor_version}}.2.0.zip
```

此插件允许你将 OpenSearch 控制面板可视化和叙述文本组合到一个界面中。

#### 报告 OpenSearch 控制面板

```bash
# x86 Linux
sudo bin/opensearch-dashboards-plugin install https://d3g5vo6xdbdb9a.cloudfront.net/downloads/opensearch-dashboards-plugins/opensearch-reports/linux/x64/opensearchReportsOpenSearch Dashboards-{{site.opensearch_major_minor_version}}.2.0-linux-x64.zip
# ARM64 Linux
sudo bin/opensearch-dashboards-plugin install https://d3g5vo6xdbdb9a.cloudfront.net/downloads/opensearch-dashboards-plugins/opensearch-reports/linux/arm64/opensearchReportsOpenSearch Dashboards-{{site.opensearch_major_minor_version}}.2.0-linux-arm64.zip
# x86 Windows
sudo bin/opensearch-dashboards-plugin install https://d3g5vo6xdbdb9a.cloudfront.net/downloads/opensearch-dashboards-plugins/opensearch-reports/windows/x64/opensearchReportsOpenSearch Dashboards-{{site.opensearch_major_minor_version}}.2.0-windows-x64.zip
```

此插件允许你从 OpenSearch 控制面板、控制面板、可视化和保存的搜索中导出和共享报告。

#### 甘特图 OpenSearch 控制面板

```bash
sudo bin/opensearch-dashboards-plugin install https://d3g5vo6xdbdb9a.cloudfront.net/downloads/opensearch-dashboards-plugins/opensearch-gantt-chart/opensearchGanttChartOpenSearch Dashboards-{{site.opensearch_major_minor_version}}.0.0.zip
```

该插件添加了新的甘特图可视化。

{% endcomment %}

## 查看已安装插件的列表

要从命令行查看已安装插件的列表，请使用以下命令：

```bash
sudo bin/opensearch-dashboards-plugin list
```

## 删除插件

要删除插件：

```bash
sudo bin/opensearch-dashboards-plugin remove <plugin-name>
```

然后从 `opensearch_dashboards.yml` 中删除所有关联的条目。

对于某些插件，你还必须删除“优化”捆绑包。以下是异常检测插件的示例命令：

```bash
sudo rm /usr/share/opensearch-dashboards/optimize/bundles/opensearch-anomaly-detection-opensearch-dashboards.*
```

然后重新启动 OpenSearch 控制面板。删除任何插件后，OpenSearch 控制面板会在你下次启动时执行优化操作。即使在快速的机器上，此操作也需要几分钟，因此请耐心等待。

## 更新插件

OpenSearch 控制面板不会更新插件。相反，你必须删除旧版本及其优化的捆绑包，重新安装它们，然后重新启动 OpenSearch 控制面板：

1. 删除旧版本：

   ```bash
   sudo bin/opensearch-dashboards-plugin remove <plugin-name>
   ```

1. 删除优化的捆绑包：

   ```bash
   sudo rm /usr/share/opensearch-dashboards/optimize/bundles/<bundle-name>
   ```

1. 重新安装新版本：

   ```bash
   sudo bin/opensearch-dashboards-plugin install <plugin-name>
   ```

1. 重新启动 OpenSearch 控制面板。

例如，要删除并重新安装异常检测插件，请执行以下操作：

```bash
sudo bin/opensearch-dashboards-plugin remove anomalyDetectionDashboards
sudo rm /usr/share/opensearch-dashboards/optimize/bundles/opensearch-anomaly-detection-opensearch-dashboards.*
sudo bin/opensearch-dashboards-plugin install <AD OpenSearch Dashboards plugin artifact URL>
```
