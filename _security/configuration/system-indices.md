---
layout: default
title: 系统索引
parent: 配置
nav_order: 4
redirect_from:
 - /security-plugin/configuration/system-indices/
---

# 系统索引

默认情况下，OpenSearch具有受保护的系统索引，`.opendistro_security`，用于存储安全配置YAML文件。您可以使用此索引使用[SecurityAdmin.sh]({{site.url}}{{site.baseurl}}/security/configuration/security-admin/)。即使使用所有索引的用户帐户，您也无法直接访问此系统索引中的数据。

相反，您首先需要使用[管理证书]({{site.url}}{{site.baseurl}}/security/configuration/tls/#configuring-admin-certificates) 获得访问：

```bash
curl -k --cert ./kirk.pem --key ./kirk-key.pem -XGET 'https://localhost:9200/.opendistro_security/_search'
```

安装安全性后，演示配置会自动创建`.opendistro_security` 系统索引。它还为与安全插件集成的各种OpenSearch插件添加了其他几个索引：

```yml
plugins.security.system_indices.enabled: true
plugins.security.system_indices.indices: [".opendistro-alerting-config", ".opendistro-alerting-alert*", ".opendistro-anomaly-results*", ".opendistro-anomaly-detector*", ".opendistro-anomaly-checkpoints", ".opendistro-anomaly-detection-state", ".opendistro-reports-*", ".opendistro-notifications-*", ".opendistro-notebooks", ".opendistro-asynchronous-search-response*"]
```

您可以在`opensearch.yml`。删除系统索引的另一种方法是将其从`plugins.security.system_indices.indices` 每个节点上的列表，然后重新启动opensearch。


