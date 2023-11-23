---
layout: default
title: 网络设置
parent: 配置 OpenSearch
nav_order: 20
---

# 网络设置

OpenSearch 使用 HTTP 设置通过 REST API 配置与外部客户端的通信，并使用传输设置在 OpenSearch 中进行内部节点到节点通信。

要了解有关静态和动态设置的详细信息，请参阅[配置 OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)。

OpenSearch 支持以下常见网络设置：

-  `network.host`（Static，list）：将 OpenSearch 节点绑定到地址。用于 `0.0.0.0` 包括所有可用的网络接口，或指定分配给特定接口的 IP 地址。如果它们的值相同，则 `network.host` 设置为和 `network.publish_host` 的组合 `network.bind_host`。另一种方法是 `network.host` 根据需要进行配置和 `network.publish_host` 单独配置 `network.bind_host`。请参见[高级网络设置](#advanced-network-settings)。

-  `http.port`（静态、单值或范围）：将 OpenSearch 节点绑定到用于 HTTP 通信的自定义端口或端口范围。你可以指定地址或地址范围。缺省值为 `9200-9300`。

-  `transport.port`（静态、单值或范围）：将 OpenSearch 节点绑定到自定义端口，以便节点之间进行通信。你可以指定地址或地址范围。缺省值为 `9300-9400`。

## 高级网络设置

OpenSearch 支持以下高级网络设置：

-  `network.bind_host`（静态、列表）：将 OpenSearch 节点绑定到传入连接的一个或多个地址。缺省值为中的值 `network.host`。

-  `network.publish_host`（静态，列表）：指定 OpenSearch 节点发布到集群中其他节点的一个或多个地址，以便它们可以连接到该节点。

## 高级 HTTP 设置

OpenSearch 支持以下用于 HTTP 通信的高级网络设置：

-  `http.host`（Static，list）：设置用于 HTTP 通信的 OpenSearch 节点的地址。如果它们的值相同，则 `http.host` 设置为和 `http.publish_host` 的组合 `http.bind_host`。另一种方法是 `http.host` 根据需要进行配置和 `http.publish_host` 单独配置 `http.bind_host`。

-  `http.bind_host`（Static，list）：指定 OpenSearch 节点绑定到以侦听传入 HTTP 连接的一个或多个地址。

-  `http.publish_host`（Static，list）：指定 OpenSearch 节点发布到其他节点以进行 HTTP 通信的一个或多个地址。

## 高级传输设置

OpenSearch 支持以下用于传输通信的高级网络设置：

-  `transport.host`（Static，list）：设置用于传输通信的 OpenSearch 节点的地址。如果它们的值相同，则 `transport.host` 设置为和 `transport.publish_host` 的组合 `transport.bind_host`。另一种方法是 `transport.host` 根据需要进行配置和 `transport.publish_host` 单独配置 `transport.bind_host`。

-  `transport.bind_host`（Static，list）：指定 OpenSearch 节点绑定到的一个或多个地址，以侦听传入的传输连接。

-  `transport.publish_host`（Static，list）：指定 OpenSearch 节点发布到其他节点以进行传输通信的一个或多个地址。
