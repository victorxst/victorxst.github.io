---
layout: default
title: OpenID连接
parent: 身份验证后端
nav_order: 50
redirect_from:
  - /security-plugin/configuration/openid-connect/
---

# OpenID连接

安全插件可以与使用OpenID Connect标准的识别提供商集成。此功能可以启用以下内容：

*自动配置

  将安全插件指向您的身份提供商（IDP）的元数据，安全插件使用该数据进行配置。

*自动钥匙提取

  安全插件会自动检索从IDP的JSON Web密钥集（JWKS）端点验证JSON Web令牌（JWTS）的公共密钥。您不必在`config.yml`。

*钥匙翻转

  您可以更改直接在IDP中签名JWT的键。如果安全插件检测到未知键，它将尝试从IDP检索。此翻转对用户是透明的。

* OpenSearch仪表板作为单个标志-在仪表板符号中多种身份验证类型中的一个选项或作为一个选项-在窗口中。


## 配置OpenID连接集成

要与OpenID IDP集成，请设置身份验证域并选择`openid` 作为HTTP身份验证类型。JWTS已经包含验证请求所需的所有信息，因此设置`challenge` 到`false` 和`authentication_backend` 到`noop`。

这是最小配置：

```yml
openid_auth_domain:
  http_enabled: true
  transport_enabled: true
  order: 0
  http_authenticator:
    type: openid
    challenge: false
    config:
      subject_key: preferred_username
      roles_key: roles
      openid_connect_url: https://keycloak.example.com:8080/auth/realms/master/.well-known/openid-configuration
  authentication_backend:
    type: noop
```

下表显示了配置参数。

姓名| 描述
:--- | :---
`openid_connect_url` | 安全插件可以找到OpenID连接元数据/配置设置的IDP的URL。此URL在IDP之间有所不同。使用OpenID Connect作为后端时需要。
`jwt_header` | 存储令牌的HTTP标头。通常是`Authorization` 标题与`Bearer` 模式：`Authorization: Bearer <token>`。选修的。默认为`Authorization`。
`jwt_url_parameter` | 如果代币未在HTTP标头中传输，而是作为URL参数，请在此处定义参数的名称。选修的。
`subject_key` | JSON有效负载中存储用户名称的键。如果未定义，[主题](https://tools.ietf.org/html/rfc7519#section-4.1.2) 使用注册索赔。大多数IDP提供商使用`preferred_username` 宣称。选修的。
`roles_key` | JSON有效负载中存储用户角色的钥匙。该密钥的价值必须是逗号-分开的角色列表。仅当您想在JWT中使用角色时才需要。


## OpenID连接URL

OpenID Connect指定各种端点以进行集成目的。最重要的终点是`well-known`，其中列出了安全插件的端点和其他配置选项。

URL在IDP之间有所不同，但通常以`/.well-known/openid-configuration`。

KeyCloak示例：

```
http(s)://<server>:<port>/auth/realms/<realm>/.well-known/openid-configuration
```

安全插件需要的主要信息是`jwks_uri`。此URI指定以JWKS格式的IDP公共键。例如：

```
jwks_uri: "https://keycloak.example.com:8080/auth/realms/master/protocol/openid-connect/certs"
```

```
{
   keys:[
      {
         kid:"V-diposfUJIk5jDBFi_QRouiVinG5PowskcSWy5EuCo",
         kty:"RSA",
         alg:"RS256",
         use:"sig",
         n:"rI8aUrAcI_auAdF10KUopDOmEFa4qlUUaNoTER90XXWADtKne6VsYoD3ZnHGFXvPkRAQLM5d65ScBzWungcbLwZGWtWf5T2NzQj0wDyquMRwwIAsFDFtAZWkXRfXeXrFY0irYUS9rIJDafyMRvBbSz1FwWG7RTQkILkwiC4B8W1KdS5d9EZ8JPhrXvPMvW509g0GhLlkBSbPBeRSUlAS2Kk6nY5i3m6fi1H9CP3Y_X-TzOjOTsxQA_1pdP5uubXPUh5YfJihXcgewO9XXiqGDuQn6wZ3hrF6HTlhNWGcSyQPKh1gEcmXWQlRENZMvYET-BuJEE7eKyM5vRhjNoYR3w",
         e:"AQAB"
      }
   ]
}
```

有关IDP端点的更多信息，请参见以下内容：

- [Okta](https://developer.okta.com/docs/api/resources/oidc#well-knownopenid-configuration)
- [KeyCloak](https://www.keycloak.org/docs/latest/securing_apps/index.html#other-openid-connect-libraries)
- [Auth0](https://auth0.com/docs/protocols/oidc/openid-connect-discovery)
- [connect2ID](https://connect2id.com/products/server/docs/api/discovery)
- [销售队伍](https://help.salesforce.com/articleView?id=remoteaccess_using_openid_discovery_endpoint.htm&type=5)
- [IBM OpenID连接](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_8.5.5/com.ibm.websphere.wlp.doc/ae/rwlp_oidc_endpoint_urls.html)


## JWT验证的时间差异补偿

有时，您可能会发现身份验证服务器和OpenSearch节点之间的时钟时间不是完全同步的。在这种情况下，即使是几秒钟，发行或接收JWT的系统可能会试图验证`nbf` （不是之前）和`exp` （到期）索赔，并且由于时间差异而无法验证用户。

默认情况下，安全性允许30秒的窗口来补偿服务器时钟时间之间可能的未对准。要为此功能设置自定义值并覆盖默认值，您可以添加`jwt_clock_skew_tolerance_seconds` 设置为`config.yml`：

```yml
http_authenticator:
  type: openid
  challenge: false
  config:
    subject_key: preferred_username
    roles_key: roles
    openid_connect_url: https://keycloak.example.com:8080/auth/realms/master/.well-known/openid-configuration
    jwt_clock_skew_tolerance_seconds: 20
```


## 获取公共钥匙

当IDP生成并签名JWT时，它必须将密钥的ID添加到JWT标头中。例如：

```
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "V-diposfUJIk5jDBFi_QRouiVinG5PowskcSWy5EuCo"
}
```

按照[OpenID连接规范](https://openid.net/specs/openid-connect-messages-1_0-20.html)， 这`kid` （密钥ID）是强制性的。如果IDP未能添加，则令牌验证无效`kid` 字段到JWT。

如果安全插件接收一个带有未知的JWT`kid`，它访问了IDP的`jwks_uri` 并检索所有可用的有效密钥。这些键被使用并缓存，直到通过检索另一个未知密钥ID触发刷新。


## 钥匙翻转和多个公共钥匙

安全插件可以一次维护多个有效的公共密钥。OpenID规范不允许公开密钥的有效期，因此，直到将其从IDP中的有效键列表中删除并且有效密钥列表已刷新为止。

如果您想滚动IDP中的钥匙，请遵循以下最佳实践：

- 在您的IDP中创建一个新的密钥对，并将新密钥比当前使用的密钥更高。

  您的IDP在旧密钥上使用此新键。

- 首次出现新的`kid` 在JWT中，安全插件将刷新键列表。

  在这一点上，旧密钥和新密钥都是有效的。用旧密钥签名的令牌也仍然有效。

- 当带有此键的最后一个JWT签名时，可以从IDP中删除旧键。

如果您必须立即更改公钥，则还可以先删除旧密钥，然后创建一个新密钥。在这种情况下，所有与旧钥匙签名的JWT立即变得无效。


## TLS设置

防止人-在-这-中间攻击，您应该使用TLS保护安全插件和IDP之间的连接。


### 启用TLS

使用以下参数启用TLS以连接到IDP：

```yml
config:
  openid_connect_idp:
    enable_ssl: <true|false>
    verify_hostnames: <true|false>
```

姓名| 描述
:--- | :---
`enable_ssl` | 是否使用TLS。默认值为false。
`verify_hostnames` | 是否验证IDP TLS证书的主机名。默认是正确的。


### 证书验证

要验证IDP的TLS证书，请配置IDP root CA的路径或根证书的内容：

```yml
config:
  openid_connect_idp:
    enable_ssl: true
    pemtrustedcas_filepath: /full/path/to/trusted_cas.pem
```

```yml
config:
  openid_connect_idp:
    enable_ssl: true
    pemtrustedcas_content: |-
      MIID/jCCAuagAwIBAgIBATANBgkqhkiG9w0BAQUFADCBjzETMBEGCgmSJomT8ixk
      ARkWA2NvbTEXMBUGCgmSJomT8ixkARkWB2V4YW1wbGUxGTAXBgNVBAoMEEV4YW1w
      bGUgQ29tIEluYy4xITAfBgNVBAsMGEV4YW1wbGUgQ29tIEluYy4gUm9vdCBDQTEh
      ...
```


| 姓名| 描述|
| :--- | :--- |
| `pemtrustedcas_filepath` | 绝对路径到包含IDP根CA的PEM文件。|
| `pemtrustedcas_content` | IDP的根CA内容。如果不能使用`pemtrustedcas_filepath` 设置。|


### TLS客户端身份验证

要使用TLS客户端身份验证，请配置PEM证书和私钥安全插件应发送用于TLS客户端身份验证（或其内容）的插件：

```yml
config:
  openid_connect_idp:
    enable_ssl: true
    pemkey_filepath: /full/path/to/private.key.pem
    pemkey_password: private_key_password
    pemcert_filepath: /full/path/to/certificate.pem
```

```yml
config:
  openid_connect_idp:
    enable_ssl: true
    pemkey_content: |-
      MIID2jCCAsKgAwIBAgIBBTANBgkqhkiG9w0BAQUFADCBlTETMBEGCgmSJomT8ixk
      ARkWA2NvbTEXMBUGCgmSJomT8ixkARkWB2V4YW1wbGUxGTAXBgNVBAoMEEV4YW1w
      bGUgQ29tIEluYy4xJDAiBgNVBAsMG0V4YW1wbGUgQ29tIEluYy4gU2lnbmluZyBD
    ...
    pemkey_password: private_key_password
    pemcert_content: |-
      MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQCHRZwzwGlP2FvL
      oEzNeDu2XnOF+ram7rWPT6fxI+JJr3SDz1mSzixTeHq82P5A7RLdMULfQFMfQPfr
      WXgB4qfisuDSt+CPocZRfUqqhGlMG2l8LgJMr58tn0AHvauvNTeiGlyXy0ShxHbD
    ...
```

姓名| 描述
:--- | :---
`enable_ssl_client_auth` | 是否将客户端证书发送到IDP服务器。默认值为false。
`pemcert_filepath` | 绝对通往客户端证书的路径。
`pemcert_content` | 客户证书的内容。不能使用`pemcert_filepath` 设置。
`pemkey_filepath` | 包含客户端证书的私钥的文件的绝对路径。
`pemkey_content` | 客户端证书的私钥内容。不能使用`pemkey_filepath` 设置。
`pemkey_password` | 私钥的密码（如果有）。


### 启用密码和协议

您可以使用以下键限制允许的密码和TLS协议。

姓名| 描述
:--- | :---
`enabled_ssl_ciphers` | 大批。启用了TLS密码套件。仅支持Java格式。
`enabled_ssl_protocols` | 大批。启用了TLS协议。仅支持Java格式。


## （高级）DOS保护

帮助防止否认-的-服务（DOS）攻击，安全插件仅允许在一定时间内允许最大数量的新密钥ID。如果新密钥ID的数量超过此阈值，则安全插件将返回HTTP状态代码503（服务不可用），并拒绝查询IDP。默认情况下，安全插件在10秒内不允许超过10个未知的密钥ID。下表显示了如何修改这些设置。

姓名| 描述
:--- | :---
`refresh_rate_limit_count` | 时间范围内未知密钥ID的最大数量。默认值为10。
`refresh_rate_limit_time_window_ms` | 以毫秒为单位检查最大未知密钥ID的时间范围。默认值为10000（10秒）。


## OpenSearch仪表板单个标志-在

通过将以下添加到`opensearch_dashboards.yml`：

```
opensearch_security.auth.type: "openid"
```


### 配置

OpenID Connect提供商通常以 *元数据URL *以JSON格式发布其配置。因此，大多数设置可以自动拉动，因此OpenSearch仪表板配置变得最小。最重要的设置是：

- [连接URL](#openid-connect-url)
- 客户ID

  每个IDP都可以托管具有不同设置和身份验证协议的多个客户端（有时称为应用程序）。启用OpenID连接时，您应该在IDP中为OpenSearch仪表板创建一个新客户端。客户端ID唯一标识OpenSearch仪表板。

- 客户秘密

  除了ID之外，每个客户端还分配了一个客户端秘密。客户秘密通常是在创建客户端时生成的。申请只有在提供客户秘密时才能获得身份令牌。您可以在IDP的客户端的设置中找到此秘密。


### 配置设置

姓名| 描述
:--- | :---
`opensearch_security.openid.connect_url` | IDP发布OpenID元数据的URL。必需的。
`opensearch_security.openid.client_id` | IDP中配置的OpenID连接客户端的ID。必需的。
`opensearch_security.openid.client_secret` | IDP中配置的OpenID Connect客户端的客户端秘密。必需的。
`opensearch_security.openid.scope` | 这[身份令牌的范围](https://openid.net/specs/openid-connect-messages-1_0-20.html#scopes) 由IDP发行。选修的。默认为`openid profile email address phone`。
`opensearch_security.openid.header` | JWT令牌的HTTP标题名称。选修的。默认为`Authorization`。
`opensearch_security.openid.logout_url` | IDP的注销网址。选修的。仅当您的IDP未在其元数据中发布注销URL时才需要。
`opensearch_security.openid.base_redirect_url` | 将发送到您的IDP的重定向URL的底座。选修的。仅当OpenSearch仪表板在反向代理后面时才需要`server.host` 和`server.port` 在`opensearch_dashboards.yml`。
`opensearch_security.openid.trust_dynamic_headers` | 计算`base_redirect_url` 从反向代理HTTP标头（`X-Forwarded-Host` /`X-Forwarded-Proto`）。选修的。默认为`false`。


### 配置示例

```yml
# Enable OpenID authentication
opensearch_security.auth.type: "openid"

# The IdP metadata endpoint
opensearch_security.openid.connect_url: "http://keycloak.example.com:8080/auth/realms/master/.well-known/openid-configuration"

# The ID of the OpenID Connect client in your IdP
opensearch_security.openid.client_id: "opensearch-dashboards-sso"

# The client secret of the OpenID Connect client
opensearch_security.openid.client_secret: "a59c51f5-f052-4740-a3b0-e14ba355b520"

# Use HTTPS instead of HTTP
opensearch.url: "https://<hostname>.com:<http port>"

# Configure the OpenSearch Dashboards internal server user
opensearch.username: "kibanaserver"
opensearch.password: "kibanaserver"

# Disable SSL verification when using self-signed demo certificates
opensearch.ssl.verificationMode: none

# allowlist basic headers and multi-tenancy header
opensearch.requestHeadersAllowlist: ["Authorization", "security_tenant"]
```

在仪表板标志中与其他身份验证类型一起包含OpenID连接-在窗口中，请参阅[配置标志-在选项中]({{site.url}}{{site.baseurl}}/security/configuration/multi-auth/)。
{:.note}

### 其他参数

一些身份提供者需要自定义参数来完成身份验证过程。您可以添加自定义参数`opensearch_dashboards.yml` 配置文件`opensearch_security.openid.additional_parameters` 名称空间。您可以通过向您的身份提供商提出请求来找到这些其他参数。此功能在与各种身份提供者进行交流时可以更大的灵活性和自定义。

在下面的示例中，两个自定义参数，`foo` 和`acr_values`，及其价值观，`bar` 和`1`，发现使用get请求对OpenID提供商的请求：

```yml
opensearch_security.openid.additional_parameters.foo: "bar"
opensearch_security.openid.additional_parameters.acr_values: "1"
```
{% include copy.html %}



#### 会话管理以及其他cookie

改善会话管理---特别是对于分配了多个角色的用户---仪表板提供了将cookie有效载荷分为多个cookie的选项，然后在接收时重组有效载荷。这可以有助于防止更大的OpenID连接断言超过每个cookie的尺寸限制。以下示例中的两个设置允许您为其他cookie设置一个前缀名称，并指定其中的数量。他们被添加到`opensearch_dashboards.yml` 文件。其他cookie的默认数量为三：

```yml
opensearch_security.openid.extra_storage.cookie_prefix: security_authentication_oidc
opensearch_security.openid.extra_storage.additional_cookies: 3
```

请注意，减少其他cookie的数量可能会导致一些在更改之前使用的cookie停止工作。我们建议建立固定数量的其他cookie，然后不更改配置。

如果来自IDP的ID令牌特别大，OpenSearch可能会丢弃服务器日志身份验证错误，以表明HTTP标头太大。在这种情况下，您可以增加`http.max_header_size` 设置在`opensearch.yml` 文件。
{: .tip }


### OpenSearch安全配置

因为OpenSearch仪表板要求内部OpenSearch dashboards Server用户可以通过HTTP基本身份验证进行身份验证，因此您必须配置两个身份验证域。对于OpenID Connect，HTTP基本域必须首先放在链中。确保将挑战旗设置为`false`。

修改并应用以下示例设置`config.yml`：

```yml
basic_internal_auth_domain:
  http_enabled: true
  transport_enabled: true
  order: 0
  http_authenticator:
    type: basic
    challenge: false
  authentication_backend:
    type: internal
openid_auth_domain:
  http_enabled: true
  transport_enabled: true
  order: 1
  http_authenticator:
    type: openid
    challenge: false
    config:
      subject_key: preferred_username
      roles_key: roles
      openid_connect_url: https://keycloak.example.com:8080/auth/realms/master/.well-known/openid-configuration
  authentication_backend:
    type: noop
```

