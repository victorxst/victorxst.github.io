---
layout: default
title: 基于代理的身份验证
parent: 身份验证后端
nav_order: 65
redirect_from:
 - /security-plugin/configuration/proxy/
---

# 基于代理的身份验证

如果您已经有一个标志-在（SSO）解决方案上，您可能需要将其用作身份验证后端。

大多数解决方案在OpenSearch和Security插件面前充当代理。如果代理身份验证成功，则代理将在HTTP标头字段中添加（经过验证的）用户名及其（经过验证的）角色。这些字段的名称取决于您已经存在的SSO解决方案。

然后，安全插件从请求中提取这些HTTP标头字段，并使用值来确定用户的权限。


## 启用代理检测

要启用OpenSearch的代理检测，请在`xff` 部分`config.yml`：

```yml
---
_meta：
  类型："config"
  config_version：2

配置：
  动态的：
    http：
      anonymous_auth_enabled：false
      XFF：
        启用：true
        internessproxies：'192 \ .168 \ .0 \ .10|192 \ .168 \ .0 \ .11'
        遥不可说：'x-转发-为了'
```

您可以配置以下设置：

姓名| 描述
：--- | ：---
`enabled` | 启用或禁用代理支持。默认值为false。
`internalProxies` | 包含所有受信任代理的IP地址的正则表达式。图案`.*` 信任所有内部代理。
`remoteIpHeader` | 具有主机名链的HTTP标头字段的名称。默认为`x-forwarded-for`。

为了确定请求是否来自受信任的内部代理，安全插件将HTTP请求的远程地址与已配置的内部代理列表进行比较。如果远程地址不在列表中，则插件将请求像客户端请求一样对待。


## Enable proxy authentication

配置携带身份验证的用户名和角色的HTTP标头字段的名称`proxy` HTTP身份验证器部分：

```yml
proxy_auth_domain：
  http_enabled：true
  transport_enabled：true
  订单：0
  http_authenticator：
    类型：代理
    挑战：错误
    配置：
      USER_HEADER："x-proxy-user"
      Roles_header："x-proxy-roles"
  Authentication_backend：
    类型：noop
```

姓名| 描述
：--- | ：---
`user_header` | HTTP标头字段包含已认证的用户名。默认为`x-proxy-user`。
`roles_header` | 包含逗号的HTTP标头字段-分离的身份验证的角色名称列表。该安全插件将在此标头字段中找到的角色作为后端角色。默认为`x-proxy-roles`。
`roles_separator` | 角色分离器。默认为`,`。


## Enable extended proxy authentication

安全插件具有扩展版本的`proxy` 键入可让您传递其他用户属性以供文档使用-级别的安全性。除了`type: extended-proxy` 和`attr_header_prefix`，配置是相同的：

```yml
proxy_auth_domain：
  http_enabled：true
  transport_enabled：true
  订单：0
  http_authenticator：
    类型：扩展-代理人
    挑战：错误
    配置：
      USER_HEADER："x-proxy-user"
      Roles_header："x-proxy-roles"
      attr_header_prefix："x-proxy-ext-"
  Authentication_backend：
    类型：noop
```

姓名| 描述
：--- | ：---
`attr_header_prefix` | 代理用来提供用户属性的标头前缀。例如，如果代理提供`x-proxy-ext-namespace: my-namespace`， 使用`${attr.proxy.namespace}` 在文档中-级别的安全查询。


## Example

以下示例在三个前面使用nginx代理-节点OpenSearch集群。为简单起见，我们使用硬编码值`x-proxy-user` 和`x-proxy-roles`。在现实世界中，您将动态设置这些标题。该示例还包括一个评论的标题，可与扩展代理一起使用。

```
事件{
  Worker_connections 1024;
}

http {

  上游OpenSearch {
    服务器node1.example.com:9200;
    服务器node2.com:9200;
    服务器node3.3200;
    保存15;
  }

  服务器 {
    听8090；
    server_name nginx.example.com;

    地点 / {
      proxy_pass https：// opensearch;
      proxy_set_header x-转发-对于$ proxy_add_x_forwarded_for;
      proxy_set_header x-代理人-用户测试；
      proxy_set_header x-代理人-角色测试；
      #proxy_set_header x-代理人-分机-命名空间-名称空间；
    }
  }

}
```

相应的最小值`config.yml` 好像：

```yml
---
_meta:
  type: "config"
  config_version: 2

config:
  dynamic:
    http:
      xff:
        enabled: true
        internalProxies: '172.16.0.203' # the nginx proxy
    authc:
      proxy_auth_domain:
        http_enabled: true
        transport_enabled: true
        order: 0
        http_authenticator:
          type: proxy
          #type: extended-proxy
          challenge: false
          config:
            user_header: "x-proxy-user"
            roles_header: "x-proxy-roles"
            #attr_header_prefix: "x-proxy-ext-"
        authentication_backend:
          type: noop
```

重要的部分是启用`X-Forwarded-For (XFF)` 解决方案并正确设置内部代理的IP：

```yml
enabled: true
internalProxies: '172.16.0.203' # nginx proxy
```

在这种情况下，`nginx.example.com` 运行`172.16.0.203`，因此将此IP添加到内部代理列表中。确保设置`internalProxies` 至少IP地址数量，因此安全插件仅接受受信任IP的请求。


## OpenSearch仪表板代理身份验证

要与OpenSearch仪表板一起使用代理身份验证，最常见的配置是将代理放置在OpenSearch仪表板的前面，并让OpenSearch搜索仪表板将用户和角色标头传递到安全插件。

在这种情况下，HTTP调用的远程地址是OpenSearch仪表板的IP，因为它直接位于OpenSearch的前面。将OpenSearch仪表板的IP添加到内部代理列表中：

```yml
---
_meta：
  类型："config"
  config_version：2

配置：
  动态的：
    http：
      XFF：
        启用：true
        遥控器："x-forwarded-for"
        内部替代：'<OpenSearch-仪表板-IP-地址>'
```

要将身份验证代理从OpenSearch仪表板添加到安全插件的用户和角色标头，请将其添加到HTTP标头中`opensearch_dashboards.yml`：

```yml
opensearch.requestheadersallowlist：["securitytenant"，，，，"Authorization"，，，，"x-forwarded-for"，，，，"x-proxy-user"，，，，"x-proxy-roles"这是给出的
```

您还必须启用身份验证类型`opensearch_dashboards.yml`：

```yml
opensearch_security.auth.type："proxy"
opensearch_security.proxycache.user_header："x-proxy-user"
opensearch_security.proxycache.roles_header："x-proxy-roles"
```

