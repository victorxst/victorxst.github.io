---
layout: default
title: SAML
parent: 身份验证后端
nav_order: 55
redirect_from:
  - /security/configuration/saml/
  - /security-plugin/configuration/saml/
---

# SAML

安全插件通过SAML单个符号支持用户身份验证-在。安全插件实现了SAML 2.0协议的Web浏览器SSO配置文件。

此配置文件旨在与Web浏览器一起使用。这不是一般-针对安全插件进行身份验证用户的目的方法，因此其主要用例是支持OpenSearch仪表板单个标志-在。


## Docker示例

我们提供了一个功能齐全的示例，可以帮助您了解如何将SAML与OpenSearch仪表板一起使用。

1. 下载[示例zip文件]({{site.url}}{{site.baseurl}}/assets/examples/saml-example-custom.zip) 到您的目录中的首选位置并解压缩。
1. 在命令行，指定目录中文件的位置并运行`docker-compose up`。
1. 查看文件：

   *`customize-docker-compose.yml`：定义两个OpenSearch节点，一个OpenSearch Dashboards Server和SAML Server。
   *`customize-opensearch_dashboards.yml`：包括默认设置的SAML设置`opensearch_dashboards.yml` 文件。
   *`customize-config.yml`：配置SAML用于身份验证。

   您可以删除"customize" 如果您打算修改并将这些文件保留进行生产，则从文件名中。
   {: .tip }

1. 在里面`docker-compose.yml` 文件，在`image` 节点1和2的字段，以及OpenSearch仪表板服务器。例如，如果您正在运行OpenSearch版本2.6，则`image` 字段将类似于以下示例：
   
   ```YML
   OpenSearch-SAML-Node1：
    图片：OpenSearchProject/OpenSearch：2.8.0
    ```
    ```YML
    OpenSearch-SAML-Node2：
    图片：OpenSearchProject/OpenSearch：2.8.0
    ```
    ```YML
    OpenSearch-SAML-仪表板：
    图片：OpenSearchProject/OpenSearch-仪表板：2.8.0
    ```

1. Access OpenSearch Dashboards at [http://localhost:5601](http://localhost:5601){:target='\_blank'}. Note that OpenSearch Dashboards immediately redirects you to the SAML login page.

1. Log in as `行政` with a password of `行政`.

1. After logging in, note that your user in the upper-right is `Samladmin`, as defined in `/var/www/simplesamlphp/config/authsources.php` SAML Server的。

1. 如果要检查SAML Server，请运行`docker ps` 找到其容器ID，然后`docker exec -it <container-id> /bin/bash`。

   特别是，您可能会发现查看内容的内容很有帮助`/var/www/simplesamlphp/config/` 和`/var/www/simplesamlphp/metadata/` 目录。


## 激活SAML

要使用SAML进行身份验证，您需要在`authc` 部分`config/opensearch-security/config.yml`。因为SAML仅在HTTP层上工作，所以您不需要任何`authentication_backend` 并可以将其设置为`noop`。将所有SAML放置-本章中的特定配置选项`config` SAML HTTP身份验证者的部分：

```yml
authc:
  saml_auth_domain:
    http_enabled: true
    transport_enabled: false
    order: 1
    http_authenticator:
      type: saml
      challenge: true
      config:
        idp:
          metadata_file: okta.xml
          ...
    authentication_backend:
      type: noop
```

在配置SAML之后`config.yml`，你也必须[在OpenSearch仪表板中激活它](#opensearch-dashboards-configuration)。


## 运行多个身份验证域

我们建议添加至少一个其他身份验证域，例如LDAP或内部用户数据库，以支持无需SAML的API访问Opensearch。对于OpenSearch仪表板和内部OpenSearch仪表板服务器用户，您还必须添加另一个支持基本身份验证的身份验证域。该身份验证域应首先放在链中，然后`challenge` 标志必须设置为`false`：

```yml
authc:
  basic_internal_auth_domain:
    http_enabled: true
    transport_enabled: true
    order: 0
    http_authenticator:
      type: basic
      challenge: false
    authentication_backend:
      type: internal
  saml_auth_domain:
    http_enabled: true
    transport_enabled: false
    order: 1
    http_authenticator:
      type: saml
      challenge: true
      config:
        ...
    authentication_backend:
      type: noop
```


## 身份提供者元数据

SAML身份提供商（IDP）提供了描述IDP功能和配置的SAML 2.0元数据文件。安全插件可以从URL或文件读取IDP元数据。您做出的选择取决于您的IDP和偏好。需要SAML 2.0元数据文件。

姓名| 描述
:--- | :---
`idp.metadata_file` | IDP的SAML 2.0元数据文件的路径。将元数据文件放在`config` OpenSearch目录。该路径必须相对于`config` 目录。如果需要`idp.metadata_url` 未设置。
`idp.metadata_url` | IDP的SAML 2.0元数据URL。如果需要`idp.metadata_file` 未设置。


## IDP和服务提供商实体ID

实体ID是SAML实体的全球唯一名称，即IDP或服务提供商（SP）。IDP实体ID通常由您的IDP提供。SP实体ID是IDP中配置的应用程序或客户端的名称。我们建议添加一个用于OpenSearch仪表板的新应用程序，并使用OpenSearch仪表板安装的URL作为SP实体ID。

姓名| 描述
:--- | :---
`idp.entity_id` | IDP的实体ID。必需的。
`sp.entity_id` | 服务提供商的实体ID。必需的。

## JWT验证的时间差异补偿

有时，您可能会发现身份验证服务器和OpenSearch节点之间的时钟时间不是完全同步的。在这种情况下，即使是几秒钟，发行或接收JSON Web令牌（JWT）的系统可能会尝试验证`nbf` （不是之前）和`exp` （到期）索赔，并且由于时间差异而无法验证用户。

默认情况下，OpenSearch Security允许使用30秒的窗口来补偿服务器时钟时间之间可能的未对准。要为此功能设置自定义值并覆盖默认值，您可以添加`jwt_clock_skew_tolerance_seconds` 设置为`config.yml`。

```yml
http_authenticator:
  type: saml
  challenge: true
  config:
    idp:
      metadata_file: okta.xml
    jwt_clock_skew_tolerance_seconds: 20
```

## OpenSearch仪表板设置

Web浏览器SSO配置文件通过HTTP GET或发布来交换信息。例如，登录到IDP后，它将HTTP帖子发送回包含SAML响应的OpenSearch仪表板。您必须配置发送HTTP请求的OpenSearch仪表板安装的基本URL。

姓名| 描述
:--- | :---
`kibana_url` | OpenSearch仪表板基础URL。必需的。


## 用户名和角色属性

受试者（例如，用户名）通常存储在`NameID` SAML响应的元素：

```
<saml2:Subject>
  <saml2:NameID>admin</saml2:NameID>
  ...
</saml2:Subject>
```

如果您的IDP符合SAML 2.0规范，则无需设置任何特殊内容。如果您的IDP使用其他元素名称，则还可以明确指定其名称。

角色属性是可选的。但是，大多数IDP也可以配置为在SAML断言中添加角色。如果存在，您可以在您的[角色映射]({{site.url}}{{site.baseurl}}/security/access-control/index/#concepts)：

```
<saml2:Attribute Name='Role'>
  <saml2:AttributeValue >Everyone</saml2:AttributeValue>
  <saml2:AttributeValue >Admins</saml2:AttributeValue>
</saml2:Attribute>
```

如果要从SAML响应中提取角色，则需要指定包含角色的元素名称。

姓名| 描述
:--- | :---
`subject_key` | SAML响应中存储主题的属性。选修的。如果未配置，`NameID` 使用属性。
`roles_key` | 在存储角色的SAML响应中的属性。选修的。如果未配置，则不使用任何角色。


## 请求签名

可以选择从安全插件到IDP的请求。使用以下设置配置请求签名。

姓名| 描述
:--- | :---
`sp.signature_private_key` | 用于签署请求或解码加密断言的私钥。选修的。不能使用`private_key_filepath` 设置。
`sp.signature_private_key_password` | 私钥的密码（如果有）。
`sp.signature_private_key_filepath` | 通往私钥的路径。该文件必须放在OpenSearch下`config` 目录和路径必须相对于同一目录指定。
`sp.signature_algorithm` | 该算法用于签署请求。有关可能的值，请参见下表。

私钥必须在PKC中#8格式。如果要使用加密密钥，则必须使用PKC进行加密#12-兼容算法（3DES）。

安全插件支持以下签名算法。

算法| 价值
:--- | :---
DSA_SHA1| http://www.w3.org/2000/09/xmldsig#DSA-sha1;
RSA_SHA1| http://www.w3.org/2000/09/xmldsig#RSA-sha1;
RSA_SHA256| http://www.w3.org/2001/04/xmldsig-更多的#RSA-SHA256;
RSA_SHA384| http://www.w3.org/2001/04/xmldsig-更多的#RSA-SHA384;
RSA_SHA512| http://www.w3.org/2001/04/xmldsig-更多的#RSA-SHA512;


## 登出

通常，IDP在其SAML 2.0元数据中提供有关其单独注销URL的信息。如果是这种情况，则安全插件使用它们在OpenSearch仪表板中呈现正确的注销链接。如果您的IDP不支持明确的注销，则可以强制RE-当用户再次访问OpenSearch仪表板时登录。

姓名| 描述
:--- | :---
`sp.forceAuthn` | 强迫-即使用户与IDP进行活动会话，登录也是如此。

当前，安全插件仅支持`HTTP-Redirect` 注销绑定。确保在IDP中正确配置这一点。


## 交换密钥设置

SAML与其他协议不同，并不是要与每个请求交换用户凭据。安全插件将SAML响应交易为轻量级JWT，该JWT存储经过验证的用户属性。这个令牌由您选择的交换键签名。请注意，当您更改此键时，所有令牌都会立即签名。

姓名| 描述
:--- | :---
`exchange_key` | 签署令牌的钥匙。该算法是HMAC256，因此它至少应具有32个字符。


## TLS设置

如果您是从URL加载IDP元数据，我们建议您使用SSL/TLS。如果使用使用受信任证书的OKTA或AUTH0（例如Okta或Auth0）的外部IDP，则通常不需要配置任何内容。如果您亲自托管IDP并使用自己的根CA，则可以按以下方式自定义TLS设置。这些设置仅用于通过HTTPS加载SAML元数据。

姓名| 描述
:--- | :---
`idp.enable_ssl` | 是否启用自定义TLS配置。默认值为false（使用JDK设置）。
`idp.verify_hostnames` | 是否验证服务器TLS证书的主机名。

例子：

```yml
authc:
  saml_auth_domain:
    http_enabled: true
    transport_enabled: false
    order: 1
    http_authenticator:
      type: saml
      challenge: true
      config:
        idp:
          enable_ssl: true
          verify_hostnames: true
          ...
    authentication_backend:
      type: noop
```


### 证书验证

配置用于通过设置验证IDP TLS证书的根CA**一** 以下配置选项：

```yml
config:
  idp:
    pemtrustedcas_filepath: path/to/trusted_cas.pem
```

```yml
config:
  idp:
    pemtrustedcas_content: |-
      MIID/jCCAuagAwIBAgIBATANBgkqhkiG9w0BAQUFADCBjzETMBEGCgmSJomT8ixk
      ARkWA2NvbTEXMBUGCgmSJomT8ixkARkWB2V4YW1wbGUxGTAXBgNVBAoMEEV4YW1w
      bGUgQ29tIEluYy4xITAfBgNVBAsMGEV4YW1wbGUgQ29tIEluYy4gUm9vdCBDQTEh
      ...
```

姓名| 描述
:--- | :---
`idp.pemtrustedcas_filepath` | 通往包含IDP根CA的PEM文件的路径。文件必须放在OpenSearch下`config` 目录，您必须指定相对于同一目录的路径。
`idp.pemtrustedcas_content` | IDP服务器的根CA内容。不能使用`pemtrustedcas_filepath` 设置。


### 客户端认证

安全插件在获取IDP元数据时可以使用TLS客户端身份验证。如果启用，安全插件将为每个元数据请求发送TLS客户端证书。使用以下键配置客户端身份验证。

姓名| 描述
:--- | :---
`idp.enable_ssl_client_auth` | 是否将客户端证书发送到IDP服务器。默认值为false。
`idp.pemcert_filepath` | 通往包含客户端证书的PEM文件的路径。该文件必须放在OpenSearch下`config` 目录，并且必须相对于`config` 目录。
`idp.pemcert_content` | 客户证书的内容。不能使用`pemcert_filepath` 设置。
`idp.pemkey_filepath` | 通往客户端证书的私钥的路径。该文件必须放在OpenSearch下`config` 目录，并且必须相对于`config` 目录。
`idp.pemkey_content` | 证书的私钥内容。不能使用`pemkey_filepath` 设置。
`idp.pemkey_password` | 私钥的密码（如果有）。


### 启用密码和协议

您可以限制IDP连接的允许的密码和TLS协议。例如，您只能启用强大的密码，并将TLS版本限制在最近的版本中。

姓名| 描述
:--- | :---
`idp.enabled_ssl_ciphers` | 启用的TLS密码数组。仅支持Java格式。
`idp.enabled_ssl_protocols` | 启用TLS协议的数组。仅支持Java格式。


## 最小配置示例
以下示例显示了最小配置：

```yml
authc:
  saml_auth_domain:
    http_enabled: true
    transport_enabled: false
    order: 1
    http_authenticator:
      type: saml
      challenge: true
      config:
        idp:
          metadata_file: metadata.xml
          entity_id: http://idp.example.com/
        sp:
          entity_id: https://opensearch-dashboards.example.com
        kibana_url: https://opensearch-dashboards.example.com:5601/
        roles_key: Role
        exchange_key: 'peuvgOLrjzuhXf ...'
    authentication_backend:
      type: noop
```

## OpenSearch仪表板配置

因为大多数saml-在安全插件中完成了特定的配置，只需激活您的SAML`opensearch_dashboards.yml` 通过添加以下内容：

```yml
opensearch_security.auth.type: "saml"
```

此外，您必须添加OpenSearch仪表板端点，以将SAML主张验证到您的允许列表：

```yml
server.xsrf.allowlist: ["/_opendistro/_security/saml/acs"]
```

如果您使用注销帖子绑定，则还需要将注销端点广告到您的允许列表：

```yml
server.xsrf.allowlist: ["/_opendistro/_security/saml/acs", "/_opendistro/_security/saml/logout"]
```

在仪表板标志中包括SAML与其他身份验证类型-在窗口中，请参阅[配置标志-在选项中]({{site.url}}{{site.baseurl}}/security/configuration/multi-auth/)。
{: .note }

#### 会话管理以及其他cookie

改善会话管理---特别是对于分配了多个角色的用户---仪表板提供了将cookie有效载荷分为多个cookie的选项，然后在接收时重组有效载荷。这可以帮助防止更大的SAML断言每个cookie的尺寸限制。以下示例中的两个设置允许您为其他cookie设置一个前缀名称，并指定其中的数量。他们被添加到`opensearch_dashboards.yml` 文件。其他cookie的默认数量为三：

```yml
opensearch_security.saml.extra_storage.cookie_prefix: security_authentication_saml
opensearch_security.saml.extra_storage.additional_cookies: 3
```

请注意，减少其他cookie的数量可能会导致一些在更改之前使用的cookie停止工作。我们建议建立固定数量的其他cookie，然后不更改配置。

如果来自IDP的ID令牌特别大，OpenSearch可能会丢弃服务器日志身份验证错误，以表明HTTP标头太大。在这种情况下，您可以增加`http.max_header_size` 设置在`opensearch.yml` 文件。
{: .tip }

### IDP-启动SSO

使用IDP-启动SSO，将IDP的主张消费者服务终点设置为：

```
/_opendistro/_security/saml/acs/idpinitiated
```

然后将此端点添加到`server.xsrf.allowlist` 在`opensearch_dashboards.yml`：

```yml
server.xsrf.allowlist: ["/_opendistro/_security/saml/acs/idpinitiated", "/_opendistro/_security/saml/acs", "/_opendistro/_security/saml/logout"]
```

