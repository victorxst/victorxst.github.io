---
layout: default
title: IP2Geo
parent: 摄入的处理器
nav_order: 130
redirect_from:
   - /api-reference/ingest-apis/processors/ip2geo/
---

# IP2GEO
**引入2.10**
{: .label .label-purple }

这`ip2geo` 处理器添加了有关IPv4或IPv6地址的地理位置的信息。这`ip2geo` 处理器使用来自外部端点的IP Geolocation（GEOIP）数据，因此需要附加组件，`datasource`，从哪里定义下载GeoIP数据以及更新数据的频率。

{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/icons/info-icon.png" class ="inline-icon" alt ="info icon"/> {:/}**笔记**<br>`ip2geo` 处理器在系统索引中维护GEOIP数据映射。在数据摄入期间，从这些索引中检索了地理映射以执行IP-到-传入数据上的地理位置转换。为了获得最佳性能，最好具有具有摄入和数据角色的节点，因为此配置避免了internode调用减少延迟。另外，作为`ip2geo` 处理器搜索从索引中的GeoIP映射数据，搜索性能受到影响。
{: .note}

## 入门

开始`ip2geo` 处理器，`opensearch-geospatial` 必须安装插件。看[安装插件]({{site.url}}{{site.baseurl}}/install-and-configure/plugins/) 了解更多。

## 集群设置

IP2GEO数据源和`ip2geo` 处理器节点设置在下表中列出。该表中的所有设置都是动态的。要了解有关静态和动态设置的更多信息，请参阅[配置OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)。

| 钥匙| 描述| 默认|
|--------------------|-------------|---------|
| `plugins.geospatial.ip2geo.datasource.endpoint` | 创建数据源API的默认端点。| 默认为`https://geoip.maps.opensearch.org/v1/geolite2-city/manifest.json`。|
| `plugins.geospatial.ip2geo.datasource.update_interval_in_days` | 默认更新间隔用于创建数据源API。| 默认值为3。|
| `plugins.geospatial.ip2geo.datasource.batch_size` | 在IP2GEO数据源创建过程中，最大的文档数量在批量请求中摄入。| 默认值为10,000。|
| `plugins.geospatial.ip2geo.processor.cache_size` | 可以缓存的最大结果数。每个节点中的所有IP2GEO处理器都仅使用一个缓存。| 默认值为1,000。|
| `plugins.geospatial.ip2geo.timeout` | 等待端点和群集响应的时间。| 默认为30秒。|

## 创建IP2GEO数据源

在创建使用的管道之前`ip2geo` 处理器，创建IP2GEO数据源。数据源定义了将下载GeoIP数据并指定更新间隔的端点值。

OpenSearch为Geolite2 City，Geolite2 Country和Geolite2 ASN数据库提供以下端点[maxmind](https://dev.maxmind.com/geoip/geolite2-free-geolocation-data)，在[cc by-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) 执照：

* geolite2城市：https：//geoip.maps.opensearch.org/v1/geolite2-城市/清单
* geolite2国家：https：//geoip.maps.opensearch.org/v1/geolite2-国家/清单
* geolite2 asn：https：//geoip.maps.opensearch.org/v1/geolite2-asn/subest.json

如果OpenSearch Cluster无法在30天内从端点上更新数据源，则该集群不会将GeoIP数据添加到文档中，而是添加`"error":"ip2geo_data_expired"`。

#### 数据源选项

下表列出了的数据源选项`ip2geo` 处理器。

| 姓名| 必需的| 默认| 描述|
|------|----------|---------|-------------|
| `endpoint` | 选修的| https://geoip.maps.opensearch.org/v1/geolite2-城市/清单| 下载GeoIP数据的端点。|
| `update_interval_in_days` | 选修的| 3| 在几天内，GeoIP数据被更新的频率进行了更新。最小值为1。|

要创建IP2GEO数据源，请运行以下查询：

```json
PUT /_plugins/geospatial/ip2geo/datasource/my-datasource
{
    "endpoint" : "https://geoip.maps.opensearch.org/v1/geolite2-city/manifest.json",
    "update_interval_in_days" : 3
}
```
{% include copy-curl.html %}

A`true` 响应意味着请求成功，并且服务器能够处理请求。A`false` 响应表明您应该检查请求以确保其有效，请检查URL以确保其正确或重试。

#### 发送get请求

要获取有关一个或多个IP2GEO数据源的信息，请发送GET请求：

```json
GET /_plugins/geospatial/ip2geo/datasource/my-datasource
```
{% include copy-curl.html %}

您将收到以下答复：

```json
{
  "datasources": [
    {
      "name": "my-datasource",
      "state": "AVAILABLE",
      "endpoint": "https://geoip.maps.opensearch.org/v1/geolite2-city/manifest.json",
      "update_interval_in_days": 3,
      "next_update_at_in_epoch_millis": 1685125612373,
      "database": {
        "provider": "maxmind",
        "sha256_hash": "0SmTZgtTRjWa5lXR+XFCqrZcT495jL5XUcJlpMj0uEA=",
        "updated_at_in_epoch_millis": 1684429230000,
        "valid_for_in_days": 30,
        "fields": [
          "country_iso_code",
          "country_name",
          "continent_name",
          "region_iso_code",
          "region_name",
          "city_name",
          "time_zone",
          "location"
        ]
      },
      "update_stats": {
        "last_succeeded_at_in_epoch_millis": 1684866730192,
        "last_processing_time_in_millis": 317640,
        "last_failed_at_in_epoch_millis": 1684866730492,
        "last_skipped_at_in_epoch_millis": 1684866730292
      }
    }
  ]
}
```

#### 更新IP2GEO数据源

有关端点列表和请求字段说明，请参见创建IP2GEO数据源部分。

要更新日期源，请运行以下查询：

```json
PUT /_plugins/geospatial/ip2geo/datasource/my-datasource/_settings
{
    "endpoint": https://geoip.maps.opensearch.org/v1/geolite2-city/manifest.json,
    "update_interval_in_days": 10
}
```
{% include copy-curl.html %}

#### 删除IP2GEO数据源

要删除IP2GEO数据源，您必须首先删除与数据源关联的所有处理器。否则，请求失败。

要删除数据源，请运行以下查询：

```json
DELETE /_plugins/geospatial/ip2geo/datasource/my-datasource
```
{% include copy-curl.html %}

## 创建管道

创建数据源后，您可以创建管道。以下是`ip2geo` 处理器：

```json 
{
  "ip2geo": {
    "field":"ip",
    "datasource":"my-datasource"
  }
}
```
{% include copy-curl.html %}

#### 配置参数

下表列出了所需的和可选参数`ip2geo` 处理器。

| 姓名| 必需的| 默认| 描述|
|------|----------|---------|-------------|
| `datasource` | 必需的| - | 用于检索地理信息的数据源名称。|
| `field` | 必需的| - | 包含地理查找的IP地址的字段。|
| `ignore_missing` | 选修的| 错误的| 如果设置为`true`，如果字段不存在或为`null`。默认为`false`。|
| `properties` | 选修的|  所有字段中`datasource` | 控制哪些属性添加到`target_field` 从`datasource`。|
| `target_field` | 选修的| IP2GEO| 包含从数据源检索到的地理信息的字段。|

## 使用处理器

按照以下步骤在管道中使用处理器。

**步骤1：创建管道。**

以下查询创建了一个命名的管道`my-pipeline`，将IP地址转换为地理信息：

```json
PUT /_ingest/pipeline/my-pipeline
{
   "description":"convert ip to geo",
   "processors":[
    {
        "ip2geo":{
            "field":"ip",
            "datasource":"my-datasource"
        }
    }
   ] 
}
```
{% include copy-curl.html %}

**步骤2（可选）：测试管道。**

{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/icons/info-icon.png" class ="inline-icon" alt ="info icon"/> {:/}**笔记**<br>建议您在摄入文档之前测试管道。
{: .note}

要测试管道，请运行以下查询：

```json
POST _ingest/pipeline/my-pipeline/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "ip": "172.0.0.1",
      }
    }
  ]
}
```

#### 回复

以下响应证实了管道正常工作：

```json
{
  "docs": [
    {
      "_index":"testindex1",
      "_id":"1",
      "_source":{
        "ip":"172.0.0.1",
        "ip2geo":{
         "continent_name":"North America",
         "region_iso_code":"AL",
         "city_name":"Calera",
         "country_iso_code":"US",
         "country_name":"United States",
         "region_name":"Alabama",
         "location":"33.1063,-86.7583",
         "time_zone":"America/Chicago"
         }
      }
    }
  ]
}
```
{% include copy-curl.html %}

**步骤3：摄取文档。**

以下查询将文档摄入到名为的索引中`my-index`：

```json
PUT /my-index/_doc/my-id?pipeline=ip2geo
{
  "ip": "172.0.0.1"
}
```
{% include copy-curl.html %}

**步骤4（可选）：检索文档。** 

要检索文档，请运行以下查询：

```json
GET /my-index/_doc/my-id
```
{% include copy-curl.html %}

