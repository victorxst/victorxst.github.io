---
layout: default
title: 通知
nav_order: 90
has_children: true
redirect_from:
  - /notifications-plugin/
  - /notifications-plugin/index/
---

# 通知

通知插件为OpenSearch插件的所有通知提供了一个中心位置。使用插件，您可以配置要使用的通信服务，并查看相关的统计信息和故障排除信息。当前，警报和ISM插件已与通知插件集成在一起。

您可以使用OpenSearch仪表板或REST API来配置通知。仪表板提供了一种更有条理的方法，可以选择要使用的频道类型并选择要使用的OpenSearch插件源，而REST API则可以使您可以编程定义您的通知频道，以更好地版本化和以后重复使用。

1. 使用仪表板UI首先创建一个从其他插件接收通知的通道。支持的通信渠道包括亚马逊钟声，亚马逊简单通知服务（Amazon SNS），Amazon简单电子邮件服务（Amazon SES），通过SMTP，Slack，Microsoft Teams和自定义Webhooks的电子邮件。配置了频道和插件源后，发送消息并开始从通知插件的仪表板中跟踪您的通知。

2. 使用通知REST API配置所有频道的设置。要使用API，您必须具有通知的姓名，描述，频道类型，将其用作源插件以及其他相关的URL或组。

## 创建一个频道

在OpenSearch仪表板中，选择**通知**，，，，**频道**， 和**创建频道**。

1. 在里面**名称和描述** 部分，为您的频道指定名称和可选描述。
2. 在里面**配置** 部分，选择频道类型，然后输入每种类型的必要信息。有关配置使用Amazon SNS或电子邮件的频道的更多信息，请参阅以下各节。如果要使用Amazon Chime或Slack，则需要指定Webhook URL。有关使用Webhooks的更多信息，请参见文档[松弛](https://api.slack.com/messaging/webhooks)，，，，[微软团队](https://learn.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/what-are-webhooks-and-connectors)， 或者[亚马逊钟](https://docs.aws.amazon.com/chime/latest/ug/webhooks.html)。

如果要使用自定义Webhooks，则必须指定更多信息：参数和标题。例如，如果您的端点需要基本的身份验证，则可能需要添加带有授权键的标头，值为`Basic <Base64-encoded-credential-string>`。您可能还需要更改`Content-Type` 无论您需要什么。流行价值是`application/json`，，，，`application/xml`， 和`text/plain`。

此信息以纯文本存储在OpenSearch集群中。我们将来会改进此设计，但是目前，其他OpenSearch用户可以看到编码的凭据（既没有加密也不被加密）。

1. 在里面**可用性** 部分，选择要与Notification Channel一起使用的OpenSearch插件。
2. 选择**创造**。

### 亚马逊SNS作为频道类型

OpenSearch支持Amazon SNS通知。与Amazon SNS的集成意味着，除其他频道类型外，通知插件还可以使用SNS主题发送电子邮件消息，文本消息，甚至运行AWS lambda函数。有关亚马逊SNS的更多信息，请参阅[亚马逊简单通知服务开发人员指南](https://docs.aws.amazon.com/sns/latest/dg/welcome.html)。

通知插件当前支持对用户进行身份验证的两种方法：

1. 为用户提供对Amazon SNS的完整访问权限。
2. 让用户扮演具有访问Amazon SNS的权限的AWS身份和访问管理（IAM）角色。将通知通道配置为使用右Amazon SNS权限后，请选择可以触发通知的OpenSearch插件。

### 提供完整的Amazon SNS访问权限

如果您想向IAM用户提供全面的Amazon SNS访问权限，请确保用户具有以下权限：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "sns:*"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```

### 在Amazon SNS权限中担任IAM角色

如果要让用户在不直接对Amazon SNS获得完整权限的情况下发送通知，请让用户扮演具有必要权限的角色。

IAM用户必须具有以下权限以扮演角色：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*",
        "iam:ListRoles",
        "sts:AssumeRole"
      ],
      "Resource": "*"
    }
  ]
}
```

然后将此策略添加到IAM用户的信任关系中，以实际扮演该角色：

```json
{
  "Version": "2012-10-17",
  "Statement": [
  {
    "Effect": "Allow",
    "Principal": {
    "AWS": "arn:aws:iam::<arn_number>:user/<iam_username>",
    },
    "Action": "sts:AssumeRole"
  }
  ]
}
```


## 电子邮件作为频道类型

要通过电子邮件发送或接收通知，请选择**电子邮件** 作为通道类型。接下来，选择至少一个发件人和默认收件人。要一次向几个人发送通知，请指定多个电子邮件地址或选择一个收件人组。如果通知插件当前没有必要的发件人或组，则可以先选择来添加它们**SMTP发件人** 然后选择**创建SMTP发件人** 或者**创建收件人组**。选择**SES发件人** 使用亚马逊简单的电子邮件服务（Amazon SES）。

### 创建电子邮件发送者

1. 指定一个唯一名称以与发件人关联。
2. 输入电子邮件地址，如果适用，则将其主机（例如SMTP.GMAIL.com）和端口。如果您使用的是Amazon SES，请输入AWS帐户的IAM角色Amazon Resource名称（ARN）与AWS区域一起发送通知。
3. 选择一种加密方法。大多数电子邮件提供商都需要安全套接字层（SSL）或传输层安全性（TLS），该层要求在OpenSearch keystore中使用用户名和密码。看[身份验证的发送者帐户](#authenticate-sender-account) 了解更多。仅当您创建SMTP发件人时，选择加密方法才适用。
4. 选择**创造** 保存配置并创建发件人。您可以在将凭据添加到OpenSearch KeyStore之前创建发件人；但是，你必须[验证每个发件人帐户](#authenticate-sender-account) 在使用频道配置中的发件人之前。

### 创建电子邮件收件人组

1. 选择后**创建收件人组**，输入一个唯一名称，与电子邮件组相关联和可选描述。
2. 选择或输入要添加到收件人组的电子邮件地址。
3. 选择**创造**。

### 身份验证的发送者帐户

如果您的电子邮件提供商需要SSL或TLS，则必须在发送电子邮件之前对每个发件人帐户进行身份验证。使用命令行接口（CLI）在OpenSearch密钥库中输入发件人帐户凭据。运行以下命令（在OpenSearch目录中）输入您的用户名和密码。＆lt; sender_name＆gt;是您输入的名称**发件人** 较早。

```json
opensearch.notifications.core.email.<sender_name>.username
opensearch.notifications.core.email.<sender_name>.password
```

要更改或更新您的凭据（在每个节点上将其添加到密钥库之后），请致电Reload API自动更新这些凭据而不重新启动OpenSearch。

```json
POST _nodes/reload_secure_settings
{
  "secure_settings_password": "1234"
}
```

