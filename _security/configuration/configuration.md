---
layout: default
title: 配置安全后端
parent: 配置
nav_order: 5
redirect_from:
 - /security-plugin/configuration/configuration/
---

# 配置安全后端

设置安全插件时的第一步之一是确定要使用哪种身份验证后端。后端在身份验证中扮演的角色涵盖[身份验证流的步骤2和3]({{site.url}}{{site.baseurl}}/security/authentication-backends/authc-index/#authentication-flow)。该插件具有内部用户数据库，但是许多人喜欢使用现有的身份验证后端，例如LDAP服务器或两者的某种组合。

用于配置身份验证和授权后端的主要文件是`config/opensearch-security/config.yml`。该文件定义了安全插件如何检索用户凭据，如何验证凭据以及当选择后端进行身份验证和授权支持此功能时如何拿起其他角色。本主题提供了配置文件及其设置安全性要求的基本概述。有关配置特定后端的信息，请参见[身份验证后端]({{site.url}}{{site.baseurl}}/security/authentication-backends/authc-index/)。

这`config.yml` 文件包括三个主要部分：

```yml
config:
  dynamic:
    http:
      ...
    authc:
      ...
    authz:
      ...
```

以下各节描述了该部分的主要元素`config.yml` 文件并提供其配置的基本示例。有关一个更详细的例子，请参阅[github上的示例文件](https://github.com/opensearch-project/security/blob/main/config/config.yml)。


## http

这`http` 部分包括以下格式：

```yml
http:
  anonymous_auth_enabled: <true|false>
  xff: # optional section
    enabled: <true|false>
    internalProxies: <string> # Regex pattern
    remoteIpHeader: <string> # Name of the header in which to look. Typically: x-forwarded-for
    proxiesHeader: <string>
    trustedProxies: <string> # Regex pattern
```

下表中描述了此配置中使用的设置。

| 环境| 描述|
| :--- | :--- |
| `anonymous_auth_enabled` | 启用或禁用匿名身份验证。什么时候`true`，HTTP身份验证器尝试在HTTP请求中查找用户凭据。如果找到凭据，则对用户进行身份验证。如果找不到，则用户被认证为"anonymous" 用户。然后，该用户具有用户名"anonymous" 还有一个命名的角色"anonymous_backendrole"。当您启用匿名身份验证时，所有定义[HTTP身份验证者](#authentication) 是非-具有挑战性的。也看[挑战设置]({{site.url}}{{site.baseurl}}/security/authentication-backends/basic-authc/#the-challenge-setting)。|
| `xff` | 用于配置代理-基于身份验证。有关此后端的更多信息，请参阅[代理人-基于身份验证]({{site.url}}{{site.baseurl}}/security/authentication-backends/proxy/)。|

如果禁用匿名身份验证，则安全插件将不会初始化，如果您没有提供至少一个`authc`。
{: .important }


## 验证

这`authc` 部分具有以下格式：

```yml
authc:
  <domain_name>:
    http_enabled: <true|false>
    transport_enabled: <true|false>
    order: <integer>
    http_authenticator:
      ...
    authentication_backend:
      ...
```

进入`authc` 部分称为 *身份验证域 *。它指定了获得用户凭据的位置以及应对其进行身份验证的后端。

您可以使用多个身份验证域。每个身份验证域都有一个名称（例如，`basic_auth_internal`），用于在其余和运输层上启用域的设置，然后`order`。该顺序使链条身份验证域成为可能。安全插件按照您提供的顺序使用它们。如果用户成功地使用一个域进行身份验证，则安全插件会跳过其余域。

下表中包含在配置的这一部分中通常找到的设置。

| 环境| 描述|
| :--- | :--- |
| `http_enabled` | 在其余层上启用或禁用身份验证。默认为`true` （启用）。|
| `transport_enabled` | 在运输层上启用或禁用身份验证。默认为`true` （启用）。|
| `order` | 确定在组合配置多个后端时，确定身份验证域与身份验证请求查询的顺序。一旦身份验证成功，任何剩余的域就无需查询。它的价值是整数。|

这`http_authenticator` 定义指定HTTP层的身份验证方法。以下示例显示了用于定义HTTP身份验证器的语法：

```yml
http_authenticator:
  type: <type>
  challenge: <true|false>
  config:
    ...
```

这`type` 设置`http_authenticator` 接受以下值。有关每个身份验证选项的更多信息，请参阅“身份验证后端的链接”[下一步](#next-steps)。

| 价值| 描述|
| :--- | :--- |
| `basic` | HTTP基本身份验证。有关使用基本身份验证的更多信息，请参见HTTP基本身份验证文档。|
| `jwt` | JSON Web令牌（JWT）身份验证。有关其他配置信息，请参见JSON Web令牌文档。|
| `openid` | OpenID连接身份验证。有关其他配置信息，请参见OpenID Connect文档。|
| `saml` | SAML身份验证。有关其他配置信息，请参见SAML文档。|
| `proxy`，`extended-proxy` | 代理人-基于身份验证。这`extended-proxy` 键入Authenticator允许您传递其他用户属性以供文档使用-级别的安全性。请参阅代理-基于其他配置信息的基于身份验证文档。|
| `clientcert` | 通过客户端TLS证书进行身份验证。该证书必须由您的节点信托店的根证书机构之一（CAS）信任。有关其他配置信息，请参见客户端证书身份验证文档。|

设置HTTP身份验证器后，您必须指定要对用户进行身份验证的后端系统：

```yml
authentication_backend:
  type: <type>
  config:
    ...
```

下表显示了`type` 设置下`authentication_backend`。

| 价值| 描述|
| :--- | :--- |
| `noop` | 没有对任何后端系统进行进一步的身份验证。使用`noop` 如果HTTP身份验证者已经完全对用户进行了认证，例如JWT或客户端证书身份验证。|
| `internal` | 使用用户和定义的角色`internal_users.yml` 用于身份验证。|
| `ldap` | 针对LDAP服务器进行身份验证用户。此设置需要[其他LDAP-特定的配置设置]({{site.url}}{{site.baseurl}}/security/authentication-backends/ldap/)。|


## 授权

这`authz` 配置用于从LDAP实现中提取后端角色。对用户进行身份验证后，安全插件可以选择从后端系统收集其他角色。授权配置具有以下格式：

```yml
authz:
  <name>:
    http_enabled: <true|false>
    transport_enabled: <true|false>
    authorization_backend:
      type: <type>
      config:
        ...
```

您可以与身份验证条目一样定义本节中的多个条目。但是，在这种情况下，执行订单与`order` 设置不使用。

下表显示了`type` 设置下`authorization_backend`。

| 价值| 描述|
| :--- | :--- |
| `noop` | 完全跳过授权配置步骤。|
| `ldap` | 从LDAP服务器中获取其他角色。此设置需要[其他LDAP-特定的配置设置]({{site.url}}{{site.baseurl}}/security/authentication-backends/ldap/)。|


## 后端配置示例

默认值`config/opensearch-security/config.yml` 您的OpenSearch发行版中包含的文件包含许多配置示例。将这些示例作为起点，并根据您的需求自定义它们。


## 下一步

要了解配置身份验证后端，请参阅[身份验证后端]({{site.url}}{{site.baseurl}}/security/authentication-backends/) 文档。另外，您可以使用以下主题列表中的链接查看特定后端的文档：

*[HTTP基本身份验证]({{site.url}}{{site.baseurl}}/security/authentication-backends/basic-authc/)
*[JSON网络令牌]({{site.url}}{{site.baseurl}}/security/authentication-backends/jwt/)
*[OpenID连接]({{site.url}}{{site.baseurl}}/security/authentication-backends/openid-connect/)
*[SAML]({{site.url}}{{site.baseurl}}/security/authentication-backends/saml/)
*[Active Directory和LDAP]({{site.url}}{{site.baseurl}}/security/authentication-backends/ldap/)
*[代理人-基于身份验证]({{site.url}}{{site.baseurl}}/security/authentication-backends/proxy/)
*[客户证书身份验证]({{site.url}}{{site.baseurl}}/security/authentication-backends/client-auth/)


<！--- 将kerberos文档删除直到发行#907已解决。
### kerberos

Kerberos身份验证不适用于OpenSearch仪表板。要跟踪OpenSearch在添加OpenSearch仪表板中对Kerberos的支持方面的进度，请参见[问题#907](https://github.com/opensearch-project/security-dashboards-plugin/issues/907) 在仪表板的安全插件存储库中。
{: .warning }

由于Kerberos的性质，您必须定义一些设置`opensearch.yml` 还有一些`config.yml`。

在`opensearch.yml`，定义以下内容：

```yml
plugins.security.kerberos.krb5_filepath: '/etc/krb5.conf'
plugins.security.kerberos.acceptor_keytab_filepath: 'eskeytab.tab'
```

- `plugins.security.kerberos.krb5_filepath` 定义kerberos配置文件的路径。该文件包含有关您的Kerberos安装的各种设置，例如，Kerberos密钥配电中心（KDC）的领域名称，主机名和端口。

- `plugins.security.kerberos.acceptor_keytab_filepath` 定义了键盘文件的路径，其中包含安全插件用于针对Kerberos发出请求的主体。

- `plugins.security.kerberos.acceptor_principal: 'HTTP/localhost'` 定义安全插件用于针对Kerberos发出请求的主体。此值必须在Keytab文件中存在。

由于安全限制，必须将keytab文件放入`config` 或子目录，以及`opensearch.yml` 必须是相对的，而不是绝对的。
{: .note }


#### 动态配置

典型的Kerberos身份验证域中`config.yml` 看起来这样：

```yml
    authc:
      kerberos_auth_domain:
        enabled: true
        order: 1
        http_authenticator:
          type: kerberos
          challenge: true
          config:
            krb_debug: false
            strip_realm_from_principal: true
        authentication_backend:
          type: noop
```

使用Spnego在HTTP级别通过浏览器对Kerberos进行身份验证。Kerberos/Spnego实施情况有所不同，具体取决于您的浏览器和操作系统。当您确定是否需要设置`challenge` 标记为`true` 或者`false`。

和[HTTP基本身份验证](#http-basic)，此标志确定安全插件在没有的情况下应如何反应`Authorization` 标题是在HTTP请求中找到的，或者此标头不等于`negotiate`。

如果设置为`true`，安全插件发送带有状态代码401和A的响应`WWW-Authenticate` 标题设置为`negotiate`。这告诉客户端（浏览器）用`Authorization` 标题集。如果设置为`false`，安全插件无法从请求中提取凭据，并且身份验证失败。环境`challenge` 到`false` 因此，只有在初始请求中发送kerberos凭据时，才有意义。

顾名思义，设置`krb_debug` 到`true` 将输出Kerberos-特定调试消息到`stdout`。如果遇到Kerberos集成问题，请使用此设置。

如果您设置`strip_realm_from_principal` 到`true`，安全插件从用户名剥离了领域。


#### 身份验证后端

由于Kerberos/Spnego在HTTP级别对用户进行身份验证，因此没有其他`authentication_backend` 需要。将此值设置为`noop`。
--->


