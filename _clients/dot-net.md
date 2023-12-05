---
layout: default
title: .NET客户端
nav_order: 75
has_children: true
has_toc: false
---

# .NET客户端

OpenSearch有两个.NET客户端：低-等级[opensearch.net]({{site.url}}{{site.baseurl}}/clients/OpenSearch-dot-net/) 客户和高位-等级[OpenSearch.Client]({{site.url}}{{site.baseurl}}/clients/OSC-dot-net/) 客户。

[opensearch.net]({{site.url}}{{site.baseurl}}/clients/OpenSearch-dot-net/) 很低-Level .NET客户端，可通过OpenSearch提供通信的基础层。它是无依赖性的，它可以处理圆-罗宾负载平衡，运输和基本请求/响应周期。OpenSearch.net包含所有OpenSearch API端点的方法。

[OpenSearch.Client]({{site.url}}{{site.baseurl}}/clients/OSC-dot-net/) 很高-在OpenSearch.net之上的Level .NET客户端。它提供了强烈键入的请求和响应以及查询DSL。它可以通过提供解析和序列化/值得序列化请求和响应的模型来自动构建RAW JSON请求和解析RAW JSON响应。openSearch.client还公开了opensearch.net low-如果需要，请级别客户端。OpenSearch.Client包括以下高级功能：

- 自动化：给定一个C# 类型，OpenSearch.Client可以推断出正确的映射以发送到OpenSearch。
- 操作员在查询中超载。
- 类型和索引推理。

您可以在控制台程序，.NET核心，ASP.NET核心或Worker Services中使用.NET客户端。

要开始使用OpenSearch.Client，请按照说明中的说明[从高处开始-级别.NET客户端]({{site.url}}{{site.baseurl}}/clients/OSC-dot-net#installing-opensearchclient) 或IN[高级功能-级别.NET客户端]({{site.url}}{{site.baseurl}}/clients/OSC-example)，稍微高级的演练

