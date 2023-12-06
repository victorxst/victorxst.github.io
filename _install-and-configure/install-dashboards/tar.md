---
layout: default
title: Tarball
parent: 安装 OpenSearch 控制面板
nav_order: 30
redirect_from: 
  - /dashboards/install/tar/
---

# 使用 tarball 运行 OpenSearch 控制面板

1. 从[OpenSearch 下载页面](https://opensearch.org/downloads.html){:target='\_blank'} 下载压缩包。

1. 将 TAR 文件解压缩到某个目录，然后切换到该目录：

   ```bash
   # x64
   tar -zxf opensearch-dashboards-{{site.opensearch_version}}-linux-x64.tar.gz
   cd opensearch-dashboards
   # ARM64
   tar -zxf opensearch-dashboards-{{site.opensearch_version}}-linux-arm64.tar.gz
   cd opensearch-dashboards
   ```

1. 如果需要，请修改 `config/opensearch_dashboards.yml`.

1. 运行 OpenSearch 控制面板：

   ```bash
   ./bin/opensearch-dashboards
   ```
