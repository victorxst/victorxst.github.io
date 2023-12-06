---
layout: default
title: 插件设置
parent: 配置 OpenSearch
nav_order: 100
---

# 插件设置

以下设置与 OpenSearch 插件相关。

## 警报插件设置

有关警报设置的信息，请参见[警报设置]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/settings/#alerting-settings)。

## 异常检测插件设置

有关异常检测设置的信息，请参阅[异常检测设置]({{site.url}}{{site.baseurl}}/observing-your-data/ad/settings/)。

## 异步搜索插件设置

有关异步搜索设置的信息，请参见[异步搜索设置]({{site.url}}{{site.baseurl}}/search-plugins/async/settings/)。

## 跨集群复制插件设置

有关跨集群复制设置的信息，请参见[复制设置]({{site.url}}{{site.baseurl}}/tuning-your-cluster/replication-plugin/settings/)。

## 地理空间插件设置

有关地理空间插件的 IP2Geo 处理器设置的信息，请参阅[群集设置]({{site.url}}{{site.baseurl}}/ingest-pipelines/processors/ip2geo/#cluster-settings)。

## 索引管理插件设置

有关索引状态管理（ISM）设置的信息，请参阅[ISM 设置]({{site.url}}{{site.baseurl}}/im-plugin/ism/settings/)。

### 索引汇总设置

有关索引汇总设置的信息，请参阅[索引汇总设置]({{site.url}}{{site.baseurl}}/im-plugin/index-rollups/settings/)。

## Job Scheduler 插件设置

有关 Job Scheduler 插件设置的信息，请参阅[作业计划程序群集设置]({{site.url}}{{site.baseurl}}/monitoring-your-cluster/job-scheduler/index/#job-scheduler-cluster-settings)。

## k-NN 插件设置

有关 k-NN 设置的信息，请参见[k-NN 设置]({{site.url}}{{site.baseurl}}/search-plugins/knn/settings/)。

## ML Commons 插件设置

有关机器学习设置的信息，请参阅[ML Commons 集群设置]({{site.url}}{{site.baseurl}}/ml-commons-plugin/cluster-settings/)。

## 神经搜索插件设置

Security Analytics 插件支持以下设置：

-  `plugins.neural_search.hybrid_search_disabled`（动态、布尔值）：禁用混合搜索。缺省值为 `false`。

## 通知插件设置

通知插件支持以下设置。此列表中的所有设置都是动态的：

-  `opensearch.notifications.core.allowed_config_types`（列表）：通知插件允许的配置类型。 `GET/_plugins/_notifications/features` 使用 API 检索此设置的值。配置类型包括 `slack`、 `sns` `smtp_account` `ses_account` `email` `webhook` `email_group`、 `chime` `microsoft_teams` 和。

-  `opensearch.notifications.core.email.minimum_header_length`（整数）：最小电子邮件标题长度。用于电子邮件总长度验证。缺省值为 `160`。

-  `opensearch.notifications.core.email.size_limit`（整数）：电子邮件大小限制。用于电子邮件总长度验证。缺省值为 `10000000`。

-  `opensearch.notifications.core.http.connection_timeout`（整数）：内部 HTTP 客户端连接超时。客户端用于基于 Webhook 的通知通道。缺省值为 `5000`。

-  `opensearch.notifications.core.http.host_deny_list`（列表）：被拒绝的主机列表。HTTP 客户端不会向此列表中的 Webhook URL 发送通知。

-  `opensearch.notifications.core.http.max_connection_per_route`（整数）：内部 HTTP 客户端每条路由的最大 HTTP 连接数。客户端用于基于 Webhook 的通知通道。缺省值为 `20`。

-  `opensearch.notifications.core.http.max_connections`（Integer）：内部 HTTP 客户端的最大 HTTP 连接数。客户端用于基于 Webhook 的通知通道。缺省值为 `60`。

-  `opensearch.notifications.core.http.socket_timeout`（Integer）：内部 HTTP 客户端的套接字超时配置。客户端用于基于 Webhook 的通知通道。缺省值为 `50000`。

-  `opensearch.notifications.core.tooltip_support`（布尔值）：启用对通知插件的工具提示支持。 `GET/_plugins/_notifications/features` 使用 API 检索此设置的值。缺省值为 `true`。

-  `opensearch.notifications.general.filter_by_backend_roles`（布尔值）：启用按后端角色进行筛选（通知通道的基于角色的访问控制）。缺省值为 `false`。

## 安全插件设置

有关安全插件设置的信息，请参阅[安全设置]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/security-settings/)。

## Security Analytics 插件设置

有关安全分析设置的信息，请参阅[Security Analytics 设置]({{site.url}}{{site.baseurl}}/security-analytics/settings/)。

## SQL 插件设置

有关与 SQL 和 PPL 相关的设置的信息，请参见[SQL 设置]({{site.url}}{{site.baseurl}}/search-plugins/sql/settings/)。