---
layout: default
title: 配置 OpenSearch
nav_order: 10
has_children: true
redirect_from:
  - /install-and-configure/configuring-opensearch/
  - /opensearch/configuration/
---

# 配置 OpenSearch

有两种类型的 OpenSearch 设置：[dynamic](#dynamic-settings)和[static](#static-settings).

## 动态设置

动态索引设置是可以随时更新的设置。你可以通过集群设置 API 配置动态 OpenSearch 设置。有关详细信息，请参见[使用 API 更新群集设置](#updating-cluster-settings-using-the-api)。

尽可能使用群集设置 API; `opensearch.yml` 是每个节点的本地设置，而 API 将设置应用于集群中的所有节点。{:.tip}

## 静态设置

某些操作是静态的，需要你修改 `opensearch.yml` [配置文件](#configuration-file)并重新启动集群。通常，这些设置与网络、群集形成和本地文件系统有关。要了解更多信息，请参阅[集群形成]({{site.url}}{{site.baseurl}}/opensearch/cluster/)。

## 将设置指定为环境变量

你可以在启动 OpenSearch 时将 `-E` 环境变量指定为参数：

```bash
./opensearch -Ecluster.name=opensearch-cluster -Enode.name=opensearch-node1 -Ehttp.host=0.0.0.0 -Ediscovery.type=single-node
```
{% include copy.html %}

## 使用 API 更新集群设置

更改设置的第一步是通过发送以下请求来查看当前设置：

```json
GET _cluster/settings?include_defaults=true
```
{% include copy-curl.html %}

有关非默认设置的更简明摘要，请发送以下请求：

```json
GET _cluster/settings
```
{% include copy-curl.html %}

群集设置 API 中存在三类设置：持久性、暂时性和默认值。持久性设置在集群重新启动后仍然存在。重新启动后，OpenSearch 会清除暂时性设置。

如果你在多个位置指定相同的设置，OpenSearch 将使用以下优先级：

1. 瞬态设置
2. 持久性设置
3. 设置 `opensearch.yml`
4. 默认设置

若要更改设置，请使用[群集设置 API]({{site.url}}{{site.baseurl}}/api-reference/cluster-api/cluster-settings/)并将新值指定为持久值或瞬态值。此示例显示平面设置窗体：

```json
PUT _cluster/settings
{
  "persistent" : {
    "action.auto_create_index" : false
  }
}
```
{% include copy-curl.html %}

你还可以使用扩展的表单，该表单允许你从 GET 响应中复制和粘贴并更改现有值：

```json
PUT _cluster/settings
{
  "persistent": {
    "action": {
      "auto_create_index": false
    }
  }
}
```
{% include copy-curl.html %}

---

## 配置文件

你可以在每个节点上的（Docker）或 `/etc/opensearch/opensearch.yml`（大多数 Linux 发行版）中找到。 `opensearch.yml` `/usr/share/opensearch/config/opensearch.yml`

你可以编辑 `OPENSEARCH_PATH_CONF=/etc/opensearch` 以更改配置目录位置。这个变量来源于 `/etc/default/opensearch`（Debian 软件包）和 `/etc/sysconfig/opensearch`（RPM 软件包）。

如果设置自定义 `OPENSEARCH_PATH_CONF` 变量，请注意不会加载其他默认环境变量。

不会将设置标记为持久性或暂时性，并且设置 `opensearch.yml` 使用平面形式：

```yml
cluster.name: my-application
action.auto_create_index: true
compatibility.override_main_response_version: true
```

演示配置包括在将 OpenSearch 用于生产工作负载之前应修改的一些[安全插件的设置]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/security-settings/)配置。要了解更多信息，请参阅[Security]({{site.url}}{{site.baseurl}}/security/)。

### （可选）CORS 标头配置

如果你正在处理针对不同域上的 OpenSearch 集群运行的客户端应用程序，则可以配置标头 `opensearch.yml` 以允许在同一台计算机上开发本地应用程序。使用[跨域资源共享](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)，以便你的应用程序可以调用本地运行的 OpenSearch API。在 `custom-opensearch.yml` 文件中添加以下行（请注意，“-”必须是每行中的第一个字符）。
```yml
- http.host:0.0.0.0
- http.port:9200
- http.cors.allow-origin:"http://localhost"
- http.cors.enabled:true
- http.cors.allow-headers:X-Requested-With,X-Auth-Token,Content-Type,Content-Length,Authorization
- http.cors.allow-credentials:true
```