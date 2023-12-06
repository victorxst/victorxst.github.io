---
layout: default
title: 使用自托管地图服务器
grand_parent: 构建数据可视化
parent: 使用坐标和区域图
nav_order: 40
redirect_from:
  - /dashboards/selfhost-maps-server/
---

# 使用自托管地图服务器

自己-OpenSearch仪表板的主机地图服务器允许用户访问AIR中的默认地图服务-间隙环境。OpenSearch-兼容的地图URL包括带有地图图块和向量，地图图块和地图向量的地图清单。

以下各节提供了设置和使用自我的步骤-带有OpenSearch仪表板的主机地图服务器。

您可以访问`maps-server` 通过官方搜索图像[Docker Hub存储库](https://hub.docker.com/u/opensearchproject)。
{:.note}

## 拉码头图像

打开终端并运行以下命令：

`docker pull opensearchproject/opensearch-maps-server:1.0.0`

## 设置服务器

在运行服务器之前，您必须设置地图图块。您有两个设置选项：使用OpenSearch-提供了地图服务瓷砖设置，或生成栅格图块集。

### 选项1：使用OpenSearch-提供地图服务瓷砖集

创建一个码头卷以保持瓷砖集：

`docker volume create tiles-data`

从OpenSearch Maps服务下载设置的图块。根据所需的变焦级别可用两个行星瓷砖集：

- Zoom Level 8（https://maps.opensearch.org/offline/planet-OSM-默认-Z0-z8.tar.gz）
- Zoom Level 10（https://maps.opensearch.org/offline/planet-OSM-默认-Z0-z10.tar.gz）

缩放级别10（2 GB压缩/6.8 GB未压缩）的行星瓷砖比缩放级别8（225 MB压缩/519 MB未压缩）的集合大约10倍。
{:.note}

```
docker run \
    -e DOWNLOAD_TILES=https://maps.opensearch.org/offline/planet-osm-default-z0-z8.tar.gz \
    -v tiles-data:/usr/src/app/public/tiles/data/ \
    opensearch/opensearch-maps-server \
    import
```

### 选项2：生成栅格瓷砖集

要生成栅格瓷砖集，请使用[栅格瓷砖生成管道](https://github.com/opensearch-project/maps/tree/main/tiles-generation/cdk) 然后使用图块设置的绝对路径来创建卷以启动服务器。

## 启动服务器

使用以下命令使用Docker卷启动服务器`tiles-data`。以下命令是使用主机URL的示例"localhost" 和端口"8080"：

```
docker run \
    -v tiles-data:/usr/src/app/public/tiles/data/ \
    -e HOST_URL='http://localhost' \
    -p 8080:8080 \
    opensearch/opensearch-maps-server \
    run
```

或者，如果您生成了栅格图块集，请使用该图块设置运行服务器：

```
docker run \
    -v /absolute/path/to/tiles/:/usr/src/app/dist/public/tiles/data/ \
    -p 8080:8080 \
    opensearch/opensearch-maps-server \
    run
```
要访问瓷砖集，请在主机上的浏览器中打开URL或使用`curl` 命令`curl http://localhost:8080/manifest.json`。


通过打开主机上的浏览器或使用一个以下每个链接来确认服务器正在运行`curl` 命令（例如，`curl http://localhost:8080/manifest.json`）。

*地图清单URL：`http://localhost:8080/manifest.json`
*地图图块URL：`http://localhost:8080/tiles/data/{z}/{x}/{y}.png`
*地图瓷砖演示网址：`http://localhost:8080/`

## 使用自我-带有OpenSearch仪表板的主机地图服务器

你可以使用自我-通过将参数添加到`opensearch_dashboards.yml` 或在OpenSearch仪表板中配置默认WMS属性。

### 选项1：配置opensearch_dashboards.yml

配置清单URL`opensearch_dashboards.yml`：

`map.opensearchManifestServiceUrl: "http://localhost:8080/manifest.json"`

### 选项2：配置OpenSearch仪表板中的默认WMS属性

1. 在OpenSearch仪表板控制台上，选择**仪表板管理** >**高级设置**。
2. 定位`visualization:tileMap:WMSdefaults` 在下面**默认WMS属性**。
3. 改变`"enabled": false` 到`"enabled": true` 并为有效的地图服务器添加URL。

## 许可证

瓷砖是生成的[天然地球矢量图数据的使用条款](https://www.naturalearthdata.com/about/terms-of-use/) 和[OpenStreetMap的版权和许可证](https://www.openstreetmap.org/copyright)。

## 相关文章

*[配置网络地图服务（WMS）]({{site.url}}{{site.baseurl}}/dashboards/visualize/maptiles/)
*[使用坐标和区域图]({{site.url}}{{site.baseurl}}/dashboards/visualize/geojson-regionmaps/)

