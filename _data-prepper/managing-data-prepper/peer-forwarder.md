---
layout: default
title: 同行前锋
nav_order: 12
parent: 管理数据预先
---

# 同行前锋

Peer Fewracker是一项HTTP服务，可执行`event` 在数据预先淋巴结之间进行聚合。此HTTP服务使用哈希-汇总事件的环方法并确定在将其重新路由到该节点之前应在给定跟踪上处理的数据预先节点。目前，同行货运员得到`aggregate`，`service_map_stateful`， 和`otel_traces_raw` [处理器]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/processors/processors/)。

同行转发器根据支持的处理器提供的标识密钥将事件分组。为了`service_map_stateful` 和`otel_traces_raw`，标识密钥是`traceId` 默认情况下，无法配置。这`aggregate` 处理器是使用`identification_keys` 配置选项。从这里，您可以指定用于Peer Forwarder的键。看[聚合处理器页面](https://github.com/opensearch-project/data-prepper/tree/main/data-prepper-plugins/aggregate-processor#identification_keys) 有关识别键的更多信息。

同行发现允许数据预先找到它将与之通信的其他节点。当前，同行发现由静态列表，DNS记录查找或AWS云图提供。

## 发现模式

以下各节提供了有关发现模式的信息。

### 静止的

静态发现模式允许数据Prepper节点使用IP地址或域名列表来发现节点。有关静态发现模式的示例，请参见以下YAML文件：

```yaml
peer_forwarder:4
  discovery_mode: static
  static_endpoints: ["data-prepper1", "data-prepper2"]
```

### DNS查找

在扩展数据PEPPER群集时，DNS发现比静态发现更优选。DNS Discovery配置DNS提供商在给出单个域名时返回Prepper主机的列表。此列表包括一个[DNS记录](https://www.cloudflare.com/learning/dns/dns-records/dns-a-record/)，以及给定域的IP地址列表。有关DNS查找的示例，请参见以下YAML文件：

```yaml
peer_forwarder:
  discovery_mode: dns
  domain_name: "data-prepper-cluster.my-domain.net"
```

### AWS云图

[AWS云图](https://docs.aws.amazon.com/cloud-map/latest/dg/what-is-cloud-map.html) 提供API-基于服务发现以及DNS-基于服务发现。

同行转发器可以使用API-AWS云图中的基于服务发现。为了支持这一点，您必须具有为API实例发现配置的现有名称空间。您可以通过按照该说明来创建一个新的说明[AWS云地图文档](https://docs.aws.amazon.com/cloud-map/latest/dg/working-with-namespaces.html)。

您的数据预先配置需要包括以下内容：
*`aws_cloud_map_namespace_name`  - 设置为AWS云地图名称名称。
*`aws_cloud_map_service_name`  - 设置为指定名称空间中的服务名称。
*`aws_region`  - 设置为存在您的命名空间的AWS区域。
*`discovery_mode` - 设置`aws_cloud_map`。

您的数据Prepper配置可以选择包括以下内容：
*`aws_cloud_map_query_parameters` - 钥匙-价值对用于根据附加到实例的自定义属性过滤结果。结果仅包括匹配所有指定密钥的实例-价值对。

#### 示例配置

请参阅以下AWS云映射配置的YAML文件示例：

```yaml
peer_forwarder:
  discovery_mode: aws_cloud_map
  aws_cloud_map_namespace_name: "my-namespace"
  aws_cloud_map_service_name: "data-prepper-cluster"
  aws_cloud_map_query_parameters:
    instance_type: "r5.xlarge"
  aws_region: "us-east-1"
```

### IAM政策具有必要的许可

Data Prepper还必须在必要的权限下运行。以下AWS身份和访问管理（IAM）策略显示了必要的权限：

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "CloudMapPeerForwarder",
            "Effect": "Allow",
            "Action": "servicediscovery:DiscoverInstances",
            "Resource": "*"
        }
    ]
}
```


## 配置

下表提供了可选的配置值。


| 价值| 类型| 描述|
| ----  | --- |  ----------- |
| `port` | 整数| 0到65535之间的值表示对等转发器服务器正在运行的端口。默认值是`4994`。|
| `request_timeout` | 整数| 代表对等转发器HTTP服务器的毫秒中的请求超时持续时间。默认值是`10000`。|
| `server_thread_count` | 整数| 表示对等转发器服务器使用的线程数。默认值是`200`。|
| `client_thread_count` | 整数| 表示对等转发器客户端使用的线程数。默认值是`200`。|
| `maxConnectionCount`  | 整数| 代表对等转发器服务器的最大开放连接数量。默认值是`500`。|
| `discovery_mode` | 细绳| 表示要使用的同伴发现模式。允许的值是`local_node`，`static`，`dns`， 和`aws_cloud_map`。默认为`local_node`，哪个在本地处理事件。|
| `static_endpoints` | 列表| 包含所有数据预先实例的端点。如果需要`discovery_mode` 被设定为`static`。|
|  `domain_name` | 细绳| 代表查询DNS的单个域名。通常通过创建多个[DNS记录](https://www.cloudflare.com/learning/dns/dns-records/dns-a-record/) 对于相同的域。如果需要`discovery_mode` 被设定为`dns`。|
| `aws_cloud_map_namespace_name`  | 细绳| 使用AWS云映射服务发现时，表示AWS云映射名称空间。如果需要`discovery_mode` 被设定为`aws_cloud_map`。|
| `aws_cloud_map_service_name` | 细绳| 使用AWS Cloud Map Service Discovery时，表示AWS云映射服务。如果需要`discovery_mode` 被设定为`aws_cloud_map`。|
| `aws_cloud_map_query_parameters`  | 地图| 钥匙-值对，用于根据附加到实例的自定义属性过滤结果。只有匹配所有指定密钥的实例-返回价值对。|
| `buffer_size` | 整数| 表示缓冲区接受的最大未检查记录数（未检查的记录的数量等于写入缓冲区中的记录数，以及仍在处理的记录数量，尚未由检查点API检查）。默认为`512`。|
| `batch_size` |  整数| 表示缓冲区在读取时返回的最大记录数量。默认为`48`。|
|  `aws_region` |  细绳| 代表使用的AWS区域`ACM`，`Amazon S3`， 或者`AWS Cloud Map` 并且在满足以下任何条件时需要：<br>- 这`use_acm_certificate_for_ssl` 设置设置为`true`。<br>- 任何一个`ssl_certificate_file` 或者`ssl_key_file` 指定Amazon简单存储服务（Amazon S3）URI（例如，S3：//mybucket/path/to/to/public.cert）。<br>- 这`discovery_mode` 被设定为`aws_cloud_map`。|
| `drain_timeout`  | 期间| 表示Peer Forwarder等待在关闭之前完成数据处理的时间。|

## SSL配置

下表提供了可选的SSL配置值，使您可以为PEER EXPRANER客户端设置信任管理器，以连接到其他数据预先实例。

| 价值| 类型| 描述|
| ----- | ---- | ----------- |
| `ssl` | 布尔| 启用TLS/SSL。默认值是`true`。|
| `ssl_certificate_file`| 细绳| 表示SSL证书链文件路径或Amazon S3路径。以下是Amazon S3路径的示例：`s3://<bucketName>/<path>`。默认为默认证书文件，`config/default_certificate.pem`。看[默认证书](https://github.com/opensearch-project/data-prepper/tree/main/examples/certificates) 有关如何生成证书的更多信息。|
| `ssl_key_file`| 细绳| 表示SSL密钥文件路径或Amazon S3路径。亚马逊S3路径示例：`s3://<bucketName>/<path>`。默认为`config/default_private_key.pem` 这是默认的私钥文件。看[默认证书](https://github.com/opensearch-project/data-prepper/tree/main/examples/certificates) 有关如何生成私钥文件的更多信息。|
| `ssl_insecure_disable_verification` | 布尔| 禁用服务器TLS证书链的验证。默认值是`false`。|
| `ssl_fingerprint_verification_only` | 布尔| 禁用服务器的TLS证书链的验证，而仅验证证书指纹。默认值是`false`。|
| `use_acm_certificate_for_ssl` | 布尔| 使用AWS证书经理（ACM）的证书和私钥启用TLS/SSL。默认值是`false`。|
| `acm_certificate_arn`| 细绳| 代表ACM证书亚马逊资源名称（ARN）。ACM证书优先于Amazon S3或本地文件系统证书。如果需要`use_acm_certificate_for_ssl` 被设定为`true`。|
| `acm_private_key_password` | 细绳| 表示将用于解密私钥的ACM私钥密码。如果未提供，将生成一个随机密码。|
| `acm_certificate_timeout_millis` | 整数| 代表ACM获得证书所需的毫秒中的超时。默认值是`120000`。|
| `aws_region` | 细绳| 代表使用ACM，Amazon S3或AWS云图的AWS区域。如果需要`use_acm_certificate_for_ssl` 被设定为`true` 或者`ssl_certificate_file`。也需要`ssl_key_file` 设置为使用Amazon S3路径或`discovery_mode` 被设定为`aws_cloud_map`。|

#### 示例配置

以下YAML文件提供了一个示例配置：

```yaml
peer_forwarder:
  ssl: true
  ssl_certificate_file: "<cert-file-path>"
  ssl_key_file: "<private-key-file-path>"
```

## 验证

`Authentication` 是可选的，是`Map` 这样可以实现相互的TLS（MTL）。可以是`mutual_tls` 或者`unauthenticated`。默认值是`unauthenticated`。以下YAML文件提供了身份验证的示例：

```yaml
peer_forwarder:
  authentication:
    mutual_tls:
```

## 指标

Core Peer Fewracker介绍以下自定义指标。所有指标都由`core.peerForwarder`。

### 计时器

PEER Forwarder的计时器功能提供以下信息：

- `requestForwardingLatency`：衡量同伴转发器客户端转发的请求的延迟。
- `requestProcessingLatency`：测量PEER Forwarder Server处理的请求的延迟。

### 柜台

下表提供了计数器选项。

| 价值| 描述|
| ----- | ----------- |
| `requests`| 衡量转发请求的总数。|
| `requestsFailed`| 测量失败请求的总数。适用于具有HTTP响应代码的请求`200`。|
| `requestsSuccessful`|  衡量成功请求的总数。适用于使用HTTP响应代码的请求`200`。|
| `requestsTooLarge`| 衡量太大的请求总数，无法写入对等转发器缓冲区。适用于使用HTTP响应代码的请求`413`。|
| `requestTimeouts`| 在将内容写入对等转发器缓冲区时，衡量该时间的请求总数。适用于使用HTTP响应代码的请求`408`。|
| `requestsUnprocessable`| 衡量由于无法处理的实体而失败的请求总数。适用于使用HTTP响应代码的请求`422`。|
| `badRequests`| 用不良的请求格式衡量请求总数。适用于使用HTTP响应代码的请求`400`。|
| `recordsSuccessfullyForwarded`| 衡量成功转发记录的总数。|
| `recordsFailedForwarding`| 衡量未转发的记录总数。|
| `recordsToBeForwarded` | 衡量要转发的记录总数。|
| `recordsToBeProcessedLocally` | 衡量要在本地处理的记录总数。|
| `recordsActuallyProcessedLocally`| 衡量实际处理的记录总数。此值是`recordsToBeProcessedLocally` 和`recordsFailedForwarding`。|
| `recordsReceivedFromPeers`| 测量从远程同行收到的记录总数。|

### 测量

`peerEndpoints` 测量动态发现的同行数据预先端点的数量。为了`static` 模式，大小是固定的。

