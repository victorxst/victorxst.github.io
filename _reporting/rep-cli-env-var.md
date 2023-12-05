---
layout: default
title: 在报告CLI中使用环境变量
nav_order: 35
parent: 使用CLI报告
grand_parent: 报告
redirect_from:
  - /dashboards/reporting-cli/rep-cli-env-var/
---

# 在报告CLI中使用环境变量

您可以将其保存为环境变量，而不是在命令行中明确提供值。报告CLI从项目内的当前目录中读取环境变量。

要在Linux中设置环境变量，请使用以下命令：

```
export NAME=VALUE
```

每行应使用格式`NAME=VALUE`。
每行以主题标签开头（#）被认为是评论。
引号（“）没有任何特殊处理。

来自命令行参数的值比环境文件具有更高的优先级。例如，如果您在 * .env *文件中添加文件名为 * test *，并且还添加`--filename report` 命令选项，生成的报告的名称将为 *报告 *。
{: .note }

#### 示例：请求使用环境变量设置的PNG报告

以下命令请求以PNG格式的基本身份验证的报告：

```
opensearch-reporting-cli --url https://localhost:5601/app/dashboards#/view/7adfa750-4c81-11e8-b3d7-01146121b73d --format png --auth basic --credentials admin:admin
```

成功后，该报告将下载到当前目录。

## 使用Amazon SES请求带有报告附件的电子邮件

要使用Amazon SE作为电子邮件传输机制，请使用以下先决条件：

- 发件人的电子邮件地址必须由[亚马逊SES](https://aws.amazon.com/ses/)。这[AWS命令线接口（AWS CLI）](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html) 与Amazon SES互动需要。要配置AWS CLI使用的基本设置，请参见[快速配置`aws configure`](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config) 在AWS命令行接口用户指南中。
- 亚马逊SES运输需要`ses:SendRawEmail` 角色：

```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ses:SendRawEmail",
      "Resource": "*"
    }
  ]
}
```

以下命令要求随附的报告一封电子邮件：

```
opensearch-reporting-cli --url https://localhost:5601/app/dashboards#/view/7adfa750-4c81-11e8-b3d7-01146121b73d --transport ses --from <sender_email_id> --to <recipient_email_id>
```

以下命令使用所有其他选项使用默认值。您也可以设置`OPENSEARCH_FROM`，`OPENSEARCH_TO`， 和`OPENSEARCH_TRANSPORT` 在您的.env文件中使用以下命令：

```
opensearch-reporting-cli --url https://localhost:5601/app/dashboards#/view/7adfa750-4c81-11e8-b3d7-01146121b73d
```

要修改电子邮件的正文，您可以编辑 * index.hbs *文件。

#### 示例：将报告发送到使用SMTP的电子邮件地址

要通过SMTP运输将报告发送到电子邮件地址，您需要设置选项`OPENSEARCH_SMTP_HOST`，`OPENSEARCH_SMTP_PORT`，`OPENSEARCH_SMTP_USER`，`OPENSEARCH_SMTP_PASSWORD`， 和`OPENSEARCH_SMTP_SECURE` 在您的.env文件中。

一旦在.ENV文件中设置了传输选项后，您可以使用以下命令发送电子邮件：

```
opensearch-reporting-cli --url https://localhost:5601/app/dashboards#/view/7adfa750-4c81-11e8-b3d7-01146121b73d --transport smtp --from <sender_email_id> --to <recipient_email_id>
```

您可以在任何组合中选择使用 * .env *文件或命令行参数值设置选项。确保指定所有必需的值以避免错误。

要修改电子邮件的正文，您可以编辑 * index.hbs *文件。

## 限制

以下限制适用于报告CLI的环境变量使用情况：

- 受支持的平台是Windows X86，Windows X64，Mac Intel，Mac Arm，Linux X86和Linux X64。
  
  对于任何其他平台，用户可以利用 * Chromium_path *环境变量使用自定义铬。

- 如果URL包含一个感叹号（！），则需要暂时禁用历史记录。根据您使用的外壳的不同，您可以使用以下命令之一禁用历史记录扩展：

  *为bash，请使用`set +H`。
  *对于ZSH，请使用`setopt nobanghist`。

  另外，您可以使用此格式添加一个URL值作为环境变量：`URL="<url-with-!>"`。

- 所有命令选项仅接受小写字母。

## 故障排除

解决**Messagereject：未验证电子邮件地址**， 看[我为什么要获得400"message rejected" 消息错误"Email address is not verified" 来自亚马逊SES？](https://repost.aws/knowledge-center/ses-554-400-message-rejected-error) 在AWS知识中心

