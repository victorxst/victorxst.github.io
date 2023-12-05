---
layout: default
title: 使用CLI报告
nav_order: 10
has_children: true
redirect_from:
  - /dashboards/reporting-cli/rep-cli-index/
---

# 使用CLI报告

您可以在不使用OpenSearch仪表板或报告插件的情况下，以PDF或PNG格式编程以PDF或PNG格式创建仪表板报告。这使您可以在电子邮件工作流中自动创建报告。

如果要下载CSV文件，则需要安装报告插件。
{： 。笔记 }

对于任何仪表板视图，您可以请求以PNG或PDF格式的报告发送到电子邮件地址。这对于通过电子邮件别名将报告发送给多个电子邮件收件人很有用。支持创建CSV报告的唯一仪表板应用程序是**发现**。

使用报告CLI，您可以在命令行中指定报告的选项。默认情况下，该报告将作为PDF附件发送到电子邮件地址。您还可以请求PNG图像或CSV文件`--formats` 争论。

您可以将报告下载到正在运行报告CLI的目录中，也可以通过指定Amazon Simple电子邮件服务（Amazon SES）或SMTP来通过电子邮件发送电子邮件。

您可以使用以下任何身份验证类型连接到OpenSearch：

- **基本的**  - 基本的HTTP身份验证。使用`-a basic`。
- **认知**  - 通过Amazon Cognito进行身份验证。使用`-a cognito`。
- **SAML**  - 身份提供商和服务提供商之间的身份验证。使用`-a saml`。Okta提供SAML第三-政党身份验证。
- **没有auth**  - 没有身份验证。使用`-a none`。身份验证默认为无验证`-a` 标志未指定。

要了解有关Amazon Cognito的更多信息，请参阅[什么是亚马逊Cognito？](https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html)。

<！--
### 旁路身份验证选项

报告CLI工具使您可以将其集成到自己的工作流程或环境中，以便您可以绕过身份验证或潜在的安全问题。例如，如果您在AWS lambda实例中使用报告CLI工具，只要您在OpenSearch仪表板中运行报告插件，就不会发生安全问题。在这种情况下，您将使用"No auth" 绕过身份验证过程。指定"No Auth" 使用`--auth none` 在您的要求中。Lambda用户应测试以确保他们可以绕过仪表板的访问，而无需使用NO auth。

要获取所有选项的列表，请参见[报告CLI选项](#reporting-cli-options)。
--

