---
layout: default
title: 对SAML进行故障排除
nav_order: 20
---

# SAML故障排除

此页面包括用于使用SAML进行OpenSearch仪表板身份验证的故障排除步骤。


---

#### Table of contents
- TOC
{:toc}


---

## 检查sp.entity_id

大多数身份提供商（IDP）允许您为不同应用程序配置多个身份验证方法。例如，在Okta中，这些客户被称为"Applications." 在KeyCloak中，它们被称为"Clients." 每个人都有自己的实体ID。确保配置`sp.entity_id` 匹配这些设置：

```yml
saml:
  ...
  http_authenticator:
    type: 'saml'
    challenge: true
    config:
      ...
      sp:
        entity_id: opensearch-dashboards-saml
```


## 检查SAML断言消费者服务URL

成功登录后，您的IDP使用HTTP帖子发送SAML响应到OpenSearch仪表板"assertion consumer service URL" （ACS）。

OpenSearch仪表板安全插件提供的端点是：

```
/_opendistro/_security/saml/acs
```

确保您在IDP中正确配置了此端点。一些IDP还要求您将所有端点添加到他们发送请求的允许列表中。确保列出ACS端点。

OpenSearch仪表板还要求您将此端点添加到允许列表中。确保您有以下条目`opensearch_dashboards.yml`：

```
server.xsrf.allowlist: [/_opendistro/_security/saml/acs]
```


## 签署所有文件

默认情况下，一些IDP不会签署SAML文档。确保IDP签名所有文档。


#### KeyCloak

![KeyCloak UI]({{site.url}}{{site.baseurl}}/images/saml-keycloak-sign-documents.png)


## 角色设置

在SAML响应中包括用户角色取决于您的IDP。例如，在KeyCloak中，此设置在**映射者** 客户部分。在OKTA中，您必须设置组属性语句。确保正确配置了，并且`roles_key` 在SAML配置中，SAML响应中的角色名称匹配：

```yml
saml:
  ...
  http_authenticator:
    type: 'saml'
    challenge: true
    config:
      ...
      roles_key: Role
```


## 检查SAML响应

如果您不确定IDP的SAML响应包含什么以及将用户名和角色放置在何处，则可以启用调试模式`log4j2.properties`：

```
logger.token.name = com.amazon.dlic.auth.http.saml.Token
logger.token.level = debug
```

此设置将打印对OpenSearch日志文件的SAML响应，以便您检查和调试。将此记录器设置为`debug` 生成许多语句，因此我们不建议在生产中使用它。

检查SAML响应的另一种方法是在登录OpenSearch仪表板时监视网络流量。IDP使用HTTP POST请求发送base64-编码的SAML响应：

```
/_opendistro/_security/saml/acs
```

检查该帖子请求的有效载荷，并使用类似的工具[base64decode.org](https://www.base64decode.org/) 解码。


## 检查角色映射

安全插件使用标准角色映射将用户或后端角色映射到一个或多个安全角色。

对于用户名，安全插件使用`NameID` 默认情况下，SAML响应的属性。对于某些IDP，此属性不包含预期的用户名，而是某些内部用户ID。检查SAML响应的内容以找到您要用作用户名的元素，并通过设置`subject_key`：

```yml
saml:
  ...
  http_authenticator:
    type: 'saml'
    challenge: true
    config:
      ...
      subject_key: preferred_username
```

要检查正确的后端角色是否包含在SAML响应中，请检查内容，并设置正确的属性名称：

```yml
saml:
  ...
  http_authenticator:
    type: 'saml'
    challenge: true
    config:
      ...
      roles_key: Role
```


## 检查JWT令牌

安全插件将SAML响应交易为更轻巧的JSON Web令牌。JWT中的用户名和后端角色最终映射到安全插件中的角色。如果映射存在问题，则可以使用与相同的设置启用令牌调试模式[检查SAML响应](#inspect-the-saml-response)。

此设置将JWT打印到OpenSearch日志文件，以便您可以使用类似的工具进行检查和调试[jwt.io](https://jwt.io/)。

