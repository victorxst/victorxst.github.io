---
layout: default
title: 故障排除TLS
nav_order: 15
---

# TLS故障排除

此页面包括使用安全插件配置TLS证书的故障排除步骤。


---

#### Table of contents
- TOC
{:toc}


---


## 验证YAML

`opensearch.yml` 和文件中的文件`config/opensearch-security/` 处于YAML格式。类似的衬里[YAML验证器](https://codebeautify.org/yaml-validator) 可以帮助验证您没有任何格式错误。


## 查看PEM证书的内容

您可以使用openssl显示每个PEM证书的内容：

```bash
openssl x509 -subject -nameopt RFC2253 -noout -in node1.pem
```

然后确保值与`opensearch.yml`。

有关证书的更多完整信息：

```bash
openssl x509 -in node1.pem -text -noout
```


### 检查DNS中的特殊字符和空格

安全插件使用[杰出名称的字符串表示（RFC1779）](https://www.ietf.org/rfc/rfc1779.txt) 验证节点证书时。

如果DN的一部分包含特殊字符（例如逗号），请确保您以配置逃脱：

```yml
plugins.security.nodes_dn:
  - 'CN=node-0.example.com,OU=SSL,O=My\, Test,L=Test,C=DE'
```

您可以在一个字段中有空格，但可以在字段之间使用。

#### 不良配置

```yml
plugins.security.nodes_dn:
  - 'CN=node-0.example.com, OU=SSL,O=My\, Test, L=Test, C=DE'
```

#### 良好的配置

```yml
plugins.security.nodes_dn:
  - 'CN=node-0.example.com,OU=SSL,O=My\, Test,L=Test,C=DE'
```


### 检查证书IP地址

有时，您的证书中的IP地址不是与集群通信的IP地址。如果您的节点具有多个接口或在双堆栈网络（IPv6和IPv4）上运行，则可能会发生此问题。

如果发生此问题，您可能会在节点的OpenSearch日志中看到以下内容：

```
SSL Problem Received fatal alert: certificate_unknown javax.net.ssl.SSLException: Received fatal alert: certificate_unknown
```

当新节点试图加入群集时，您可能还会在集群的主日志中看到以下消息：

```
Caused by: java.security.cert.CertificateException: No subject alternative names matching IP address 10.0.0.42 found
```

检查证书中的IP地址：

```
IPAddress: 2001:db8:0:1:1.2.3.4
```

在此示例中，节点试图通过IPv4地址加入集群`10.0.0.42`，但证书包含IPv6地址`2001:db8:0:1:1.2.3.4`。


### 验证证书链

TLS证书在证书链中组织。您可以检查`keytool` 通过检查所有者和每个证书的发行人，证书链是正确的。如果您使用了带有安全插件的演示安装脚本，则链条看起来像：

#### 节点证书

```
Owner: CN=node-0.example.com, OU=SSL, O=Test, L=Test, C=DE
Issuer: CN=Example Com Inc. Signing CA, OU=Example Com Inc. Signing CA, O=Example Com Inc., DC=example, DC=com
```

#### 签名证书

```
Owner: CN=Example Com Inc. Signing CA, OU=Example Com Inc. Signing CA, O=Example Com Inc., DC=example, DC=com
Issuer: CN=Example Com Inc. Root CA, OU=Example Com Inc. Root CA, O=Example Com Inc., DC=example, DC=com
```

#### 根证书

```
Owner: CN=Example Com Inc. Root CA, OU=Example Com Inc. Root CA, O=Example Com Inc., DC=example, DC=com
Issuer: CN=Example Com Inc. Root CA, OU=Example Com Inc. Root CA, O=Example Com Inc., DC=example, DC=com
```

从条目中，您可以看到根证书签署了中间证书，该证书签署了节点证书。根证书本身签名，因此名称"self-signed certificate." 如果您使用的是单独的密钥库和TrustStore文件，则您的根CA最有可能在TrustStore中。

通常，密钥库包含客户端或节点证书和所有中间证书，并且TrustStore包含根证书。


### 检查配置的别名

如果您在密钥库中有多个条目，并且正在使用别名来引用它们，请确保已配置的别名`opensearch.yml` 匹配密钥库中的一个。如果密钥库中只有一个条目，则不需要配置别名。


## 查看密钥店和信托店的内容

为了查看有关密钥库或信托店中存储的证书的信息，请使用`keytool` 命令类似：

```bash
keytool -list -v -keystore keystore.jks
```

`keytool` 提示获取密钥库的密码，并列出所有条目。例如，您可以使用此输出来检查SAN和EKU设置的正确性。


## 检查SAN主机名和IP地址

TLS证书的有效主机名和IP地址存储为`SAN` 条目。检查是否在`SAN` 部分是正确的，尤其是当您使用主机名验证时：

```
Certificate[1]:
Owner: CN=node-0.example.com, OU=SSL, O=Test, L=Test, C=DE
...
Extensions:
...
#5: ObjectId: 2.5.29.17 Criticality=false
SubjectAlternativeName [
  DNSName: node-0.example.com
  DNSName: localhost
  IPAddress: 127.0.0.1
  ...
]
```


## 检查OID是否有节点证书

如果您使用OID表示有效的节点证书，请检查`SAN` 您的节点证书的扩展名包含正确的`OIDName`：

```
Certificate[1]:
Owner: CN=node-0.example.com, OU=SSL, O=Test, L=Test, C=DE
...
Extensions:
...
#5: ObjectId: 2.5.29.17 Criticality=false
SubjectAlternativeName [
  ...
  OIDName: 1.2.3.4.5.5
]
```


## 检查EKU字段中的节点证书

节点证书需要两个`serverAuth` 和`clientAuth` 设置在扩展密钥用法字段中：

```
#3: ObjectId: 2.5.29.37 Criticality=false
ExtendedKeyUsages [
  serverAuth
  clientAuth
]
```


## TLS版本

安全插件默认情况下禁用TLS 1.0版；它过时，不安全和脆弱。如果您需要使用`TLSv1` 并接受风险，您可以将其启用`opensearch.yml`：

```yml
plugins.security.ssl.http.enabled_protocols:
  - "TLSv1"
  - "TLSv1.1"
  - "TLSv1.2"
```


## 支持的密码

TLS依靠服务器和客户端谈判通用密码套件。根据您的系统，可用的密码会有所不同。它们取决于您正在使用的JDK或OpenSSL版本，以及是否是`JCE Unlimited Strength Jurisdiction Policy Files` 已安装。

出于法律原因，JDK不包括像AES256这样的强密码。为了使用强密码，您需要下载并安装[Java密码扩展（JCE）无限强度管辖权政策文件](https://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)。如果您没有安装它们，则可能会在启动上看到一条错误消息：

```
[INFO ] AES-256 not supported, max key length for AES is 128 bit.
That is not an issue, it just limits possible encryption strength.
To enable AES 256 install 'Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files'
```

安全插件仍然可以使用，并恢复到较弱的密码套件。该插件还在启动期间打印出所有可用的密码套件：

```
[INFO ] sslTransportClientProvider:
JDK with ciphers [TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256, TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256, TLS_DHE_RSA_WITH_AES_128_CBC_SHA256,
TLS_DHE_DSS_WITH_AES_128_CBC_SHA256, ...]
```

