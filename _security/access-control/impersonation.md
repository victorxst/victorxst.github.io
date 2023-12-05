---
layout: default
title: 用户模仿
parent: 访问控制
nav_order: 100
redirect_from:
 - /security/access-control/impersonation/
 - /security-plugin/access-control/impersonation/
---

# 用户模仿

用户模仿允许特别特权用户充当另一个用户，而不了解或访问假冒用户的凭据。

模仿对于测试和故障排除或允许系统服务安全充当用户可能是有用的。

在休息界面或运输层上可能发生模拟。


## REST接口

要允许一个用户模仿另一个用户，请将以下内容添加到`opensearch.yml`：

```yml
plugins.security.authcz.rest_impersonation_user:
  <AUTHENTICATED_USER>:
    - <IMPERSONATED_USER_1>
    - <IMPERSONATED_USER_2>
```

模仿的用户领域支持通配符。设置为`*` 允许`AUTHENTICATED_USER` 冒充任何用户。


## 运输界面

以类似的方式，添加以下内容以使运输层模仿：

```yml
plugins.security.authcz.impersonation_dn:
  "CN=spock,OU=client,O=client,L=Test,C=DE":
    - worf
```


## 模仿用户

为了模拟另一个用户，请使用HTTP标头向系统提交请求`opendistro_security_impersonate_as` 设置为要模拟的用户名称。一个很好的测试是向`_plugins/_security/authinfo` URI：

```bash
curl -XGET -u 'admin:admin' -k -H "opendistro_security_impersonate_as: user_1" https://localhost:9200/_plugins/_security/authinfo?pretty
```

