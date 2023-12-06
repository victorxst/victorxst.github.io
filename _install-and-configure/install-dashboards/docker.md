---
layout: default
title: Docker
parent: 安装 OpenSearch 控制面板
nav_order: 1
redirect_from: 
  - /dashboards/install/docker/
  - /opensearch/install/docker-security/
---

# 使用 Docker 运行 OpenSearch 控制面板

你可以使用*能* after [创建 Docker 网络](https://docs.docker.com/engine/reference/commandline/network_create/)和 start OpenSearch 启动 OpenSearch 控制面板，但使用 Docker Compose 文件将 OpenSearch 控制面板 `docker run` 连接到 OpenSearch 的过程要容易得多。

1. 运行 `docker pull opensearchproject/opensearch-dashboards:2`。

1. 创建适合你的环境的 [ `docker-compose.yml`]（https://docs.docker.com/compose/compose-file/）文件。OpenSearch 上提供了包含 OpenSearch [Docker 安装页面]({{site.url}}{{site.baseurl}}/opensearch/install/docker#sample-docker-composeyml)控制面板的示例文件。

   就像 `opensearch.yml` 一样，你可以将自定义 `opensearch_dashboards.yml` 项传递给 Docker Compose 文件中的容器。
{:.tip}

1. 运行 `docker-compose up`。

   等待容器启动。然后查看[OpenSearch 控制面板文档]({{site.url}}{{site.baseurl}}/dashboards/index/).

1. 完成后，运行 `docker-compose down`.
