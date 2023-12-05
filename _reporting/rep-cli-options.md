---
layout: default
title: 报告CLI选项
nav_order: 30
parent: 使用CLI报告
grand_parent: 报告
redirect_from:
  - /dashboards/reporting-cli/rep-cli-options/
---

# 报告CLI选项

您可以将以下任何参数与`opensearch-reporting-cli` 工具。

| 争论| 描述| 可接受的价值和用法| 环境变量
：--------------------- | :--- | :--- |
`-u`，`--url` | 可视化的URL。| 从OpenSearch仪表板>可视化>共享>“永久链接”>复制链接获得。| OpenSearch_url
`-a`，`--auth` | 报告的身份验证类型。| 您可以指定任何一个基本`basic`，认知`cognito`，saml`saml`，或没有auth`none`。如果未指定值，则报告CLI工具默认为无身份验证，键入`none`。基本，cognito和saml需要凭证`-c` 旗帜。| N/A。
`-c`，`--credentials` | OpenSearch登录凭据。| 输入您的用户名和密码，并由结肠分开。例如，用户名：密码。基本，Cognito和SAML身份验证类型所需。| opensearch_username和opensearch_password
`-t`，`--tenant` | OpenSearch仪表板的租户。| 默认租户是私人的。| N/A。
`-f`，`--format` | 报告的文件格式。| 可以是`pdf`，`png`， 或者`csv`。默认值为`pdf`。| N/A。
`-w`，`--width` | 报告的像素中的窗口宽度。| 默认为`1680`。| N/A。
`-l`，`--height` | 报告中的最小窗口高度。| 默认为`600`。| N/A。
`-n`，`--filename` | 报告的文件名。| 默认为`reporting`。| OpenSearch-报告-是的-毫米-DDTHH-毫米-SS.SSSZ
`-e`，`--transport` | 发送电子邮件的运输机制。| 对于亚马逊SES，指定`ses`。Amazon SES需要系统上的AWS配置来存储凭据。对于SMTP，请使用`smtp` 并用`--smtpusername` 和`--smtppassword`。| OpenSearch_transport
`-s`，`--from` | 发件人的电子邮件地址。| 例如，`user@amazon.com`。| opensearch_from
`-r`，`--to` | 收件人的电子邮件地址。| 例如，`user@amazon.com`。| opensearch_to
`--smtphost` | SMTP服务器的主机名。| 例如，`SMTP_HOST`。| opensearch_smtp_host
`--smtpport` | SMTP连接的端口。| 例如，`SMTP_PORT`。| OPENSEARCH_SMTP_PORT
`--smtpsecure` | 指定连接到服务器时使用TLS。| 例如，`SMTP_SECURE`。| opensearch_smtp_secure
`--smtpusername` | SMTP用户名。| 例如，`SMTP_USERNAME`。| opensearch_smtp_username
`--smtppassword` | SMTP密码。| 例如，`SMTP_PASSWORD`。| opensearch_smtp_password
`--subject` | 电子邮件主题文本包含在报价中。| 可以是任何字符串。默认值为"This is an email containing your dashboard report"。| opensearch_email_subject
`--note` | 电子邮件主体，是字符串或文本文件的路径。| 默认注释是"Hi,\\nHere is the latest report!" | opensearch_email_note
`-h`，`--help` | 指定从命令行显示可选参数列表。| N/A。

## 得到帮助

要获取所有可用CLI参数的列表，请运行以下命令：

``` 
$ opensearch-reporting-cli -h
```
