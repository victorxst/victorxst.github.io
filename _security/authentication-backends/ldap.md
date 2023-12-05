---
layout: default
title: 活动目录和 LDAP
parent: 身份验证后端
nav_order: 60
redirect_from:
  - /security/configuration/ldap/
  - /security-plugin/configuration/ldap/
---

# Active Directory和LDAP

Active Directory和LDAP可用于身份验证和授权（`authc` 和`authz` 配置的部分）。身份验证检查用户是否已输入有效的凭据。授权检索用户的任何后端角色。

在大多数情况下，您需要配置身份验证和授权。您还可以仅使用身份验证，并将从LDAP检索到的用户直接映射到安全插件角色。


## Docker示例

我们提供了一个功能齐全的示例，可以帮助您了解如何使用LDAP服务器进行身份验证和授权。

1. 下载并解压缩[示例zip文件]({{site.url}}{{site.baseurl}}/assets/examples/ldap-example.zip)。
1. 在命令行，运行`docker-compose up`。
1. 查看文件：

   *`docker-compose.yml` 定义单个OpenSearch节点，LDAP服务器和LDAP服务器的PHP管理工具。

     您可以通过https：// localhost访问管理工具：6443。确认安全警告并登录使用`cn=admin,dc=example,dc=org` 和`changethis`。

   *`directory.ldif` 用三个用户和两个组播种LDAP服务器。

     `psantos` 在里面`Administrator` 和`Developers` 组。`jroe` 和`jdoe` 在`Developers` 团体。安全插件将这些组作为后端角色加载。

   *`roles_mapping.yml` 地图`Administrator` 和`Developers` LDAP组（作为后端角色）担任安全角色，以便用户在身份验证后获得适当的权限。

   *`internal_users.yml` 除了`administrator` 和`kibanaserver`。

   *`config.yml` 包括所有必要的LDAP设置。

1. 索引文档为`psantos`：

   ```bash
   curl -XPUT 'https://localhost:9200/new-index/_doc/1' -H 'Content-Type: application/json' -d '{"title": "Spirited Away"}' -u 'psantos:password' -k
   ```

   如果您尝试与`jroe`， 它失败。这`Developers` 组映射到`readall`，`manage_snapshots`， 和`kibana_user` 角色，没有写入权限。

1. 搜索文档`jroe`：

   ```bash
   卷曲-XGet'https：// localhost：9200/new-索引/_search？漂亮'-u'Jroe：密码'-k
   ```

   This request succeeds, because the `开发人员` group is mapped to the `读取` role.

1. If you want to examine the contents of the various containers, run `Docker PS` to find the container ID and then `Docker Exec-它<容器-id> /bin /bash`.


## Connection settings

To enable LDAP authentication and authorization, add the following lines to `config/opensearch-安全/config.yml`:

```yml
authc:
  ldap:
    http_enabled: true
    transport_enabled: true
    order: 1
    http_authenticator:
      type: basic
      challenge: false
    authentication_backend:
      type: ldap
      config:
        ...
```

```yml
authz:
  ldap:
    http_enabled: true
    transport_enabled: true
    authorization_backend:
      type: ldap
      config:
      ...
```

The connection settings are identical for authentication and authorization and are added to the `config` 部分。


### 主机名和端口

要配置Active Directory服务器的主机名和端口，请使用以下内容：

```yml
config:
  hosts:
    - primary.ldap.example.com:389
    - secondary.ldap.example.com:389
```

您可以在此处配置多个服务器。如果安全插件无法连接到第一台服务器，则试图依次连接到其余服务器。


### 超时

要将连接和响应超时配置为Active Directory Server，请使用以下（值为毫秒）：

```yml
config:
  connect_timeout: 5000
  response_timeout: 0
```

如果您的服务器支持两个-因子身份验证（2FA），默认超时设置可能会导致登录错误。您可以增加`connect_timeout` 适应2FA过程。环境`response_timeout` 到0（默认值）表示无限期的等待时间。


### 绑定DN和密码

配置`bind_dn` 和`password` 安全插件在向您的服务器发出查询时使用，请使用以下内容：

```yml
config:
  bind_dn: cn=admin,dc=example,dc=com
  password: password
```

如果您的服务器支持匿名身份验证，则`bind_dn` 和`password` 可以设置为`null`。


### TLS设置

使用以下参数配置TLS以连接到您的服务器：

```yml
config:
  enable_ssl: <true|false>
  enable_start_tls: <true|false>
  enable_ssl_client_auth: <true|false>
  verify_hostnames: <true|false>
```

姓名| 描述
:--- | :---
`enable_ssl` | 是否在SSL（LDAP）上使用LDAP。
`enable_start_tls` | 是否使用StartTL。不能与LDAP结合使用。
`enable_ssl_client_auth` | 是否将客户端证书发送到LDAP服务器。
`verify_hostnames` | 是否验证服务器TLS证书的主机名。


### 证书验证

默认情况下，安全插件验证了LDAP服务器的TLS证书，该证书与配置在`opensearch.yml`，作为PEM证书或信托基地：

```
plugins.security.ssl.transport.pemtrustedcas_filepath: ...
plugins.security.ssl.http.truststore_filepath: ...
```

如果您的服务器使用其他CA签名的证书，请将此CA导入您的信托店，或将其添加到每个节点上的受信任的CA文件中。

您还可以通过设置以下配置选项之一，以PEM格式使用单独的根CA：

```yml
config:
  pemtrustedcas_filepath: /full/path/to/trusted_cas.pem
```

```yml
config:
  pemtrustedcas_content: |-
    MIID/jCCAuagAwIBAgIBATANBgkqhkiG9w0BAQUFADCBjzETMBEGCgmSJomT8ixk
    ARkWA2NvbTEXMBUGCgmSJomT8ixkARkWB2V4YW1wbGUxGTAXBgNVBAoMEEV4YW1w
    bGUgQ29tIEluYy4xITAfBgNVBAsMGEV4YW1wbGUgQ29tIEluYy4gUm9vdCBDQTEh
    ...
```


姓名| 描述
:--- | :---
`pemtrustedcas_filepath` | 绝对路径到包含Active Directory/LDAP服务器的root CAS的PEM文件。
`pemtrustedcas_content` | Active Directory/LDAP服务器的根CA内容。不能使用`pemtrustedcas_filepath` 设置。


### 客户端认证

如果使用TLS客户端身份验证，则安全插件将发送节点的PEM证书，如在`opensearch.yml`。设置以下配置选项之一：

```yml
config:
  pemkey_filepath: /full/path/to/private.key.pem
  pemkey_password: private_key_password
  pemcert_filepath: /full/path/to/certificate.pem
```

或者

```yml
config:
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
`pemkey_filepath` | 包含证书的私钥的文件的绝对路径。
`pemkey_content` | 证书的私钥内容。不能使用`pemkey_filepath` 设置。
`pemkey_password` | 私钥的密码（如果有）。
`pemcert_filepath` | 绝对通往客户端证书的路径。
`pemcert_content` | 客户证书的内容。不能使用`pemcert_filepath` 设置。


### 启用密码和协议

您可以限制LDAP连接的允许的密码和TLS协议。例如，您只能允许强烈的密码，并将TLS版本限制在最近的版本中：

```yml
ldap:
  http_enabled: true
  transport_enabled: true
  ...
  authentication_backend:
    type: ldap
    config:
      enabled_ssl_ciphers:
        - "TLS_DHE_RSA_WITH_AES_256_CBC_SHA"
        - "TLS_DHE_DSS_WITH_AES_128_CBC_SHA256"
      enabled_ssl_protocols:
        - "TLSv1.1"
        - "TLSv1.2"
```

姓名| 描述
:--- | :---
`enabled_ssl_ciphers` | 数组，启用了TLS密码。仅支持Java格式。
`enabled_ssl_protocols` | 数组，启用的TLS协议。仅支持Java格式。


---

## Use Active Directory and LDAP for authentication

要使用Active Directory/LDAP进行身份验证，请首先在`authc` 部分`config/opensearch-security/config.yml`：

```yml
authc：
  LDAP：
    http_enabled：true
    transport_enabled：true
    订单：1
    http_authenticator：
      类型：基本
      挑战：是的
    Authentication_backend：
      类型：LDAP
      配置：
        ...
```

接下来，添加[连接设置]（#联系-设置））为Active Directory/LDAP服务器到身份验证域的配置部分：

```yml
配置：
  enable_ssl：true
  enable_start_tls：false
  enable_ssl_client_auth：false
  verify_hostnames：true
  主持人：
    - ldap.example.com:8389
  bind_dn：cn = admin，dc =示例，dc = com
  密码：passw0rd
```

身份验证通过发出包含LDAP树用户子树的LDAP查询来起作用。

安全插件首先采用配置的LDAP查询并替换占位符`{0}` 使用用户凭据的用户名。

```yml
usersearch：'（samaccountname = {0}）'
```

然后，它针对用户子树发出此查询。当前，配置的整个子树`userbase` 被搜索：

```yml
用户群：'OU =人，DC =示例，dc = com'
```

如果查询成功，则安全插件将从LDAP条目中检索用户名。您可以从安全插件应用作用户名的LDAP条目中指定哪些属性：

```yml
username_attribute：uid
```

如果此键未设置或空，则使用LDAP条目的特殊名称（DN）。


### Configuration summary

姓名| 描述
:--- | :---
`userbase` | 指定存储用户信息的目录中的子树。
`usersearch` | 安全插件尝试身份验证用户时执行的实际LDAP查询。变量{0}用用户名代替。
`username_attribute` | 安全插件使用目录条目的此属性来查找用户名。如果设置为null，则使用DN（默认值）。


### Complete authentication example

```yml
LDAP：
  http_enabled：true
  transport_enabled：true
  订单：1
  http_authenticator：
    类型：基本
    挑战：是的
  Authentication_backend：
    类型：LDAP
    配置：
      enable_ssl：true
      enable_start_tls：false
      enable_ssl_client_auth：false
      verify_hostnames：true
      主持人：
        - ldap.example.com:636
      bind_dn：cn = admin，dc =示例，dc = com
      密码：密码
      用户群：'OU =人，DC =示例，dc = com'
      usersearch：'（samaccountname = {0}）'
      username_attribute：uid
```


---

## 使用Active Directory和LDAP进行授权

要使用Active Directory/LDAP进行授权，请首先在`authz` 部分`config.yml`：

```yml
authz:
  ldap:
    http_enabled: true
    transport_enabled: true
    authorization_backend:
      type: ldap
      config:
      ...
```

授权是从LDAP服务器检索身份验证用户的后端角色的过程。这通常是您用于身份验证的服务器，但您也可以使用其他服务器。唯一的要求是，您用来获取角色的用户实际上存在于LDAP服务器上。

因为安全插件始终检查LDAP服务器中的用户是否存在，因此您还必须配置`userbase`，`usersearch` 和`username_attribute` 在里面`authz` 部分。

授权与身份验证类似。安全插件发布了包含用户名的LDAP查询，该查询针对LDAP树的角色子树。

作为替代方案，安全插件还可以获取定义为用户子树中用户条目的直接属性的角色。


### 方法1：查询角色子树

安全插件首先采用LDAP查询以获取角色（"rolesearch"）并替换查询中发现的任何变量。例如，对于标准的Active Directory安装，您将使用以下角色搜索：

```yml
rolesearch: '(member={0})'
```

您可以使用以下变量：

- `{0}` 用用户的DN替换。
- `{1}` 用用户名代替`username_attribute` 环境。
- `{2}` 从身份验证的用户目录条目中替换为任意属性值。

变量`{2}` 请参考用户目录条目中的属性。您应该使用的属性由`userroleattribute` 环境：

```yml
userroleattribute: myattribute
```

然后，安全插件对配置的角色子树发出替换的查询。整个子树下`rolebase` 被搜索：

```yml
rolebase: 'ou=groups,dc=example,dc=com'
```

如果您使用嵌套角色（角色的角色），则可以配置安全插件以解决它们：

```yml
resolve_nested_roles: false
```

在所有角色都得到了所有角色之后，安全插件从角色条目的可配置属性中提取了最终角色名称：

```yml
rolename: cn
```

如果未设置此功能，则使用角色输入的DN。您现在可以使用此角色名称将其映射到一个或多个安全插件角色，如在`roles_mapping.yml`。


### 方法2：使用用户的属性作为角色名称

如果将角色作为用户条目的直接属性存储在用户子树中，则需要仅配置属性名称：

```yml
userrolename: roles
```

您可以配置多个属性名称：

```yml
userrolename: roles, otherroles
```

该方法可以与查询角色子树的查询结合使用。安全插件从用户的角色属性中获取角色，然后执行角色搜索。

如果您不使用或有角色子树，则可以完全禁用角色搜索：

```yml
rolesearch_enabled: false
```


### （高级）控制LDAP用户属性

默认情况下，安全插件读取所有LDAP用户属性，并使其可用于索引名称变量替换和DLS查询变量替换。如果您的LDAP条目具有很多属性，则可能需要控制应提供哪些属性。属性越少，性能就越好。

请注意，此设置是在身份验证中进行的`authc` config.yml文件的部分。

姓名| 描述
:--- | :---
`custom_attr_allowlist`  | 字符串数组。指定应可用于可变替换的LDAP属性。
`custom_attr_maxval_len`  | 整数。指定每个属性的最大允许长度。所有属性更长的属性都被丢弃。一个值`0` 完全禁用自定义属性。默认值为36。

例子：

```yml
authc:
  ldap:
    http_enabled: true
    transport_enabled: true
    authentication_backend:
      type: ldap
      config:
        custom_attr_allowlist:
          - attribute1
          - attribute2
        custom_attr_maxval_len: 36
      ...
```


### （高级）将某些用户排除在角色查找之外

如果您使用的是多种身份验证方法，则将某些用户从LDAP角色查找中排除是有意义的。

考虑典型的OpenSearch仪表板设置的以下方案：所有OpenSearch Dashboards用户都存储在LDAP/Active Directory Server中。

但是，您还拥有OpenSearch仪表板服务器用户。OpenSearch仪表板使用此用户来管理存储的对象并执行监视和维护任务。您不想将此用户添加到Active Directory安装中，而是将其存储在“安全插件”内部用户数据库中。

在这种情况下，将OpenSearch仪表板服务器用户从LDAP授权中排除是有意义的，因为我们已经知道没有相应的条目。您可以使用`skip_users` 配置设置以定义应跳过哪些用户。支持通配符和正则表达式：

```yml
skip_users:
  - kibanaserver
  - 'cn=Jane Doe,ou*people,o=TEST'
  - '/\S*/'
```


### （高级）将角色排除在嵌套角色查找之外

如果LDAP安装中的用户具有大量角色，并且您还需要解决嵌套角色，那么您可能会遇到性能问题。

但是，在大多数情况下，并非所有用户角色都与Opensearch和OpenSearch仪表板有关。您可能只需要几个角色。在这种情况下，您可以使用嵌套的角色过滤器功能来定义从用户角色列表中滤除的角色列表。支持通配符和正则表达式。

只有在`resolve_nested_roles` 是`true`：

```yml
nested_role_filter:
  - 'cn=Jane Doe,ou*people,o=TEST'
  - ...
```


### 配置摘要

姓名| 描述
:--- | :---
`rolebase`  | 在存储角色/组信息的目录中指定子树。
`rolesearch` | 安全插件在尝试确定用户角色时执行的实际LDAP查询。您可以在此处使用三个变量（见下文）。
`userroleattribute`  | 用户条目中的属性用于使用`{2}` 可变替代。
`userrolename`  | 如果用户的角色/组未存储在组子树中，而是作为用户目录条目的属性，请在此处定义此属性名称。
`rolename`  | 角色输入的属性应用作角色名称。
`resolve_nested_roles`  | 布尔。是否解决嵌套角色。默认为`false`。
`max_nested_depth`  | 整数。什么时候`resolve_nested_roles` 是`true`，这定义了嵌套角色的最大数量。设置较小的值可以减少从LDAP检索到的数据量，并以未能发现深嵌套角色的成本来改善身份验证时间。默认为`30`。
`skip_users`  | 取回角色时应该跳过的用户阵列。支持通配符和正则表达式。
`nested_role_filter`  | 在解决嵌套角色之前应过滤的角色DN阵列。支持通配符和正则表达式。
`rolesearch_enabled`  | 布尔。启用或禁用角色搜索。默认为`true`。
`custom_attr_allowlist`  | 字符串数组。指定应可用于可变替换的LDAP属性。
`custom_attr_maxval_len`  | 整数。指定每个属性的最大允许长度。所有属性更长的属性都被丢弃。一个值`0` 完全禁用自定义属性。默认值为36。


### 完整的授权示例

```yml
authz:
  ldap:
    http_enabled: true
    transport_enabled: true
    authorization_backend:
      type: ldap
      config:
        enable_ssl: true
        enable_start_tls: false
        enable_ssl_client_auth: false
        verify_hostnames: true
        hosts:
          - ldap.example.com:636
        bind_dn: cn=admin,dc=example,dc=com
        password: password
        userbase: 'ou=people,dc=example,dc=com'
        usersearch: '(uid={0})'
        username_attribute: uid
        rolebase: 'ou=groups,dc=example,dc=com'
        rolesearch: '(member={0})'
        userroleattribute: null
        userrolename: none
        rolename: cn
        resolve_nested_roles: true
        skip_users:
          - kibanaserver
          - 'cn=Jane Doe,ou*people,o=TEST'
          - '/\S*/'
```

### （高级）配置多个用户和角色库

要在Authc和/或Authz部分中配置多个用户基础，请使用以下语法：

```yml
        ...
        bind_dn: cn=admin,dc=example,dc=com
        password: password
        users:
          primary-userbase:
             base: 'ou=people,dc=example,dc=com'
             search: '(uid={0})'
          secondary-userbase:
             base: 'cn=users,dc=example,dc=com'
             search: '(uid={0})'
        username_attribute: uid
        ...
```

同样，使用以下设置在Authz部分中配置多个角色库：

```yml
        ...
        username_attribute: uid
        roles:
          primary-rolebase:
            base: 'ou=groups,dc=example,dc=com'
            search: '(uniqueMember={0})'
          secondary-rolebase:
            base: 'ou=othergroups,dc=example,dc=com'
            search: '(member={0})'
        userroleattribute: null
        ...
```

### 具有多个用户和角色库的完整身份验证和授权示例：

```yml
authc:
  ...
  ldap:
    http_enabled: true
    transport_enabled: true
    order: 1
    http_authenticator:
      type: basic
      challenge: true
    authentication_backend:
      type: ldap
      config:
        enable_ssl: true
        enable_start_tls: false
        enable_ssl_client_auth: false
        verify_hostnames: true
        hosts:
          - ldap.example.com:636
        bind_dn: cn=admin,dc=example,dc=com
        password: password
        users:
          primary-userbase:
             base: 'ou=people,dc=example,dc=com'
             search: '(uid={0})'
          secondary-userbase:
             base: 'cn=users,dc=example,dc=com'
             search: '(uid={0})'
        username_attribute: uid
authz:
  ldap:
    http_enabled: true
    transport_enabled: true
    authorization_backend:
      type: ldap
      config:
        enable_ssl: true
        enable_start_tls: false
        enable_ssl_client_auth: false
        verify_hostnames: true
        hosts:
          - ldap.example.com:636
        bind_dn: cn=admin,dc=example,dc=com
        password: password
        users:
          primary-userbase:
             base: 'ou=people,dc=example,dc=com'
             search: '(uid={0})'
          secondary-userbase:
             base: 'cn=users,dc=example,dc=com'
             search: '(uid={0})'
        username_attribute: uid
        roles:
          primary-rolebase:
            base: 'ou=groups,dc=example,dc=com'
            search: '(uniqueMember={0})'
          secondary-rolebase:
            base: 'ou=othergroups,dc=example,dc=com'
            search: '(member={0})'
        userroleattribute: null
        userrolename: none
        rolename: cn
        resolve_nested_roles: true
```

