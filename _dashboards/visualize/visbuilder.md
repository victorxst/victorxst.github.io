---
layout: default
title: 使用visbuilder
parent: 构建数据可视化
nav_order: 100
redirect_from:
  - /dashboards/drag-drop-wizard/
---

# 使用visbuilder

Visbuilder是一个实验特征，不应在生产环境中使用。有关其进度的最新信息，或者如果要留下有助于改进功能的反馈，请参阅[Github问题](https://github.com/opensearch-project/OpenSearch-Dashboards/issues/2280)。
{: .warning}

您可以在OpenSearch仪表板中使用Visbuilder可视化类型来创建数据可视化-和-掉落的手势。有了visbuilder，您有：

*即时视图，无需预先选择可视化输出。
*快速更改可视化类型和索引模式的灵活性。
*在多个屏幕之间轻松导航的能力。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/vis-builder-2.png" alt="VisBuilder new visualization start page">

## 尝试在OpenSearch仪表板游乐场中的Visbuilder

如果您想在不本地安装OpenSearch的情况下尝试Visbuilder，则可以在[仪表板操场](https://playground.opensearch.org/app/vis-builder#/)。默认情况下启用Visbuilder。

## 在本地尝试粘粘器

默认情况下启用Visbuilder。如果要禁用它，请设置功能`flag vis_builder.enabled:` 到`false` 在里面`opensearch_dashboards.yml` 文件如下：

```
# Set the value of this setting to false to disable VisBuilder
# functionality in Visualization.
vis_builder.enabled: false
``` 

请按照以下步骤在您的环境中使用Visbuilder创建新的可视化：

1. 开放仪表板：
    - 如果您不运行安全插件，请访问http：// localhost：5601。
    - 如果您正在运行安全插件，请访问https：// localhost：5601，并使用您的用户名和密码登录（默认为admin/admin）。

2. 确认**启用实验可视化** 选项已打开。
   - 从顶部菜单中选择**管理** >**仪表板管理** >**高级设置**。
   - 选择**可视化** 并验证该选项是否已打开。
   
   <img src ="{{site.url}}{{site.baseurl}}/images/enable-experimental-viz.png" alt ="Enable experimental visualizations" 宽度="600">

3. 从顶部菜单中选择**可视化** **>** **创建可视化** **>** **visbuilder**。

   <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/vis-builder-1.png" alt ="Select the VisBuilder visualization type" 宽度="550">

4. 将左列的字段名称拖动到**配置** 面板生成可视化。

这是一个示例可视化。根据您的数据和选择的字段，您的可视化看起来会有所不同。

<img src="{{site.url}}{{site.baseurl}}/images/drag-drop-generated-viz.png" alt="Visualization generated using sample data">

