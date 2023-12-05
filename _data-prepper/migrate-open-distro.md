---
layout: default
title: 从开放式发行中迁移
nav_order: 30
---

# 从开放式发行中迁移

现有用户可以从开放的发行数据预先迁移到OpenSearch Data Prepper。从数据Prepper版本1.1开始，只有一个OpenSearch Data Prepper的分布。

## 更改管道配置

这`elasticsearch` 水槽已更改为`opensearch`。因此，更改现有管道以使用`opensearch` 插件而不是`elasticsearch`。

数据预先插件的标题为`opensearch`，它与开放的发行版和Elasticsearch 7.x保持兼容。
{： 。笔记}

## 更新Docker映像

在您的数据中，Prepper Docker配置，调整`amazon/opendistro-for-elasticsearch-data-prepper` 到`opensearchproject/data-prepper`。此更改将下载最新的数据Prepper Docker映像。

## 下一步

有关数据预先配置的更多信息，请参见[开始使用数据预先]({{site.url}}{{site.baseurl}}/clients/data-prepper/get-started/)。

