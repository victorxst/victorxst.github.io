---
layout: default
title: 配置登录选项
parent: 配置
nav_order: 35
---

# 配置仪表板登录以使用多个身份验证选项

您可以配置标志-在OpenSearch仪表板的窗口中，可以为在标志处验证用户提供一个选项-在或多个选项中。当前，仪表板支持基本身份验证，OpenID Connect和SAML作为多个选项。

## 配置多个身份验证选项的一般步骤

在配置符号之前，请考虑以下步骤顺序-在多个身份验证选项的窗口中。

1. 确定在标志处提供哪种类型的身份验证-在。
1. 配置每种身份验证类型，包括身份提供商（IDP）的身份验证域和给出每个类型标志的基本设置-访问OpenSearch仪表板。对于OpenID Connect Backend配置，请参阅[OpenID连接]({{site.url}}{{site.baseurl}}/security/authentication-backends/openid-connect/)。对于SAML后端配置，请参阅[SAML]({{site.url}}{{site.baseurl}}/security/authentication-backends/saml/)。
1. 添加，启用和配置多个选项身份验证设置`opensearch_dashboards.yml` 文件。

## 启用多个身份验证选项

默认情况下，仪表板为符号提供基本身份验证-在。要启用多个身份验证选项，请先添加`opensearch_security.auth.multiple_auth_enabled` 到`opensearch_dashboards.yml` 文件并将其设置为`true`。

指定多种身份验证类型为符号期间的选项-在中，添加`opensearch_security.auth.type` 设置为`opensearch_dashboards.yml` 文件并输入多种类型作为值。当将多种身份验证类型添加到设置中时，仪表板符号-在窗口中识别多种类型和调整以适应标志-在选项中。

在设置仪表板以提供多个身份验证选项时，始终需要基本身份验证作为设置的值之一。
{: .note}

当仅需要一种身份验证类型时，将单个值添加到设置中。

```yml
opensearch_security.auth.type: "openid"
```

对于多个身份验证选项，请将值添加到设置中，作为由逗号分隔的数组。提醒您，仪表板当前支持基本身份验证，OpenID Connect和SAML作为有效值集的组合。在设置中，这些值表示为`"basicauth"`，`"openid"`， 和`"saml"`。

```yml
opensearch_security.auth.type: ["basicauth","openid"]
opensearch_security.auth.multiple_auth_enabled: true
```

```yml
opensearch_security.auth.type: ["basicauth","saml"]
opensearch_security.auth.multiple_auth_enabled: true
```

```yml
opensearch_security.auth.type: ["basicauth","saml","openid"]
opensearch_security.auth.multiple_auth_enabled: true
```

当。。。的时候`opensearch_security.auth.type` 设置包含`basicauth` 还有另一种身份验证类型，标志-在窗口中显示如下示例。

<img src="{{site.url}}{{site.baseurl}}/images/Security/OneOptionWithoutLogo.png" alt="Basic authentication and one other type in the sign-in window" width="350">

指定了所有三种有效的身份验证类型，-在窗口中显示如下示例。

<img src="{{site.url}}{{site.baseurl}}/images/Security/TwoOptionWithoutLogo.png" alt="All three authentication types specified in the sign-in window" width="350">

## 自定义标志-在环境中

除了基本迹象-在每种身份验证类型的设置中，您可以在`opensearch_dashboards.yml` 文件以自定义标志-在窗口中，可以清楚地表示可用的选项。例如，您可以替换标牌上的标签-在包含IDP的名称和图标的按钮中。请参阅以下的设置和描述。

<img src="{{site.url}}{{site.baseurl}}/images/Security/TwoOptionWithLogo.png" alt="Multi-option sign-in window with with some customization" width="350">

### 基本身份验证设置

这些设置允许您自定义基本的用户名和密码符号-在按钮中。

环境| 描述
:--- | :--- |:--- |:--- |
`opensearch_security.ui.basicauth.login.brandimage` |  登录按钮徽标。支持的文件类型为SVG，PNG和GIF。
`opensearch_security.ui.basicauth.login.showbrandimage` |  确定是否显示登录按钮的徽标。默认为`true`。

### OpenID连接身份验证设置

这些设置使您可以自定义标志-在与OpenID Connect身份验证相关的按钮中。对于使用OpenID Connect所需的基本设置-在选项中，请参阅[OpenSearch仪表板单个标志-在]({{site.url}}{{site.baseurl}}/security/authentication-backends/openid-connect/#opensearch-dashboards-single-sign-on)。

环境| 描述
:--- | :--- |:--- |:--- |
`opensearch_security.ui.openid.login.buttonname` |  显示登录按钮的名称。"Log in with single sign-on" 默认情况下。
`opensearch_security.ui.openid.login.brandimage` |  登录按钮徽标。支持的文件类型为SVG，PNG和GIF。
`opensearch_security.ui.openid.login.showbrandimage` |  确定是否显示登录按钮的徽标。默认为`false`。

### SAML身份验证设置

这些设置使您可以自定义标志-在与SAML身份验证相关的按钮中。对于使用SAML作为标志所需的基本设置-在选项中，请参阅[OpenSearch仪表板配置]({{site.url}}{{site.baseurl}}/security/authentication-backends/saml/#opensearch-dashboards-configuration)。

环境| 描述
:--- | :--- |:--- |:--- |
`opensearch_security.ui.saml.login.buttonname` |  显示登录按钮的名称。"Log in with single sign-on" 默认情况下。
`opensearch_security.ui.saml.login.brandimage` |  登录按钮徽标。支持的文件类型为SVG，PNG和GIF。
`opensearch_security.ui.saml.login.showbrandimage` |  确定是否显示登录按钮的徽标。默认为`false`。

## 样本设置
以下示例在`opensearch_dashboards.yml` 在标志时为两种类型的身份验证配置的文件-在。

```yml
# The several settings directly below are typical of all `opensearch_dashboards.yml` configurations. #
server.host: 0.0.0.0
server.port: 5601
opensearch.hosts: ["https://localhost:9200"]
opensearch.ssl.verificationMode: none
opensearch.username: <preferred username>
opensearch.password: <preferred password>
opensearch.requestHeadersAllowlist: ["securitytenant","Authorization"]
opensearch_security.multitenancy.enabled: true
opensearch_security.multitenancy.tenants.preferred: ["Private", "Global"]
opensearch_security.readonly_mode.roles: ["<role_for_read_only>"]

# Settings that enable multiple option authentication in the sign-in window #
opensearch_security.auth.multiple_auth_enabled: true
opensearch_security.auth.type: ["basicauth","openid"]

# Basic authentication customization #
opensearch_security.ui.basicauth.login.brandimage: <path/to/OSlogo.png>
opensearch_security.ui.basicauth.login.showbrandimage: true

# OIDC auth customization and start settings #
opensearch_security.ui.openid.login.buttonname: Log in with <IdP name or other> 
opensearch_security.ui.openid.login.brandimage: <path/to/brand-logo.png>
opensearch_security.ui.openid.login.showbrandimage: true

opensearch_security.openid.base_redirect_url: <"OIDC redirect URL">
opensearch_security.openid.verify_hostnames: false
opensearch_security.openid.refresh_tokens: false
opensearch_security.openid.logout_url: <"OIDC logout URL">

opensearch_security.openid.connect_url: <"OIDC connect URL">
opensearch_security.openid.client_id: <Client ID>
opensearch_security.openid.client_secret: <Client secret>
```

