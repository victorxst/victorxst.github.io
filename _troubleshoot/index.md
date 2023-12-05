---
layout: default
title: 常见问题
nav_order: 1
has_toc: false
nav_exclude: true
redirect_from: /troubleshoot/
---

# 常见问题

此页面包含常见问题和解决方法的列表。


## OpenSearch仪表板无法启动

如果遇到错误`FATAL  Error: Request Timeout after 30000ms` 在启动过程中，尝试在更强大的机器上运行OpenSearch仪表板。我们建议四个CPU内核和8 GB RAM。


## 对OpenSearch仪表板的请求失败"Request must contain a osd-xsrf header"

如果您运行Legacy Kibana OSS脚本，请使用OpenSearch仪表板---例如，curl命令从文件导入对象---他们可能会因以下错误而失败：

```json
{"status": 400, "body": "Request must contain a osd-xsrf header."}
```

在这种情况下，您的脚本可能包括`"kbn-xsrf: true"` 标题。切换到`osd-xsrf: true` 标头：

```
curl -XPOST -u 'admin:admin' 'https://DASHBOARDS_ENDPOINT/api/saved_objects/_import' -H 'osd-xsrf:true' --form file=@export.ndjson
```


## 多-OpenSearch仪表板中的租赁问题

如果您在OpenSearch仪表板中测试了多个用户，并遇到租户意外的更改，请在隐身窗口中使用Google Chrome或在私人窗口中使用Firefox。


## 过期证书

如果您的证书已过期，则可能会收到以下错误或类似的错误：

```
ERROR org.opensearch.security.ssl.transport.SecuritySSLNettyTransport - Exception during establishing a SSL connection: javax.net.ssl.SSLHandshakeException: PKIX path validation failed: java.security.cert.CertPathValidatorException: validity check failed
Caused by: java.security.cert.CertificateExpiredException: NotAfter: Thu Sep 16 11:27:55 PDT 2021
```

要检查证书的到期日期，请运行此命令：

```bash
openssl x509 -enddate -noout -in <certificate>
```


## 休息时加密

每个OpenSearch节点的操作系统都在静止下处理数据的加密。要在大多数Linux分布中启用静止的加密，请使用`cryptsetup` 命令：

```bash
cryptsetup luksFormat --key-file <key> <partition>
```

有关命令的完整文档，请参阅[CryptSetup（8） -  Linux手册页面](https://man7.org/linux/man-pages/man8/cryptsetup.8.html)。

{％ 评论 ％}
## 节拍

如果尝试将Beats连接到OpenSearch时遇到兼容性问题，请确保您使用的是Apache 2.0 Beats的分布，而不是使用专有许可证的默认分发。

尝试使用安全插件的Beats来尝试此最小输出配置：

```yml
output.elasticsearch:
  hosts: ["localhost:9200"]
  protocol: https
  username: "admin"
  password: "admin"
  ssl.certificate_authorities:
    - /full/path/to/root-ca.pem
  ssl.certificate: "/full/path/to/client.pem"
  ssl.key: "/full/path/to/client-key.pem"
```

即使使用OSS版本，Beats也可能在OpenSearch服务器上检查专有插件，并在启动过程中丢弃错误。要禁用支票，请尝试添加以下设置：

```yml
setup.ilm.enabled: false
setup.ilm.check_exists: false
```


## logstash

如果您在将LogStash连接到OpenSearch时遇到困难，请尝试使用此最小输出配置，该配置可与安全插件一起使用：

```conf
output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "logstash-index-test"
    user => "admin"
    password => "admin"
    ssl => true
    cacert => "/full/path/to/root-ca.pem"
    ilm_enabled => false
  }
}
```
{％endcomment％}

## FLS，DLS或现场掩码处于活动状态时无法通过脚本更新

安全插件通过脚本操作阻止更新（`POST <index>/_update/<id>`）字段-级别安全性，文档-级别安全性或现场掩码处于活动状态。您仍然可以使用标准索引操作更新文档（`PUT <index>/_doc/<id>`）。


## 日志中的非法反射访问操作

这是性能分析仪的已知问题，不应影响功能。

