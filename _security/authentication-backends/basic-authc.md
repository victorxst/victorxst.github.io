---
layout: default
title: HTTP基本身份验证
parent: 身份验证后端
nav_order: 46
redirect_from:
---


# HTTP基本身份验证

HTTP基本身份验证提供了一个简单的挑战-和-获得访问OpenSearch及其资源的响应过程，该过程提示您使用用户名和密码登录。您在`http_authenticator` 通过指定配置部分`type` 作为`basic`，如以下示例所示：

```yml
authc:
  basic_internal_auth_domain:
    description: "Authenticate using HTTP basic against the internal users database"
    http_enabled: true
    transport_enabled: true
    order: 1
    http_authenticator:
      type: basic
      challenge: true
    authentication_backend:
      type: internal
```

此外，您可以通过指定来指定内部用户数据库作为身份验证后端`internal` 作为类型`authentication_backend`。看[内部用户数据库](#the-internal-user-database) 有关此后端的信息。

一次`basic` 为HTTP身份验证者的类型指定`internal` 为身份验证后端的类型指定，没有进一步的配置`config.yml` 需要，除非您打算使用HTTP基本身份验证的其他身份验证后端。继续阅读与此类型的设置有关的注意事项，以及有关此类设置的更多信息`challenge` 环境。


## 挑战设置

在大多数情况下，设置适合`challenge` 到`true` 用于基本身份验证。此设置定义了当安全插件的行为`Authorization` 未指定HTTP标头中的字段。默认情况下，设置为`true`。

什么时候`challenge` 被设定为`true`，安全插件发送带有状态的响应`UNAUTHORIZED` （401）回到客户。如果客户端正在使用浏览器访问群集，则该触发身份验证对话框，并提示用户输入用户名和密码。当HTTP基本身份验证是唯一使用的后端时，这是一种常见的配置。

什么时候`challenge` 被设定为`false` 和`Authorization` 尚未在请求中指定标头，安全插件未发送`WWW-Authenticate` 回复对客户的响应，身份验证失败。当您有多个具有挑战性时，通常会使用此配置`http_authenticator` 您配置的身份验证域中包含的设置。例如，当您计划一起使用基本身份验证和SAML时，可能就是这种情况。有关此配置的示例和更完整的说明，请参见[运行多个身份验证域]({{site.url}}{{site.baseurl}}/security/authentication-backends/saml/#running-multiple-authentication-domains) 在SAML文档中。

定义多个HTTP身份验证器时，请确保订购非-首先具有挑战性的身份验证者---例如`proxy` 和`clientcert`---并命令挑战HTTP身份验证者。例如，在非配置中-挑战性的HTTP基本身份验证后端与具有挑战性的SAML后端配对，您可以指定`order: 0` 在HTTP基本中`authc` 域和`order: 1` 在SAML域中。
{:.note}


## 内部用户数据库

使用HTTP基本身份验证时，内部用户数据库存储内部用户，并包含其Hashed密码和其他用户属性（例如角色）。用户及其设置保存在`internal_users.yml` 配置文件。有关此文件的更多信息，请参阅[internal_users.yml]({{site.url}}{{site.baseurl}}/security/configuration/yaml/#internal_usersyml) 在安全配置文档中。


