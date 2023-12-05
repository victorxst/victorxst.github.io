---
layout: default
title: JSON网络令牌
parent: 身份验证后端
nav_order: 47
redirect_from:
---


# JSON网络令牌

JSON Web令牌（JWTS）是JSON-主张一个或多个主张的基于访问令牌。它们通常用于实现单个标志-在（SSO）解决方案上，属于令牌类别-基于身份验证系统。基本信息-传输和身份-以下步骤描述了JWT的验证生命周期：

1. 用户通过提供凭据（例如，用户名和密码）来登录身份验证服务器。
1. 身份验证服务器验证凭据。
1. 身份验证服务器创建访问令牌并签名。
1. 身份验证服务器将令牌返回给用户。
1. 用户存储访问令牌。
1. 用户将访问令牌与每个请求一起发送到要使用的服务。
1. 该服务将验证令牌和赠款或拒绝访问权限。
1. 通过授予访问权限，用户可以访问到令牌的到期时间。到期时间通常由发行人在令牌有效载荷中设置。

JWT是自我的-从某种意义上说，它包含在其中验证用户所需的所有信息。令牌是base64-编码，签名的JSON对象。


## JWT元素

JWT由三个部分组成：

*标题
*有效载荷
* 签名


### 标题

标题包含有关正在使用的签名机制的信息，包括用于编码令牌的算法。以下示例显示了标头的典型属性和值：

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

在这种情况下，标题指出该消息是使用Hashing算法HMAC签署的-SHA256。


### 有效载荷

JWT的有效载荷包含[JWT声称](https://auth0.com/docs/secure/tokens/json-web-tokens/json-web-token-claims)。索赔是有关令牌用户的一条信息，它用作唯一的标识符。这允许令牌的发行人验证身份。索赔是名字-价值对，有效载荷通常包括多个索赔。虽然增加索赔的选项很多，但避免添加太多并使有效载荷过大的习惯是一种很好的做法，这将使JWT紧凑的目的失败。

主张有三种类型：

*[注册索赔](https://www.iana.org/assignments/jwt/jwt.xhtml#claims) 由JWT规范定义，并包括一组带有保留名称的标准索赔。这些主张的一些示例包括令牌发行人（ISS），到期时间（EXP）和主题（Sub）。
*另一方面，公开索赔是在共享令牌的各方的意愿下定义的。它们可以包含任意信息，例如用户的用户名和角色。为了预防措施，该规范建议注册该名称，或者至少确保名称为[抵抗碰撞](https://www.rfc-editor.org/rfc/rfc7519#section-4.2) 与其他主张。
*私人索赔提供了将自定义信息分配给有效载荷的另一种选择：例如，电子邮件地址。因此，它们也称为_custom_索赔。共享令牌的双方必须同意他们的使用，因为他们既不是注册也不是公开索赔。

以下示例将这些JSON属性显示为名称-价值对：

```json
{
  "iss": "example.com",
  "exp": 1300819380,
  "name": "John Doe",
  "roles": "admin, devops"
}
```

### 签名

令牌的发行人通过将加密哈希函数应用于base64来生成令牌的签名-编码标头和有效载荷。接收JWT解密并在传输的最后一步中验证此签名的客户。

这三个部分---标题，有效载荷和签名---使用时期串联以形成完整的JWT：

```
encoded = base64UrlEncode(header) + "." + base64UrlEncode(payload)
signature = HMACSHA256(encoded, 'secretkey');
jwt = encoded + "." + base64UrlEncode(signature)
```

例子：
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJsb2dnZWRJbkFzIjoiYWRtaW4iLCJpYXQiOjE0MjI3Nzk2Mzh9.gzSraSYS8EXBxLN_oWnFSRgCzcmJmMjLiuyu5CSpyHI
```


## 配置JWT

如果使用JWT作为唯一的身份验证方法，请通过设置用户缓存来禁用用户缓存`plugins.security.cache.ttl_minutes` 财产为`0`。有关此属性的更多信息，请参阅[opensearch.yml]({{site.url}}{{site.baseurl}}/security/configuration/yaml/#opensearchyml)。
{: .important }

设置身份验证域并选择`jwt` 作为HTTP身份验证类型。由于令牌已经包含所有必需的信息来验证请求，所以`challenge` 必须设置为`false` 和`authentication_backend` 到`noop`：

```yml
jwt_auth_domain:
  http_enabled: true
  transport_enabled: true
  order: 0
  http_authenticator:
    type: jwt
    challenge: false
    config:
      signing_key: "base64 encoded key"
      jwt_header: "Authorization"
      jwt_url_parameter: null
      subject_key: null
      roles_key: null
      jwt_clock_skew_tolerance_seconds: 20
  authentication_backend:
I    type: noop
```

下表列出了配置参数。

姓名| 描述
:--- | :---
`signing_key` | 验证令牌时要使用的签名键。如果使用对称密钥算法，则是base64-编码共享的秘密。如果您使用不对称算法，则包含公共密钥。
`jwt_header` | 将令牌传输的HTTP标头。这通常是`Authorization` 标题与`Bearer` 模式：`Authorization: Bearer <token>`。默认为`Authorization`。
`jwt_url_parameter` | 如果代币不是在HTTP标头中传输的，而是在URL参数中传输，请在此处定义参数的名称。
`subject_key` | 存储用户名的JSON有效载荷中的密钥。如果未设置，[主题](https://tools.ietf.org/html/rfc7519#section-4.1.2) 使用注册索赔。
`roles_key` | JSON有效负载中存储用户角色的钥匙。该密钥的价值必须是逗号-分开的角色列表。
`jwt_clock_skew_tolerance_seconds` | 在几秒钟内设置一个时间窗口，以弥补JWT身份验证服务器和OpenSearch节点时钟时间之间的任何差异，从而防止由于未对准而导致身份验证故障。安全设置为30秒为默认值。使用此设置应用自定义值。

因为JWT是自我-包含并在HTTP级别进行身份验证，没有其他`authentication_backend` 需要。将此值设置为`noop`。


### 对称密钥算法：HMAC

哈希-基于消息身份验证代码（HMACS）是一组算法，可提供一种通过共享密钥签名消息的方法。密钥在身份验证服务器和安全插件之间共享。它必须配置为base64-编码值`signing_key` 环境：

```yml
jwt_auth_domain:
  ...
    config:
      signing_key: "a3M5MjEwamRqOTAxOTJqZDE="
      ...
```


### 非对称密钥算法：RSA和ECDSA

RSA和ECDA是不对称的加密和数字签名算法，使用公共/私钥对签名和验证令牌。这意味着他们使用私钥签名令牌，而安全插件只需要知道公共密钥来验证它。

因为您无法使用公钥发布新令牌---而且因为您可以对代币的创建者做出有效的假设---RSA和ECDA被认为比HMAC更安全。

要使用RS256，您需要仅配置（非-基础64-编码）公共RSA密钥作为`signing_key` 在JWT配置中：

```yml
jwt_auth_domain:
  ...
    config:
      signing_key: |-
        -----BEGIN PUBLIC KEY-----
        MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQK...
        -----END PUBLIC KEY-----
      ...
```

安全插件自动检测算法（RSA/ECDSA）。如有必要，您可以将密钥分解为多行。


### HTTP请求的载体身份验证

在HTTP请求中传输JWT的最常见方法是将其添加为带有载体身份验证模式的HTTP标头：

```
Authorization: Bearer <JWT>
```

标题的默认名称是`Authorization`。如果您的身份验证服务器或代理需要，您也可以使用其他HTTP标头名称使用`jwt_header` 配置密钥。

与HTTP基本身份验证一样，在HTTP请求中传输JWT时，应使用HTTP而不是HTTP。


### HTTP请求的URL参数

尽管在HTTP请求中传输JWT的最常见方法是使用标头字段，但安全插件也支持参数。配置名称`GET` 使用以下键参数：

```yml
    config:
      signing_key: ...
      jwt_url_parameter: "parameter_name"
      subject_key: ...
      roles_key: ...
```

与HTTP基本身份验证一样，您应该使用HTTP而不是HTTP。


### 经过验证的注册索赔

以下注册索赔会自动验证：

*"iat" （在）索赔
*"nbf" （不是以前）
*"exp" （到期时间）索赔


### 支持格式和算法

安全插件支持具有所有标准算法的数字签名，紧凑的JWT：

```
HS256: HMAC using SHA-256
HS384: HMAC using SHA-384
HS512: HMAC using SHA-512
RS256: RSASSA-PKCS-v1_5 using SHA-256
RS384: RSASSA-PKCS-v1_5 using SHA-384
RS512: RSASSA-PKCS-v1_5 using SHA-512
PS256: RSASSA-PSS using SHA-256 and MGF1 with SHA-256
PS384: RSASSA-PSS using SHA-384 and MGF1 with SHA-384
PS512: RSASSA-PSS using SHA-512 and MGF1 with SHA-512
ES256: ECDSA using P-256 and SHA-256
ES384: ECDSA using P-384 and SHA-384
ES512: ECDSA using P-521 and SHA-512
```


## 使用JWKS端点验证JWT

验证签名JWT的签名是授予用户访问的最后一步。OpenSearch通过REST请求发送JWT时验证签名。在每个身份验证请求中验证签名。

而不是存储用于验证的加密密钥`config.yml` 文件的`authc` 部分，您可以指定JSON Web密钥集（JWKS）端点，以从发行者服务器上的位置检索键。这种验证JWT的方法可以帮助简化公共钥匙和证书的管理。

在OpenSearch中，这种验证方法利用了[OpenID连接身份验证域配置]({{site.url}}{{site.baseurl}}/security/authentication-backends/openid-connect/#configure-openid-connect-integration)。要指定JWKS端点，请替换`openid_connect_url` 用配置设置`jwks_uri` 设置并将URL添加到设置为其值。这在以下示例中显示：

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
      jwks_uri: https://keycloak.example.com:8080/auth/realms/master/.well-known/jwks-keys.json
  authentication_backend:
    type: noop
```

端点应由JWT发行人记录。您可以使用它来检索验证签名的JWT所需的密钥。有关JSON Web键的内容和格式的更多信息，请参见[JSON Web密钥（JWK）格式](https://datatracker.ietf.org/doc/html/rfc7517#section-4)。


## 解决常见问题

本节详细介绍了如何使用安全配置解决常见问题。


### 验证正确的索赔

确保JWT令牌包含正确的`iat` （发出于），`nbf` （不是之前），`exp` （有效期）索赔，所有这些索引都会自动验证。


### JWT URL参数

使用包含默认管理角色的JWT URL参数时`all_access` （例如，`curl http://localhost:9200?jwtToken=<jwt-token>`），请求失败并引发以下错误：

```json
{
   "error":{
      "root_cause":[
         {
            "type":"security_exception",
            "reason":"no permissions for [cluster:monitor/main] and User [name=admin, backend_roles=[all_access], requestedTenant=null]"
         }
      ],
      "type":"security_exception",
      "reason":"no permissions for [cluster:monitor/main] and User [name=admin, backend_roles=[all_access], requestedTenant=null]"
   },
   "status":403
}
```

要纠正这一点，请确保角色`all_access` 直接映射到内部用户，而不是后端角色。为此，导航到**安全>角色> all_access** 并选择**映射的用户** 标签。选择**管理映射** 并添加"admin" 到**用户** 部分。

![图像](https://user-images.githubusercontent.com/5849965/179158704-b2bd6d48-8816-4b03-a960-8c612465cf75.png)

然后，用户应出现在**映射的用户** 标签。

![图像](https://user-images.githubusercontent.com/5849965/179158750-1bb5e232-dd61-449a-a561-0613b71bfd68.png)


### OpenSearch仪表板配置

即使直接查询OpenSearch时，JWT URL参数身份验证有效，但用于访问OpenSearch仪表板时，它会失败。

**解决方案：** 确保存在以下几行`opensearch_dashboards.yml` 配置文件：

```yml
opensearch_security.auth.type: "jwt"
opensearch_security.jwt.url_param: <your-param-name-here>
```

