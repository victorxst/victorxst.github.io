---
layout: default
title: 禁用安全性
parent: 配置
nav_order: 40
redirect_from: 
 - /security-plugin/configuration/disable/
---

# 禁用安全性

您可能需要暂时禁用安全插件，以使测试或内部用法更加简单。要禁用插件，请在`opensearch.yml`：

```yml
plugins.security.disabled: true
```

一个更永久的选择是完全删除安全插件：

1. 删除`plugins/opensearch-security` 所有节点上的文件夹。
1. 删除所有`plugins.security.*` 配置条目`opensearch.yml`。

要在Docker图像上执行这些步骤，请参见[使用插件]({{site.url}}{{site.baseurl}}/opensearch/install/docker#working-with-plugins)。

禁用或删除该插件会曝光安全插件的配置索引。如果该索引包含敏感信息，请确保通过其他一些方式保护它。如果您不再需要索引，请删除。
{: .warning }


## 删除OpenSearch仪表板插件

安全插件实际上是两个插件：一个用于OpenSearch，一个用于OpenSearch仪表板。您可以独立使用OpenSearch插件，但是OpenSearch仪表板插件取决于固定的OpenSearch cluster。

如果禁用安全插件`opensearch.yml` （或完全删除插件）并且仍然需要使用OpenSearch仪表板，必须删除相应的OpenSearch仪表板插件。有关更多信息，请参阅[OpenSearch仪表板删除插件]({{site.url}}{{site.baseurl}}/install-and-configure/install-dashboards/plugins/#remove-plugins)。


### Docker

1. 创建一个新的`Dockerfile`：

   ```
   来自OpenSearchProject/OpenSearch-仪表板：{{site.opensearch_dashboards_version}}
   运行/usr/share/opensearch-仪表板/bin/opensearch-仪表板-插件删除SecurityDashboard
   复制--chown = OpenSearch-仪表板：OpenSearch-仪表板opensearch_dashboards.yml/usr/share/opensearch-仪表板/config/
   ```

   In this case, `OpenSearch_Dashboards.yml` is a "vanilla" version of the file with no entries for the Security plugin. It might look like this:

   ```YML
   ---
   server.name：opensearch-仪表板
   server.host："0.0.0.0"
   OpenSearch.Hosts：http：// localhost：9200
   ```


1. To build the new Docker image, run the following command:

   ```bash
   Docker Build--tag = OpenSearch-仪表板-不-安全 。
   ```

1. In `Docker-compose.yml`, change `OpenSearchProject/OpenSearch-仪表板：{{site.opensearch_dashboards_version}}` to `OpenSearch-仪表板-不-安全`。
1. 改变`OPENSEARCH_HOSTS` 或者`opensearch.hosts` 到`http://` 而不是`https://`。
1. 进入`docker-compose up`。

