---
layout: default
title: 故障排除OpenID连接
nav_order: 30
---

# OpenID连接故障排除

此页面包括用于使用Security插件的OpenID连接的故障排除步骤。


---

#### Table of contents
- TOC
{:toc}


---

## 将日志级别设置为调试

要帮助对OpenID连接进行故障排除，请将日志级别设置为`debug` 在OpenSearch上。添加以下几行`config/log4j2.properties` 并重新启动节点：

```
logger.securityjwt.name = com.amazon.dlic.auth.http.jwt
logger.securityjwt.level = trace
```

此设置将打印出许多有用的信息到您的日志文件。如果此信息还不够，您也可以将日志级别设置为`trace`。


## "Failed when trying to obtain the endpoints from your IdP"

此错误表明安全插件无法达到IDP的元数据端点。在`opensearch_dashboards.yml`，检查以下设置：

```
plugins.security.openid.connect_url: "http://keycloak.example.com:8080/auth/realms/master/.well-known/openid-configuration"
```

如果此错误发生在OpenSearch上，请检查以下设置`config.yml`：

```yml
openid_auth_domain:
  enabled: true
  order: 1
  http_authenticator:
    type: "openid"
    ...
    config:
      openid_connect_url: http://keycloak.examplesss.com:8080/auth/realms/master/.well-known/openid-configuration
    ...
```

## "ValidationError: child 'opensearch_security' fails"

这表明缺少一个或多个OpenSearch仪表板配置设置。

查看`opensearch_dashboards.yml` 并确保您设置了以下最小配置：

```yml
plugins.security.openid.connect_url: "..."
plugins.security.openid.client_id: "..."
plugins.security.openid.client_secret: "..."
```


## "Authentication failed. Please provide a new token."

此错误具有几个潜在的根本原因。


### 剩下的饼干或缓存的凭据

请删除所有缓存的浏览器数据，或在私人浏览器窗口中重试。


### 客户秘密错误

为了将访问令牌换成身份令牌，大多数IDP都要求您提供客户秘密。检查客户是否秘密`opensearch_dashboards.yml` 匹配IDP配置的客户端秘密：

```
plugins.security.openid.client_secret: "..."
```


### "Failed to get subject from JWT claims"

此错误已在OpenSearch上记录，这意味着无法从ID令牌中提取用户名。确保以下设置与JWT中的索赔匹配您的IDP问题：

```
openid_auth_domain:
  enabled: true
  order: 1
  http_authenticator:
    type: "openid"
    ...
    config:
      subject_key: <subject key>
    ...
```

### "Failed to get roles from JWT claims with roles_key"

此错误表明您在`config.yml` 您的IDP发出的JWT中不存在。确保以下设置与JWT中的索赔匹配您的IDP问题：

```
openid_auth_domain:
  enabled: true
  order: 1
  http_authenticator:
    type: "openid"
    ...
    config:
      roles_key: <roles key>
    ...
```

