---
layout: default
title: API
nav_order: 50
parent: 通知
redirect_from:
  - /notifications-plugin/api/
---

# 通知API

如果要通过编程方式定义您的通知频道和版本控制和重用的源，则可以使用Notifications REST API来定义，配置和删除通知频道并发送测试消息。

---

#### Table of contents
1. TOC
{:toc}

---

## 列表支持的频道配置

要检索所有受支持的通知配置类型的列表，请将Get请求发送到`features` 资源。

#### 示例请求

```json
GET /_plugins/_notifications/features
```

#### 示例响应

```json
{
  "allowed_config_type_list" : [
    "slack",
    "chime",
    "webhook",
    "email",
    "sns",
    "ses_account",
    "smtp_account",
    "email_group"
  ],
  "plugin_features" : {
    "tooltip_support" : "true"
  }
}
```

## 列出所有通知配置

要检索所有通知配置的列表，请将get请求发送到`configs` 资源。

#### 示例请求

```json
GET _plugins/_notifications/configs
```

#### 示例响应

```json
{
  "start_index" : 0,
  "total_hits" : 2,
  "total_hit_relation" : "eq",
  "config_list" : [
    {
      "config_id" : "sample-id",
      "last_updated_time_ms" : 1652760532774,
      "created_time_ms" : 1652760532774,
      "config" : {
        "name" : "Sample Slack Channel",
        "description" : "This is a Slack channel",
        "config_type" : "slack",
        "is_enabled" : true,
        "slack" : {
          "url" : "https://sample-slack-webhook"
        }
      }
    },
    {
      "config_id" : "sample-id2",
      "last_updated_time_ms" : 1652760735380,
      "created_time_ms" : 1652760735380,
      "config" : {
        "name" : "Test chime channel",
        "description" : "A test chime channel",
        "config_type" : "chime",
        "is_enabled" : true,
        "chime" : {
          "url" : "https://sample-chime-webhook"
        }
      }
    }
  ]
}
```

要过滤通知配置类型此请求返回，您可以使用以下可选路径参数来完善查询。

范围| 描述
:--- | :---
config_id| 指定通道标识符。
config_id_list| 指定逗号-频道ID的分开列表。
from_index| 搜索的起始索引。
max_items| 根据您的要求返回的最大项目数量。
排序| 指定对结果进行排序的方向。有效的选项是`asc` 和`desc`。
sort_field| 字段与结果进行排序。
last_updated_time_ms| 频道最后更新频道的时间为毫秒的时间。
create_time_ms| 创建频道的时间毫秒的Unix时间。
IS_ENABLED| 指示是否启用了通道。
config_type| 通道类型。有效的选项是`sns`，`slack`，`chime`，`webhook`，`smtp_account`，`ses_account`，`email_group`， 和`email`。
姓名| 频道名称。
描述| 频道描述。
email.email_account_id| 发件人电子邮件地址频道使用。
email.email_group_id_list| 电子邮件组使用的电子邮件组。
email.recipient_list| 频道收件人列表。
email_group.recipient_list| 电子邮件收件人组的频道列表。
smtp_account.method| 电子邮件加密方法。
Slack.url| 松弛通道URL。
chime.url| Amazon Chime连接URL。
webhook.url| Webhook URL。
SMTP_ACCOUNT.HOST| SMTP帐户的域。
smtp_account.from_address| 电子邮件帐户的发送者地址。
smtp_account.method| SMTP帐户的加密方法。
sns.topic_arn| 亚马逊简单通知服务（SNS）主题的ARN。
sns.role_arn| 亚马逊SNS主题的角色。
SES_ACCOUNT.RIGION| 亚马逊简单电子邮件服务（SES）帐户的AWS地区。
ses_account.role_arn| 亚马逊SES帐户的角色。
ses_account.from_address| Amazon SES帐户的发送者电子邮件地址。

## 创建通道配置

要创建通知频道配置，请将POST请求发送到`configs` 资源。

#### 示例请求

```json
POST /_plugins/_notifications/configs/
{
  "config_id": "sample-id",
  "name": "sample-name",
  "config": {
    "name": "Sample Slack Channel",
    "description": "This is a Slack channel",
    "config_type": "slack",
    "is_enabled": true,
    "slack": {
      "url": "https://sample-slack-webhook"
    }
  }
}
```

创建通道API操作接受其请求主体中的以下字段：

场地|数据类型|描述|必需的
:--- | :--- | :--- | :---
config_id| 细绳| 配置的自定义ID。| 不
config| 目的|包含所有相关信息，例如频道名称，配置类型和插件源。|是的
姓名| 细绳|频道的名称。| 是的
描述|细绳| 频道的描述。| 不
config_type|细绳| 您的通知目的地。有效的选项是`sns`，`slack`，`chime`，`webhook`，`smtp_account`，`ses_account`，`email_group`， 和`email`。| 是的
IS_ENABLED| 布尔| 指示是否启用了用于发送和接收通知的通道。默认是正确的。| 不

创建频道操作接受多个`config_types` 作为可能的通知目的地，因此请遵循您的首选格式`config_type`。

```json
"sns": {
  "topic_arn": "<arn>",
  "role_arn": "<arn>" //optional
}
"slack": {
  "url": "https://sample-chime-webhoook"
}
"chime": {
  "url": "https://sample-amazon-chime-webhoook"
}
"webhook": {
      "url": "https://custom-webhook-test-url.com:8888/test-path?params1=value1&params2=value2"
}
"smtp_account": {
  "host": "test-host.com",
  "port": 123,
  "method": "start_tls",
  "from_address": "test@email.com"
}
"ses_account": {
  "region": "us-east-1",
  "role_arn": "arn:aws:iam::012345678912:role/NotificationsSESRole",
  "from_address": "test@email.com"
}
"email_group": { //Email recipient group
  "recipient_list": [
    {
      "recipient": "test-email1@test.com"
    },
    {
      "recipient": "test-email2@test.com"
    }
  ]
}
"email": { //The channel that sends emails
  "email_account_id": "<smtp or ses account config id>",
  "recipient_list": [
    {
      "recipient": "custom.email@test.com"
    }
  ],
  "email_group_id_list": []
}
```

以下示例演示了如何使用电子邮件作为一个`config_type`：

```json
POST /_plugins/_notifications/configs/
{
  "id": "sample-email-id",
  "name": "sample-name",
  "config": {
    "name": "Sample Email Channel",
    "description": "Sample email description",
    "config_type": "email",
    "is_enabled": true,
    "email": {
      "email_account_id": "<email_account_id>",
      "recipient_list": [
        "sample@email.com"
      ]
    }
  }
}
```

#### 示例响应

```json
{
  "config_id" : "<config_id>"
}
```


## 获取通道配置

通过`config_id`，发送get请求并指定`config_id` 作为路径参数。

#### 示例请求

```json
GET _plugins/_notifications/configs/<config_id>
```

#### 示例响应

```json
{
  "start_index" : 0,
  "total_hits" : 1,
  "total_hit_relation" : "eq",
  "config_list" : [
    {
      "config_id" : "sample-id",
      "last_updated_time_ms" : 1652760532774,
      "created_time_ms" : 1652760532774,
      "config" : {
        "name" : "Sample Slack Channel",
        "description" : "This is a Slack channel",
        "config_type" : "slack",
        "is_enabled" : true,
        "slack" : {
          "url" : "https://sample-slack-webhook"
        }
      }
    }
  ]
}
```


## 更新频道配置

要更新频道配置，请将发布请求发送到`configs` 资源并指定频道的`config_id` 作为路径参数。在请求主体中指定新的配置详细信息。

#### 示例请求

```json
PUT _plugins/_notifications/configs/<config_id>
{
  "config": {
    "name": "Slack Channel",
    "description": "This is an updated channel configuration",
    "config_type": "slack",
    "is_enabled": true,
    "slack": {
      "url": "https://hooks.slack.com/sample-url"
    }
  }
}
```

#### 示例响应

```json
{
  "config_id" : "<config_id>"
}
```


## 删除通道配置

要删除通道配置，请将删除请求发送到`configs` 资源并指定`config_id` 作为路径参数。

#### 示例请求

```json
DELETE /_plugins/_notifications/configs/<config_id>
```

#### 示例响应

```json
{
  "delete_response_list" : {
  "<config_id>" : "OK"
  }
}
```

您也可以提交逗号-您要删除的频道ID列表，然后OpenSearch删除所有指定的通知频道。

#### 示例请求

```json
DELETE /_plugins/_notifications/configs/?config_id_list=<config_id1>,<config_id2>,<config_id3>...
```

#### 示例响应

```json
{
  "delete_response_list" : {
  "<config_id1>" : "OK",
  "<config_id2>" : "OK",
  "<config_id3>" : "OK"
  }
}
```


## 发送测试通知

要发送测试通知，请将GET请求发送到`/feature/test/` 并指定频道配置的`config_id` 作为路径参数。

#### 示例请求

```json
GET _plugins/_notifications/feature/test/<config_id>
```

#### 示例响应

```json
{
  "event_source" : {
    "title" : "Test Message Title-0Jnlh4ABa4TCWn5C5H2G",
    "reference_id" : "0Jnlh4ABa4TCWn5C5H2G",
    "severity" : "info",
    "tags" : [ ]
  },
  "status_list" : [
    {
      "config_id" : "0Jnlh4ABa4TCWn5C5H2G",
      "config_type" : "slack",
      "config_name" : "sample-id",
      "email_recipient_status" : [ ],
      "delivery_status" : {
        "status_code" : "200",
        "status_text" : """<!doctype html>
<html>
<head>
</head>
<body>
<div>
    <h1>Example Domain</h1>
    <p>Sample paragraph.</p>
    <p><a href="sample.example.com">TO BE OR NOT TO BE, THAT IS THE QUESTION</a></p>
</div>
</body>
</html>
"""
      }
    }
  ]
}

```

