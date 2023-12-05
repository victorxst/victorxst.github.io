---
layout: default
title: 审核日志存储类型
parent: 审核日志
nav_order: 135
redirect_from:
  - /security/audit-logs/storage-types/
  - /security-plugin/audit-logs/storage-types/
---

# 审核日志存储类型

审核日志可以占用很多空间，因此安全插件为存储位置提供了几种选择。

环境| 描述
:--- | :---
调试| 输出到Stdout。可用于测试和调试。
Internal_opensearch| 写入当前OpenSearch集群上的审核索引。
external_opensearch| 将远程搜索集群上的审核索引写入。
Webhook| 将事件发送到任意的HTTP端点。
log4j| 将事件写入log4j logger。您可以使用任何log4j[appender](https://logging.apache.org/log4j/2.x/manual/appenders.html)，例如SNMP，JDBC，Cassandra和Kafka。

您将输出位置配置为`opensearch.yml`：

```
plugins.security.audit.type: <debug|internal_opensearch|external_opensearch|webhook|log4j>
```

`external_opensearch`，`webhook`， 和`log4j` 所有都有其他配置选项。详细信息如下。


## 外部opensearch

这`external_opensearch` 存储类型需要一个或多个带有主机/IP地址和端口的OpenSearch端点。（可选）提供索引名称和文档类型。

```yml
plugins.security.audit.type: external_opensearch
plugins.security.audit.config.http_endpoints: [<endpoints>]
plugins.security.audit.config.index: <indexname>
plugins.security.audit.config.type: _doc
```

与任何其他索引请求一样，安全插件使用OpenSearch REST API发送事件。为了`plugins.security.audit.config.http_endpoints`，使用逗号-主机/IP地址和其余端口的分开列表（默认为9200）。

```
plugins.security.audit.config.http_endpoints: [192.168.178.1:9200,192.168.178.2:9200]
```

如果您使用`external_opensearch` 远程集群还使用安全插件，您必须提供一些其他参数以进行身份验证。这些参数取决于您为远程群集配置的哪种身份验证类型。


### TLS设置

姓名| 数据类型| 描述
:--- | :--- | :---
`plugins.security.audit.config.enable_ssl` | 布尔| 如果在接收群集上启用了SSL/TLS，请设置为true。默认值为false。
`plugins.security.audit.config.verify_hostnames` |  布尔| 是否验证接收群集的SSL/TLS证书的主机名。默认是正确的。
`plugins.security.audit.config.pemtrustedcas_filepath` | 细绳| 相对于外部openSearch集群的可信根证书`config` 目录。
`plugins.security.audit.config.pemtrustedcas_content` | 细绳| 而不是指定路径（`plugins.security.audit.config.pemtrustedcas_filepath`），您可以配置base64-直接编码证书内容。
`plugins.security.audit.config.enable_ssl_client_auth` | 布尔| 是否启用SSL/TLS客户端身份验证。如果将其设置为True，则审核日志模块将与请求一起发送节点的证书。接收集群可以使用此证书验证呼叫者的身份。
`plugins.security.audit.config.pemcert_filepath` | 细绳| TLS证书发送到外部OpenSearch集群的路径，相对于`config` 目录。
`plugins.security.audit.config.pemcert_content` | 细绳| 而不是指定路径（`plugins.security.audit.config.pemcert_filepath`），您可以配置base64-直接编码证书内容。
`plugins.security.audit.config.pemkey_filepath` | 细绳| 相对于TLS证书的私钥的路径，以发送到外部OpenSearch集群`config` 目录。
`plugins.security.audit.config.pemkey_content` | 细绳| 而不是指定路径（`plugins.security.audit.config.pemkey_filepath`），您可以配置base64-直接编码证书内容。
`plugins.security.audit.config.pemkey_password` | 细绳| 私钥的密码。


### 基本验证设置

如果在接收群集上启用了HTTP基本身份验证，请使用这些设置指定用户名和密码：

```yml
plugins.security.audit.config.username: <username>
plugins.security.audit.config.password: <password>
```


## Webhook

使用以下键配置`webhook` 存储类型。

姓名| 数据类型| 描述
:--- | :--- | :---
`plugins.security.audit.config.webhook.url` | 细绳| HTTP或HTTPS URL将日志发送到。
`plugins.security.audit.config.webhook.ssl.verify` | 布尔| 如果为true，则将验证端点（如果有）提供的TLS证书。如果设置为false，则不会执行验证。如果您使用自我，可以禁用此检查-签名证书。
`plugins.security.audit.config.webhook.ssl.pemtrustedcas_filepath` | 细绳| 验证了Webhook的TLS证书的信任证书的路径。
`plugins.security.audit.config.webhook.ssl.pemtrustedcas_content` | 细绳| 与...一样`plugins.security.audit.config.webhook.ssl.pemtrustedcas_content`，但是您可以直接配置基本64编码的证书内容。
`plugins.security.audit.config.webhook.format` | 细绳| 审核日志消息的格式记录，可以是一种`URL_PARAMETER_GET`，`URL_PARAMETER_POST`，`TEXT`，`JSON`，`SLACK`。看[格式](#formats)。


### 格式

格式| 描述
:--- | :---
`URL_PARAMETER_GET` | 使用http get将日志发送到Webhook URL。所有记录的信息都作为请求参数附加到URL。
`URL_PARAMETER_POST` | 使用HTTP帖子将日志发送到Webhook URL。所有记录的信息都作为请求参数附加到URL。
`TEXT` | 使用HTTP帖子将日志发送到Webhook URL。请求主体包含纯文本格式的审核日志消息。
`JSON` | 使用HTTP帖子将日志发送到Webhook URL。请求主体包含JSON格式的审核日志消息。
`SLACK` | 使用HTTP帖子将日志发送到Webhook URL。请求主体包含适用于Slack消费的JSON格式的审核日志消息。默认实现返回`"text": "<AuditMessage#toText>"`。


## log4j

这`log4j` 存储类型允许您指定记录器和日志级别的名称。

```yml
plugins.security.audit.config.log4j.logger_name: audit
plugins.security.audit.config.log4j.level: INFO
```

默认情况下，安全插件使用Logger名称`audit` 并记录事件`INFO` 等级。审核事件以JSON格式存储。

