---
layout: default
title: 客户证书身份验证
parent: 身份验证后端
nav_order: 70
redirect_from:
  - /security/configuration/client-auth/
  - /security-plugin/configuration/client-auth/
---

# 客户证书身份验证

从证书机构（CA）或通过[使用openssl生成自己的证书]({{site.url}}{{site.baseurl}}/security/configuration/generate-certificates)，您可以开始配置OpenSearch使用客户端证书对用户进行身份验证。

客户端证书身份验证提供的安全优势不仅仅是使用基本身份验证（用户名和密码）。由于客户端证书身份验证需要客户证书及其私钥，这通常是用户拥有的，因此它不太容易受到蛮力攻击的攻击，其中恶意个人试图猜测用户的密码。

客户证书身份验证的另一个好处是，您可以将其与基本身份验证一起使用，提供两层安全性。

## 启用客户证书身份验证

要启用客户端证书身份验证，您必须首先设置`clientauth_mode` 在`opensearch.yml` 要点`OPTIONAL` 或者`REQUIRE`：

```yml
plugins.security.ssl.http.clientauth_mode: OPTIONAL
```

接下来，在`client_auth_domain` 部分`config.yml`。

```yml
clientcert_auth_domain:
  description: "Authenticate via SSL client certificates"
  http_enabled: true
  transport_enabled: true
  order: 1
  http_authenticator:
    type: clientcert
    config:
      username_attribute: cn #optional, if omitted DN becomes username
    challenge: false
  authentication_backend:
    type: noop
```

## 将角色分配给您的通用名称

现在，您可以将证书的通用名称（CN）分配给角色。在此步骤中，您必须知道证书的CN和要分配的角色。要获取OpenSearch中所有预定义角色的列表，请参阅我们的[预定角色清单]({{site.url}}{{site.baseurl}}/security/access-control/users-roles/#predefined-roles)。如果您想首先创建角色，请参考[如何创造角色]({{site.url}}{{site.baseurl}}/security/access-control/users-roles/#create-users)，然后将您的证书的CN映射到该角色。

在确定要映射证书的CN的角色之后，您可以使用[OpenSearch仪表板]({{site.url}}{{site.baseurl}}/security/access-control/users-roles/#map-users-to-roles)，[`roles_mapping.yml`]({{site.url}}{{site.baseurl}}/security/configuration/yaml/#roles_mappingyml)， 或者[REST API]({{site.url}}{{site.baseurl}}/security/access-control/api/#create-role-mapping) 将证书的CN映射到角色。以下示例使用`REST API` 映射通用名称`CLIENT1` 扮演角色`readall`。

**示例请求**

```json
PUT _plugins/_security/api/rolesmapping/readall
{
  "backend_roles" : ["sample_role" ],
  "hosts" : [ "example.host.com" ],
  "users" : [ "CLIENT1" ]
}
```

**示例响应**

```json
{
  "status": "OK",
  "message": "'readall' updated."
}
```

将角色映射到客户端证书的CN之后，您可以使用这些凭据连接到群集。

下面的代码示例使用Python`requests` 库连接到本地OpenSearch集群，并将GET请求发送到`movies` 指数。

```python
import requests
import json
base_url = 'https://localhost:9200/'
headers = {
  'Content-Type': 'application/json'
}
cert_file_path = "/full/path/to/client-cert.pem"
key_file_path = "/full/path/to/client-cert-key.pem"
root_ca_path = "/full/path/to/root-ca.pem"

# Send the request.
path = 'movies/_doc/3'
url = base_url + path
response = requests.get(url, cert = (cert_file_path, key_file_path), verify=root_ca_path)
print(response.text)
```

{% comment %}
## 配置节拍

您还可以配置Beats，以便使用OpenSearch使用客户端证书进行身份验证。之后，它可以开始将输出发送到OpenSearch。

此输出配置指定客户端证书身份验证所需的设置：

```yml
output.opensearch:
  enabled: true
  # Array of hosts to connect to.
  hosts: ["localhost:9200"]
  # Protocol - either `http` (default) or `https`.
  protocol: "https"
  ssl.certificate_authorities: ["/full/path/to/CA.pem"]
  ssl.verification_mode: certificate
  ssl.certificate: "/full/path/to/client-cert.pem"
  ssl.key: "/full/path/to/to/client-cert-key.pem"
```
{% endcomment %}

## 与Docker一起使用证书

虽然我们建议使用[tarball]({{site.url}}{{site.baseurl}}/install-and-configure/install-opensearch/tar/) 安装ODFE来测试客户端证书身份验证配置，还可以使用其他任何安装类型。有关使用Docker安全的说明，请参阅[配置基本的安全设置]({{site.url}}{{site.baseurl}}/install-and-configure/install-opensearch/docker/#configuring-basic-security-settings)。

