---
layout: default
title: 配置TLS证书
parent: 配置
nav_order: 15
redirect_from:
  - /security-plugin/configuration/tls/
---

# 配置TLS证书

TLS配置在`opensearch.yml`。证书用于确保运输-层流量（节点-到-群集中的节点通信）和休息-层流量（客户群和群集中的节点之间的通信）。TLS对于其余层是可选的，对于运输层是必不可少的。

您可以在所有选项上找到一个示例配置模板[github](https://github.com/opensearch-project/security/blob/main/config/opensearch.yml.example)。
{: .note}


## X.509 PEM证书和PKCS \#8个钥匙

以下表包含您可以使用的设置来配置PEM证书和私钥的位置。


### 运输层TLS

姓名| 描述
:--- | :---
`plugins.security.ssl.transport.pemkey_filepath` | 通往证书密钥文件的路径（PKCS \#8），必须在`config` 目录，使用相对路径指定。必需的。
`plugins.security.ssl.transport.pemkey_password` | 关键密码。如果密钥没有密码，则省略此设置。选修的。
`plugins.security.ssl.transport.pemcert_filepath` | X.509节点证书链（PEM格式）的路径，必须在`config` 目录，使用相对路径指定。必需的。
`plugins.security.ssl.transport.pemtrustedcas_filepath` | 通往根CA（PEM格式）的路径，必须在`config` 目录，使用相对路径指定。必需的。


### 休息层TLS

姓名| 描述
:--- | :---
`plugins.security.ssl.http.enabled` | 是否在其余层上启用TLS。如果启用，则仅允许HTTPS。选修的。默认为`false`。
`plugins.security.ssl.http.pemkey_filepath` | 通往证书密钥文件的路径（PKCS \#8），必须在`config` 目录，使用相对路径指定。必需的。
`plugins.security.ssl.http.pemkey_password` | 关键密码。如果密钥没有密码，则省略此设置。选修的。
`plugins.security.ssl.http.pemcert_filepath` | X.509节点证书链（PEM格式）的路径，必须在`config` 目录，使用相对路径指定。必需的。
`plugins.security.ssl.http.pemtrustedcas_filepath` | 通往根CA（PEM格式）的路径，必须在`config` 目录，使用相对路径指定。必需的。


## 钥匙店和信托店文件

作为PEM格式的证书和私钥的替代方案，您可以使用JKS或PKCS12/PFX格式的密钥库和TrustStore文件。为了使安全插件运行，您需要证书和私钥。

以下设置配置了您的密钥库和信托店文件的位置和密码。如果需要，您可以在其余部分和运输层中使用不同的密钥库和信托店文件。


### 运输层TLS

姓名| 描述
:--- | :---
`plugins.security.ssl.transport.keystore_type` | 密钥库文件，JKS或PKCS12/PFX的类型。选修的。默认值为JK。
`plugins.security.ssl.transport.keystore_filepath` | 到达密钥库文件的路径，必须在`config` 目录，使用相对路径指定。必需的。
`plugins.security.ssl.transport.keystore_alias: my_alias` | 别名。选修的。默认值是第一个别名。
`plugins.security.ssl.transport.keystore_password` | 密钥店密码。默认为`changeit`。
`plugins.security.ssl.transport.truststore_type` | TrustStore文件，JKS或PKCS12/PFX的类型。默认值为JK。
`plugins.security.ssl.transport.truststore_filepath` | 通往信托店文件的路径，必须在`config` 目录，使用相对路径指定。必需的。
`plugins.security.ssl.transport.truststore_alias` | 别名。选修的。默认是所有证书。
`plugins.security.ssl.transport.truststore_password` | TrustStore密码。默认为`changeit`。


### 休息层TLS

姓名| 描述
:--- | :---
`plugins.security.ssl.http.enabled` | 是否在其余层上启用TLS。如果启用，则仅允许HTTPS。选修的。默认值为false。
`plugins.security.ssl.http.keystore_type` | 密钥库文件，JKS或PKCS12/PFX的类型。选修的。默认值为JK。
`plugins.security.ssl.http.keystore_filepath` | 到达密钥库文件的路径，必须在`config` 目录，使用相对路径指定。必需的。
`plugins.security.ssl.http.keystore_alias` | 别名。选修的。默认值是第一个别名。
`plugins.security.ssl.http.keystore_password` | 密钥店密码。默认为`changeit`。
`plugins.security.ssl.http.truststore_type` | TrustStore文件，JKS或PKCS12/PFX的类型。默认值为JK。
`plugins.security.ssl.http.truststore_filepath` | 通往信托店文件的路径，必须在`config` 目录，使用相对路径指定。必需的。
`plugins.security.ssl.http.truststore_alias` | 别名。选修的。默认是所有证书。
`plugins.security.ssl.http.truststore_password` | TrustStore密码。默认为`changeit`。


## 配置节点证书

OpenSearch Security需要确定群集中的节点之间的请求。它使用节点证书来保护这些请求。配置节点证书的最简单方法是列出这些证书的杰出名称（DN）`opensearch.yml`。所有DN必须包括在`opensearch.yml` 在所有节点上。请记住，安全插件支持通配符和正则表达式：

```yml
plugins.security.nodes_dn:
  - 'CN=node.other.com,OU=SSL,O=Test,L=Test,C=DE'
  - 'CN=*.example.com,OU=SSL,O=Test,L=Test,C=DE'
  - 'CN=elk-devcluster*'
  - '/CN=.*regex/'
```

如果您的节点证书在SAN部分中具有对象ID（OID）标识符，则可以省略此配置。


## 配置管理证书

管理员证书是常规客户证书，具有执行管理任务的权利较高。您需要管理证书来更改安全插件配置[`plugins/opensearch-security/tools/securityadmin.sh`]({{site.url}}{{site.baseurl}}/security/configuration/security-admin/) 或剩下的API。管理证书已配置在`opensearch.yml` 通过说明他们的DN：

```yml
plugins.security.authcz.admin_dn:
  - CN=admin,OU=SSL,O=Test,L=Test,C=DE
```

出于安全原因，您无法在此处使用通配符或正则表达式。


## （高级）openssl

安全插件支持OpenSSL，但是我们仅在使用Java 8时建议它，如果您使用Java 11，我们建议使用默认配置。

要使用OpenSSL，您必须安装OpenSSL，Apache Portable运行时和带有OpenSSL的Netty版本，将支持您在所有节点上的平台匹配。

如果启用了OpenSSL，但是由于一个或另一个原因，安装不起作用，则安全插件将返回到Java JCE作为安全引擎。

姓名| 描述
:--- | :---
`plugins.security.ssl.transport.enable_openssl_if_available` | 如果可用，请在传输层上启用OpenSSL。选修的。默认是正确的。
`plugins.security.ssl.http.enable_openssl_if_available` | 如果可用，请在其余层上启用OpenSSL。选修的。默认是正确的。

{% comment %}
1. 安装[OpenSSL 1.1.0](https://www.openssl.org/community/binaries.html) 在每个节点上。
1. 安装[Apache便携式运行时](https://apr.apache.org) 在每个节点上：

  ```
  sudo yum install apr
  ```
{% endcomment %}


## （高级）主机名验证和DNS查找

除了针对Root CA和/或中间CA（S）验证TLS证书外，安全插件还可以在传输层上应用其他检查。

和`enforce_hostname_verification` 启用，安全插件验证了通信合作伙伴的主机名与证书中的主机名匹配。主机名取自`subject` 或者`SAN` 证书的条目。例如，如果您的节点的主机名是`node-0.example.com`，然后必须将TLS证书中的主机名设置为`node-0.example.com`，也是。否则，会丢弃错误：

```
[ERROR][c.a.o.s.s.t.opensearchSecuritySSLNettyTransport] [WX6omJY] SSL Problem No name matching <hostname> found
[ERROR][c.a.o.s.s.t.opensearchSecuritySSLNettyTransport] [WX6omJY] SSL Problem Received fatal alert: certificate_unknown
```

另外，什么时候`resolve_hostname` 已启用，安全插件将（经过验证的）主机名与您的DNS解析。如果主机名无法解决，则会丢弃错误：


姓名| 描述
:--- | :---
`plugins.security.ssl.transport.enforce_hostname_verification` | 是否在运输层上验证主机名。选修的。默认是正确的。
`plugins.security.ssl.transport.resolve_hostname` | 是否要在传输层上针对DNS解析主机名。选修的。默认是正确的。仅当还启用主机名验证时才有效。


## （高级）客户身份验证

通过启用TLS客户端身份验证，REST客户端可以发送带有HTTP请求的TLS证书，以向安全插件提供身份信息。TLS客户端身份验证有三种主要用法方案：

- 使用REST管理API时提供管理证书。
- 基于客户端证书配置角色和权限。
- 为OpenSearch仪表板，LogStash或Beats等工具提供身份信息。

TLS客户端身份验证具有三种模式：

*`NONE`：安全插件不接受TLS客户端证书。如果发送，则将其丢弃。
*`OPTIONAL`：安全插件如果发送了TLS客户端证书，但不需要它们。
*`REQUIRE`：安全插件仅在发送有效的客户端TLS证书时接受REST请求。

对于REST管理API，客户端身份验证模式至少必须是可选的。

您可以使用以下设置来配置客户端身份验证模式：

姓名| 描述
:--- | :---
plugins.security.ssl.http.clientauth_mode| 使用TLS客户端身份验证模式。可以是`NONE`，`OPTIONAL` （默认）或`REQUIRE`。选修的。


## （高级）启用密码和协议

您可以限制其余层的允许的密码和TLS协议。例如，您只能允许强烈的密码，并将TLS版本限制在最近的版本中。

如果未启用此设置，则自动在浏览器和安全插件之间协商密码和TLS版本，在某些情况下，这可能会导致使用较弱的密码套件。您可以使用以下设置配置密码和协议。

姓名| 数据类型| 描述
:--- | :--- | :---
`plugins.security.ssl.http.enabled_ciphers` | 大批| 启用了其余层的TLS密码套件。仅支持Java格式。
`plugins.security.ssl.http.enabled_protocols` | 大批| 启用了其余层的TLS协议。仅支持Java格式。
`plugins.security.ssl.transport.enabled_ciphers` | 大批| 启用了运输层的TLS密码套件。仅支持Java格式。
`plugins.security.ssl.transport.enabled_protocols` | 大批| 启用了运输层的TLS协议。仅支持Java格式。

### 示例设置

```yml
plugins.security.ssl.http.enabled_ciphers:
  - "TLS_DHE_RSA_WITH_AES_256_CBC_SHA"
  - "TLS_DHE_DSS_WITH_AES_128_CBC_SHA256"
plugins.security.ssl.http.enabled_protocols:
  - "TLSv1.1"
  - "TLSv1.2"
```

因为它是不安全的，所以安全插件禁用`TLSv1` 默认情况下。如果您需要使用`TLSv1` 并接受风险，您仍然可以启用它：

```yml
plugins.security.ssl.http.enabled_protocols:
  - "TLSv1"
  - "TLSv1.1"
  - "TLSv1.2"
```


## （高级）禁用客户启动Java的重新谈判8

放`-Djdk.tls.rejectClientInitiatedRenegotiation=true` 为了禁用安全客户端的启动重新谈判，默认情况下将启用。这可以通过`OPENSEARCH_JAVA_OPTS` 在`config/jvm.options`。

## （高级）使用SSL的加密密码设置

默认的不安全SSL密码设置已弃用。为了使用这些设置的安全替代方案，用户可以使用其替代表格。具体来说，用户可以附加`_secure` SSL设置的后缀。由此产生的安全替代方案是：

* plugins.security.ssl.http.pemkey_password_secure
* plugins.security.ssl.http.keystore_password_secure
* plugins.security.ssl.http.keystore_keypassword_secure
* plugins.security.ssl.http.truststore_password_secure
* plugins.security.ssl.transport.pemkey_password_secure
* plugins.security.ssl.transport.server.pemkey_password_secure
* plugins.security.ssl.transport.client.pemkey_password_secure
* plugins.security.ssl.transport.keystore_password_secure
* plugins.security.ssl.transport.keystore_keypassword_secure
* plugins.security.ssl.transport.server.keystore_keypassword_secure
* plugins.security.ssl.transport.client.keystore_keypassword_secure
* plugins.security.ssl.transport.truststore_password_secure

这些设置允许在设置中使用加密密码。

