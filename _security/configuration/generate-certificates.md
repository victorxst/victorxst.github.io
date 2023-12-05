---
layout: default
title: 生成自签名证书
parent: 配置
nav_order: 20
redirect_from:
  - /security-plugin/configuration/generate-certificates/
---

# 生成自签名证书

如果您无法访问组织的证书授权（CA），而是想将OpenSearch用于非-演示目的，您可以产生自己的自我-使用的签名证书[Openssl](https://www.openssl.org/){：target ='\ _ blank'}。

您可能可以在操作系统的软件包管理器中找到OpenSSL。

在CentOS上，使用百胜：

```bash
sudo yum install openssl
```

在MacOS上，使用[自制](https://brew.sh/){：target ='\ _ blank'}：

```bash
brew install openssl
```


## 生成私钥

此过程的第一步是使用`openssl genrsa` 命令。顾名思义，您应该将此文件保密。

私钥必须足够长才能安全，因此请指定`2048`：

```bash
openssl genrsa -out root-ca-key.pem 2048
```

您可以选择添加`-aes256` 使用AES加密密钥的选项-256标准。此选项需要密码。


## 生成根证书

接下来，使用私钥生成自我-Root CA的签名证书：

```bash
openssl req -new -x509 -sha256 -key root-ca-key.pem -out root-ca.pem -days 730
```

默认值`-days` 30值仅对测试目的有用。此示例命令指定证书到期日期的730（两年），但使用任何值对您的组织有意义的。

- 这`-x509` 选项指定您想要自我-签名证书而不是证书请求。
- 这`-sha256` 选项将哈希算法设置为SHA-256.莎-256是后期版本的openssl的默认值，但早期版本可能会使用SHA-1。

请按照提示为您的组织指定详细信息。这些细节共同构成了您CA的杰出名称（DN）。


## 生成管理证书

要生成管理证书，请首先创建一个新密钥：

```bash
openssl genrsa -out admin-key-temp.pem 2048
```

然后将该键转换为PKCS#8格式用于使用PKC在Java中使用#12-兼容算法（3DE）：

```bash
openssl pkcs8 -inform PEM -outform PEM -in admin-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out admin-key.pem
```

接下来，创建一个证书签名请求（CSR）。该文件充当签名证书的CA的申请：

```bash
openssl req -new -key admin-key.pem -out admin.csr
```

请按照提示填写详细信息。您无需指定挑战密码。如在[OpenSSL食谱](https://www.feistyduck.com/books/openssl-cookbook/){：target ='\ _ blank'}，"Having a challenge password does not increase the security of the CSR in any way."

如果您生成TLS证书并通过设置启用了主机名验证`plugins.security.ssl.transport.enforce_hostname_verification` 到`true` （默认值），请确保为每个证书签名请求（CSR）指定一个与相应的DNS记录匹配的通用名称（CSR）。

如果要在所有节点上使用相同的节点证书（不建议），请将主机名验证设置为`false`。有关更多信息，请参阅[配置TLS证书]({{site.url}}{{site.baseurl}}/security/configuration/tls/#advanced-hostname-verification-and-dns-lookup)。

现在已经创建了私钥和签名请求，生成证书：

```bash
openssl x509 -req -in admin.csr -CA root-ca.pem -CAkey root-ca-key.pem -CAcreateserial -sha256 -out admin.pem -days 730
```

就像根证书一样，使用`-days` 指定到期日期超过30天的到期日期的选项。


## （可选）生成节点和客户端证书

类似于步骤[生成管理证书](#generate-an-admin-certificate)，您将生成每个节点的新文件名和最多客户证书的密钥和CSR。例如，您可能会生成一个用于OpenSearch仪表板的客户端证书，而为Python客户端生成了另一个客户端证书。每个证书应使用自己的私钥，并应从唯一的企业社会责任中生成，其匹配的SAN扩展特定于预期的主机。管理证书不需要SAN扩展名，因为该证书与特定主机没有绑定。

要生成节点或客户端证书，请首先创建一个新密钥：

```bash
openssl genrsa -out node1-key-temp.pem 2048
```

然后将该键转换为PKCS#8格式用于使用PKC在Java中使用#12-兼容算法（3DE）：

```bash
openssl pkcs8 -inform PEM -outform PEM -in node1-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out node1-key.pem
```

接下来，创建CSR：

```bash
openssl req -new -key node1-key.pem -out node1.csr
```

对于所有主持人和客户证书，您应该指定主题替代名称（SAN）以确保符合[RFC 2818（TLS上的HTTP）](https://datatracker.ietf.org/doc/html/rfc2818)。SAN应与相应的CN匹配，以便两者都指相同的DNS记录。
{： 。笔记 }

在生成签名证书之前，创建一个SAN扩展文件，该文件描述了主机的DNS记录：

```bash
echo 'subjectAltName=DNS:node1.dns.a-record' > node1.ext
```

生成证书：

```bash
openssl x509 -req -in node1.csr -CA root-ca.pem -CAkey root-ca-key.pem -CAcreateserial -sha256 -out node1.pem -days 730 -extfile node1.ext
```


## 示例脚本

如果您已经知道证书详细信息并且不想交互方式指定它们，请使用`-subj` 您的选项`root-ca.pem` 和CSR命令。该脚本创建一个根证书，管理证书，两个节点证书和客户证书，均为两年（730天）：

```bash
#!/bin/sh
# Root CA
openssl genrsa -out root-ca-key.pem 2048
openssl req -new -x509 -sha256 -key root-ca-key.pem -subj "/C=CA/ST=ONTARIO/L=TORONTO/O=ORG/OU=UNIT/CN=root.dns.a-record" -out root-ca.pem -days 730
# Admin cert
openssl genrsa -out admin-key-temp.pem 2048
openssl pkcs8 -inform PEM -outform PEM -in admin-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out admin-key.pem
openssl req -new -key admin-key.pem -subj "/C=CA/ST=ONTARIO/L=TORONTO/O=ORG/OU=UNIT/CN=A" -out admin.csr
openssl x509 -req -in admin.csr -CA root-ca.pem -CAkey root-ca-key.pem -CAcreateserial -sha256 -out admin.pem -days 730
# Node cert 1
openssl genrsa -out node1-key-temp.pem 2048
openssl pkcs8 -inform PEM -outform PEM -in node1-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out node1-key.pem
openssl req -new -key node1-key.pem -subj "/C=CA/ST=ONTARIO/L=TORONTO/O=ORG/OU=UNIT/CN=node1.dns.a-record" -out node1.csr
echo 'subjectAltName=DNS:node1.dns.a-record' > node1.ext
openssl x509 -req -in node1.csr -CA root-ca.pem -CAkey root-ca-key.pem -CAcreateserial -sha256 -out node1.pem -days 730 -extfile node1.ext
# Node cert 2
openssl genrsa -out node2-key-temp.pem 2048
openssl pkcs8 -inform PEM -outform PEM -in node2-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out node2-key.pem
openssl req -new -key node2-key.pem -subj "/C=CA/ST=ONTARIO/L=TORONTO/O=ORG/OU=UNIT/CN=node2.dns.a-record" -out node2.csr
echo 'subjectAltName=DNS:node2.dns.a-record' > node2.ext
openssl x509 -req -in node2.csr -CA root-ca.pem -CAkey root-ca-key.pem -CAcreateserial -sha256 -out node2.pem -days 730 -extfile node2.ext
# Client cert
openssl genrsa -out client-key-temp.pem 2048
openssl pkcs8 -inform PEM -outform PEM -in client-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out client-key.pem
openssl req -new -key client-key.pem -subj "/C=CA/ST=ONTARIO/L=TORONTO/O=ORG/OU=UNIT/CN=client.dns.a-record" -out client.csr
echo 'subjectAltName=DNS:client.dns.a-record' > client.ext
openssl x509 -req -in client.csr -CA root-ca.pem -CAkey root-ca-key.pem -CAcreateserial -sha256 -out client.pem -days 730 -extfile client.ext
# Cleanup
rm admin-key-temp.pem
rm admin.csr
rm node1-key-temp.pem
rm node1.csr
rm node1.ext
rm node2-key-temp.pem
rm node2.csr
rm node2.ext
rm client-key-temp.pem
rm client.csr
rm client.ext
```


## 在opensearch.yml中添加杰出的名称

您必须为所有管理员指定所有管理员和节点证书`opensearch.yml` 在所有节点上。使用上面示例脚本的证书，一部分`opensearch.yml` 看起来像这样：

```yml
plugins.security.authcz.admin_dn:
  - 'CN=A,OU=UNIT,O=ORG,L=TORONTO,ST=ONTARIO,C=CA'
plugins.security.nodes_dn:
  - 'CN=node1.dns.a-record,OU=UNIT,O=ORG,L=TORONTO,ST=ONTARIO,C=CA'
  - 'CN=node2.dns.a-record,OU=UNIT,O=ORG,L=TORONTO,ST=ONTARIO,C=CA'
```

但是，如果您看着`subject` 证书创建后，您可能会看到不同的格式：

```
subject=/C=CA/ST=ONTARIO/L=TORONTO/O=ORG/OU=UNIT/CN=node1.dns.a-record
```

如果将此字符串与上面的字符串进行比较，则可以看到需要倒入元素的顺序并使用逗号而不是斜线。输入此命令以获取正确的字符串：

```bash
openssl x509 -subject -nameopt RFC2253 -noout -in node.pem
```

然后将输出复制并粘贴到`opensearch.yml`。


## 将证书文件添加到OpenSearch.yml

此过程生成了许多文件，但是这些文件是您需要添加到每个节点的文件：

- `root-ca.pem`
- （选修的）`admin.pem`
- （选修的）`admin-key.pem`
- （选修的）`node1.pem`
- （选修的）`node1-key.pem`

对于大多数用户，`admin.pem` 和`admin-key.pem` 文件仅需要添加到您计划运行的节点`securityadmin` 脚本或重新加载证书。有关如何使用的信息`securityadmin` 脚本，请参阅[将更改应用于配置文件]({{site.url}}{{site.baseurl}}/security/configuration/security-admin/)。如果您打算运行`securityadmin` 直接来自节点的脚本，该节点需要具有`admin.pem` 和`admin-key.pem` 在上面。

在一个节点上，安全配置部分`opensearch.yml` 看起来像这样：

```yml
plugins.security.ssl.transport.pemcert_filepath: node1.pem
plugins.security.ssl.transport.pemkey_filepath: node1-key.pem
plugins.security.ssl.transport.pemtrustedcas_filepath: root-ca.pem
plugins.security.ssl.transport.enforce_hostname_verification: false
plugins.security.ssl.http.enabled: true
plugins.security.ssl.http.pemcert_filepath: node1.pem
plugins.security.ssl.http.pemkey_filepath: node1-key.pem
plugins.security.ssl.http.pemtrustedcas_filepath: root-ca.pem
plugins.security.authcz.admin_dn:
  - 'CN=A,OU=UNIT,O=ORG,L=TORONTO,ST=ONTARIO,C=CA'
plugins.security.nodes_dn:
  - 'CN=node1.dns.a-record,OU=UNIT,O=ORG,L=TORONTO,ST=ONTARIO,C=CA'
  - 'CN=node2.dns.a-record,OU=UNIT,O=ORG,L=TORONTO,ST=ONTARIO,C=CA'
```

有关在您自己的设置中添加和使用这些证书的更多信息，请参见[配置基本的安全设置]({{site.url}}{{site.baseurl}}/install-and-configure/install-opensearch/docker/#configuring-basic-security-settings) 对于Docker，[配置TLS证书]({{site.url}}{{site.baseurl}}/security/configuration/tls/)， 和[客户证书身份验证]({{site.url}}{{site.baseurl}}/security/configuration/client-auth/)。

## OpenSearch仪表板

有关使用Root CA和客户端证书启用TLS的信息，请参见[为OpenSearch仪表板配置TLS]({{site.url}}{{site.baseurl}}/install-and-configure/install-dashboards/tls/)。

