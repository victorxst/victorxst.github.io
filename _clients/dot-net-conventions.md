---
layout: default
title: .NET客户端的注意事项
nav_order: 20
has_children: false
parent: .NET客户端
---

# .NET客户的考虑和最佳实践

以下各节提供了有关使用.NET客户端的注意事项和最佳实践的信息。

## 注册opensearch.client作为单身人士

通常，您应该设置opensearch.client作为单身人士。OpenSearch.Client管理群集中与节点的连接。此外，每个客户端为其设置使用了很多配置。因此，创建一个openSearch.client实例并将其重复使用用于所有OpenSearch操作是有益的。客户端是线程安全的，因此可以通过多个线程共享相同的实例。

## 例外

以下是.NET客户端可能会抛出的例外类型：

- `OpenSearchClientException` 是请求管道中发生的已知例外（例如，到达超时）或opensearch（例如，畸形查询）。如果是opensearch例外，`ServerError` 响应属性包含OpenSearch返回的错误。
- `UnexpectedOpenSearchClientException` 是一个未知的例外（例如，在避难所期间发生错误），并且是OpenSearchClientException的子类。
- 当不正确使用API时，会抛出系统异常。

## 节点

要创建节点，请通过`Uri` 对象进入其构造函数：

```cs
var uri = new Uri("http://example.org/opensearch");
var node = new Node(uri);
```
{% include copy.html %}

首次创建时，一个节点是符合人资格的，它的`HoldsData` 属性设置为true。
这`AbsolutePath` 上面创建的节点的属性是`"/opensearch/"`：附加了尾随的前向斜线，以便可以轻松地组合路径。如果未指定，默认`Port` 是80。

如果节点具有相同的端点，则将其视为平等。在检查节点时，未考虑元数据是否相等。
{:.note}

## 连接池

连接池是`IConnectionPool` 并负责管理OpenSearch集群中的节点。我们建议创建一个[Singleton客户](#registering-opensearchclient-as-a-singleton) 与单个`ConnectionSettings` 目的。客户及其一生`ConnectionSettings` 是应用程序的寿命。

以下是连接池类型。

- **Singlenodeconnectionpool**

`SingleNodeConnectionPool` 是如果没有连接池传递给该连接池的默认连接池`ConnectionSettings` 构造函数。使用`SingleNodeConnectionPool` 如果群集中只有一个节点，或者群集的负载平衡器作为入口点。`SingleNodeConnectionPool` 不支持嗅探或刺，也不支持节点死亡或活着。

- **CloudConnectionpool**

`CloudConnectionPool` 是`SingleNodeConnectionPool` 这需要云ID和凭据。喜欢`SingleNodeConnectionPool`，`CloudConnectionPool` 不支持嗅探或刺。

- **staticconnectionpool**

`StaticConnectionPool` 当您不想打开嗅探以了解群集拓扑时，可用于小集群。`StaticConnectionPool` 不支持嗅探，但可以支持ping。

- **SniffingConnectionpool**

`SniffingConnectionPool` 是`StaticConnectionPool`。它是安全的，并支持嗅探和刺。`SniffingConnectionPool` 可以在运行时重新播放，您可以在播种时指定节点角色。

- **StickyConnectionpool**

`StickyConnectionPool` 设置为返回第一个实时节点，然后在请求之间持续存在。可以使用枚举的`Uri` 或者`Node` 对象。`StickyConnectionPool` 不支持嗅探，而是支持ping。

- **StickySniffingConnectionpool**

`StickySniffingConnectionPool` 是`SniffingConnectionPool`。喜欢`StickyConnectionPool`，它返回第一个Live Node2，然后在请求之间持续存在。`StickySniffingConnectionPool` 支持嗅探和分类，以便应用程序的每个实例都可以偏爱其他节点。节点具有与之相关的权重，并且可以按重量进行排序。

## 重试

如果请求未成功，则将自动重新进行。默认情况下，重试的数量是opensearch.clent中已知的节点数量。重试的数量也受超时参数的限制，因此OpenSearch.Clent.client重试的请求在超时期内尽可能多次。

要设置最大的检索数，请指定`MaximumRetries` 在`ConnectionSettings` 目的。

```cs
var settings = new ConnectionSettings(connectionPool).MaximumRetries(5);
```
{% include copy.html %}

您也可以设置一个`RequestTimeout` 这指定了单个请求和一个超时`MaxRetryTimeout` 这指定了所有重试尝试的时间限制。在下面的示例中，`RequestTimeout` 设置为4秒，并且`MaxRetryTimeout` 设置为12秒，因此查询的最大尝试数为3。

```cs
var settings = new ConnectionSettings(connectionPool)
            .RequestTimeout(TimeSpan.FromSeconds(4))
            .MaxRetryTimeout(TimeSpan.FromSeconds(12));
```
{% include copy.html %}

## 故障转移

如果您使用具有多个节点的连接池，则请求如果返回502（不良网关），503（服务不可用）或504（网关超时）HTTP错误响应代码，则将重试。如果响应代码是400–501或505–599范围中的错误代码，则请求未重新进行。

如果响应代码在2xx范围内，则认为响应是有效的，或者响应代码具有此请求的预期值之一。例如，404（未找到）是检查是否存在索引的请求的有效响应

