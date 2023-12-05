---
layout: default
title: 创建和请求可视化报告
nav_order: 15
parent: 使用CLI报告
grand_parent: 报告
redirect_from:
  - /dashboards/reporting-cli/rep-cli-create/
---

# 创建和请求可视化报告

首先，您需要获取要下载为图像文件或PDF的可视化的URL。

要生成可视化报告，您需要指定仪表板URL。

打开要生成报告的可视化，然后选择**共享>永久链接>生成链接作为shapshot>简短的URL>复制链接**，如下图所示。

![复制链接]({{site.url}}{{site.baseurl}}/images/dashboards/dash-url.png)

您将需要与`-u` 当您在CLI中请求报告时，请参数。

#### 示例：请求PNG文件

以下命令要求使用基本身份验证的PNG格式报告，并使用Amazon SES发送报告到电子邮件地址：

```
opensearch-reporting-cli -u https://localhost:5601/app/dashboards#/view/7adfa750-4c81-11e8-b3d7-01146121b73d -a basic -c admin:Test@1234 -e ses -s <email address>  -r <email address> -f png
```

#### 示例：请求PDF文件

以下命令请求PDF文件并指定收件人的电子邮件地址：

```
opensearch-reporting-cli -u https://localhost:5601/app/dashboards#/view/7adfa750-4c81-11e8-b3d7-01146121b73d -a basic -c admin:Test@1234 -e ses -s <email address> -r <email address> -f pdf
```

成功后，文件将发送到指定的电子邮件地址。下图显示了示例PDF报告。

![PDF示例]({{site.url}}{{site.baseurl}}/images/dashboards/cli-pdf-report.png)

#### 示例：请求CSV文件

以下命令生成一个报告，该报告包含CSV格式的所有表内容，并使用Amazon SES Transport将报告发送到电子邮件地址：

```
opensearch-reporting-cli -u https://localhost:5601/app/dashboards#/view/7adfa750-4c81-11e8-b3d7-01146121b73d -f csv -a basic -c admin:Test@1234 -e ses -s <email address> -r <email address>
```

成功后，电子邮件将通过附加的CSV文件发送到指定的电子邮件地址。

