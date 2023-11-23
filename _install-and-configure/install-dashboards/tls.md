---
layout: default
title: 配置 TLS
parent: 安装 OpenSearch 控制面板
nav_order: 40
redirect_from: 
  - /dashboards/install/tls/
---

# 为 OpenSearch 控制面板配置 TLS

默认情况下，为了便于测试和入门，OpenSearch 控制面板通过 HTTP 运行。要为 HTTPS 启用 TLS，请更新中的以下设置 `opensearch_dashboards.yml`。

设置 | 描述
:--- | :---
opensearch.ssl.verificationMode | 此设置用于 OpenSearch 和 OpenSearch 控制面板之间的通信。有效值为 `full`、 `certificate` 或 `none`。 `full` 我们建议你启用 TLS，这将启用主机名验证。 `certificate` 只检查证书，而不是主机名，不 `none` 执行任何检查（适用于 HTTP）。缺省值为 `full`。
opensearch.ssl.certificate 颁发机构 | 如果 `opensearch.ssl.verificationMode` 为 `full` 或 `certificate`，请指定一个或多个 CA 证书的完整路径，这些证书构成 OpenSearch 集群的可信链。例如，如果你使用中间 CA 颁发管理员、客户端和节点证书，则可能需要包含根 CA 和中间 CA _和_。
server.ssl.enabled 服务器.ssl.enabled | 此设置用于 OpenSearch 控制面板和 Web 浏览器之间的通信。对于 HTTPS，设置为 true，对于 HTTP，设置为 false。
server.ssl.证书 | 如果 `server.ssl.enabled` 为 true，请指定 OpenSearch 集群的有效客户端证书的完整路径。你可以从[生成你自己的]({{site.url}}{{site.baseurl}}/security/configuration/generate-certificates/)证书颁发机构获取一个。
server.ssl.key 密钥 | 如果 `server.ssl.enabled` 为 true，请指定完整路径（例如， `/usr/share/opensearch-dashboards-1.0.0/config/my-client-cert-key.pem` 客户端证书的密钥。你可以从[生成你自己的]({{site.url}}{{site.baseurl}}/security/configuration/generate-certificates/)证书颁发机构获取一个。
server.ssl.certificate 颁发机构 | 此设置添加 SSL 证书颁发机构，该证书颁发机构以列表格式为 Dashboard 的服务器颁发 SSL 证书。
opensearch.ssl.certificate 颁发机构 | 此设置将添加 OpenSearch 的 SSL 证书颁发机构。
opensearch_security.cookie.secure | 如果你为 OpenSearch 控制面板启用 TLS，请将此设置更改为 `true`。对于 HTTP，请将其设置为 `false`。

此 `opensearch_dashboards.yml` 配置显示 OpenSearch 和 OpenSearch 控制面板在具有演示配置的同一台计算机上运行：

```yml
opensearch.hosts: ["https://localhost:9200"]
opensearch.ssl.verificationMode: full
opensearch.username: "kibanaserver"
opensearch.password: "kibanaserver"
opensearch.requestHeadersAllowlist: [ authorization,securitytenant ]
server.ssl.enabled: true
server.ssl.certificate: /usr/share/opensearch-dashboards/config/client-cert.pem
server.ssl.key: /usr/share/opensearch-dashboards/config/client-cert-key.pem
server.ssl.certificateAuthorities: [ "/usr/share/opensearch-dashboards/config/root-ca.pem", "/usr/share/opensearch-dashboards/config/intermediate-ca.pem" ]
opensearch.ssl.certificateAuthorities: [ "/usr/share/opensearch-dashboards/config/root-ca.pem", "/usr/share/opensearch-dashboards/config/intermediate-ca.pem" ]
opensearch_security.multitenancy.enabled: true
opensearch_security.multitenancy.tenants.preferred: ["Private", "Global"]
opensearch_security.readonly_mode.roles: ["kibana_read_only"]
opensearch_security.cookie.secure: true
```

如果使用 Docker 安装，则可以将自定义项 `opensearch_dashboards.yml` 传递到容器。要了解更多信息，请参阅[Docker 安装页面]({{site.url}}{{site.baseurl}}/opensearch/install/docker/).

启用这些设置并启动 OpenSearch 控制面板后，你可以在上 `https://localhost:5601` 连接到它。如果你的证书是自签名的，则可能需要确认浏览器警告。为了避免这种警告（或完全不兼容浏览器），最佳做法是使用来自受信任证书颁发机构的证书。
