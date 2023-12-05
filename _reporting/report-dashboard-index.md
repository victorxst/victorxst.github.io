---
layout: default
title: 使用OpenSearch仪表板报告
nav_order: 5
redirect_from:
  - /dashboards/reporting/
---


# 使用OpenSearch仪表板报告

您可以使用OpenSearch仪表板来创建PNG，PDF和CSV报告。要创建报告，您必须拥有正确的权限。有关预定义角色及其授予的权限的摘要，请参阅[安全插件]({{site.url}}{{site.baseurl}}/security/access-control/users-roles#predefined-roles)。

CSV报告没有-可配置的10,000行限制。它们没有明确的尺寸限制（例如，MB），但是非常大的文档可能会导致报告生成失败，而V8 JavaScript引擎的内存错误失败。
{： 。提示 }

## 生成报告

从接口生成报告：

1. 从导航面板中选择**报告**。
2. 对于仪表板，可视化或笔记本，请选择**下载PDF** 或者**下载PNG**。如果您正在从发现页面创建报告，请选择**生成CSV**。

报告在后台产生异步，可能需要几分钟，具体取决于报告的大小。当您的报告准备下载时，会出现通知。
{： 。笔记}

3. 创建时间表-基于报告，选择**创建报告定义**。然后继续[使用定义创建报告](#creating-reports-using-a-definition)。此选项pre-根据您正在查看的可视化，仪表板或数据，为您填写许多字段。


## 使用定义创建报告

定义可让您按定期时间表生成报告。

1. 从导航面板中选择**报告**。
1. 选择**创造**。
1. 在下面**报告设置**，为您的报告输入名称和可选描述。
1. 选择**报告来源** （即生成报告的页面）。您可以从**仪表板**，，，，**可视化**，，，，**发现** （保存搜索）或**笔记本** 页面。
1. 选择您的仪表板，可视化，保存的搜索或笔记本。然后选择报告的时间范围。
1. 为报告选择适当的文件格式。
1. （可选）在报告中添加标头或页脚。标题和页脚仅用于仪表板，可视化和笔记本报告。
1. 在下面**报告触发器**，选择**一经请求** 或者**日程**。

   对于预定的报告，选择任一**再次发生的** 或者**基于克朗**。您可以每天或在其他某些时间间隔收到报告，而Cron表达式可以使您更加灵活。看[cron表达参考]({{site.url}}{{site.baseurl}}/monitoring-plugins/alerting/cron/) 了解更多信息。

2. 选择**创造**。

## 故障排除

您可以使用以下主题来解决和解决报告问题。

### Chromium无法使用OpenSearch仪表板启动

在创建仪表板或可视化的报告时，您可能会看到以下错误：

![OpenSearch仪表板报告流行-UP错误消息]({{site.url}}{{site.baseurl}}/images/reporting-error.png)

这个问题可能有两个原因：

- 您没有正确的版本`headless-chrome` 要匹配OpenSearch仪表板正在运行的操作系统。下载[正确的版本](https://github.com/opensearch-project/reporting/releases/tag/chromium-1.12.0.0)。

- 您缺少其他依赖关系。从[其他库](https://github.com/opensearch-project/dashboards-reports/blob/1.x/dashboards-reports/rendering-engine/headless-chrome/README.md#additional-libaries) 部分。

### 字符不加载在报告中

您可能会遇到一个UTF的问题-8个编码字符在您的浏览器中看起来不错，但是由于您缺少所需的字体依赖项，因此它们不会加载到生成的报告中。安装[字体依赖性](https://github.com/opensearch-project/dashboards-reports#missing-font-dependencies)，然后再次生成您的报告。

