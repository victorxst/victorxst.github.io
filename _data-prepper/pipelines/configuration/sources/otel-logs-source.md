---
layout: default
title: otel_logs_source 
parent: 来源
grand_parent: 管道
nav_order: 25
---

# otel_logs_source


这`otel_logs_source` 来源是遵循opentelemetry的源[OPENTELEMETRY协议规范](https://github.com/open-telemetry/oteps/blob/master/text/0035-opentelemetry-protocol.md) 并以`ExportLogsServiceRequest` 记录。

此来源支持`OTLP/gRPC` 协议。
{: .note}

## 配置

您可以配置`otel_logs_source` 带有以下选项的源。

| 选项| 类型| 描述|
| :--- | :--- | :--- |
| 港口| int| 代表端口`otel_logs_source` 来源正在运行。默认值是`21892`。|
| 小路| 细绳| 表示发送无框HTTP请求的路径。您可以使用此选项来支持带有HTTP惯用路径的无框GRPC请求。路径应该以`/`，其长度至少应为1。`/opentelemetry.proto.collector.logs.v1.LogsService/Export` 如果配置了路径，则禁用GRPC和HTTP请求的端点。路径可以包含一个`${pipelineName}` 占位符，替换为管道名称。如果值为空，并且`unframed_requests` 是`true`，然后来源提供的路径是`/opentelemetry.proto.collector.logs.v1.LogsService/Export`。| 
| 请求超时| int| 代表毫秒中的请求超时持续时间。默认值是`10000`。|
| health_check_service| 布尔| 在下面启用GRPC健康检查服务`grpc.health.v1/Health/Check`。默认值是`false`。|
| proto_reflection_service| 布尔| 启用Protobuf服务的反射服务（请参阅[ProtoreRefertionservice](https://grpc.github.io/grpc-java/javadoc/io/grpc/protobuf/services/ProtoReflectionService.html) 和[GRPC反射](https://github.com/grpc/grpc-java/blob/master/documentation/server-reflection-tutorial.md)）。默认值是`false`。|
| unframed_requests| 布尔| 启用不使用GRPC线协议构成的请求。默认值是`false`。|
| thread_count| int| 要将线程保持在`ScheduledThreadPool`。默认值是`500`。|
| max_connection_count| int| 允许的最大开放连接数量。默认值是`500`。|

### SSL

您可以在`otel_logs_source` 带有以下选项的源。

| 选项| 类型| 描述|
| :--- | :--- | :--- |
| SSL| 布尔| 启用TLS/SSL。默认值是`true`。|
| sslkeycertchainfile| 细绳| 表示SSL证书链文件路径或Amazon简单存储服务（Amazon S3）路径。例如，请参阅Amazon S3路径`s3://<bucketName>/<path>`。如果需要`ssl` 被设定为`true`。|
| sslkeyfile| 细绳| 表示SSL密钥文件路径或Amazon S3路径。例如，请参阅Amazon S3路径`s3://<bucketName>/<path>`。如果需要`ssl` 被设定为`true`。|
| USEACMCERTFORSSL| 布尔| 使用AWS证书经理（ACM）的证书和私钥启用TLS/SSL。默认值是`false`。|
| acmcertificatearn| 细绳| 代表ACM证书亚马逊资源名称（ARN）。ACM证书优先于Amazon S3或本地文件系统证书。如果需要`useAcmCertForSSL` 被设定为`true`。|
| awsregion| 细绳| 代表ACM或Amazon S3使用的AWS区域。如果需要`useAcmCertForSSL` 被设定为`true` 或者`sslKeyCertChainFile` 或者`sslKeyFile` 是亚马逊S3路径。|

## 用法

首先，创建一个`pipeline.yaml` 文件并添加`otel_logs_source` 作为来源：

```
source:
    - otel_logs_source:
```

## 指标

您可以将以下指标与`otel_logs_source` 来源。

| 选项| 类型| 描述|
| :--- | :--- | :--- | 
| `requestTimeouts` | 柜台| 衡量超时的请求总数。| 
| `requestsReceived` | 柜台| 衡量该请求的总数`otel_logs_source` 来源。|
| `badRequests` | 柜台| 衡量无法解析的请求总数。|
| `requestsTooLarge` | 柜台| 衡量超过最大允许大小的请求总数。表明正在写入缓冲区的数据的大小超出了缓冲区的最大容量。|
| `internalServerError` | 柜台| 衡量由于错误而导致错误的请求总数`requestTimeouts` 或者`requestsTooLarge`。|
| `successRequests` | 柜台| 测量成功写入缓冲区的请求总数。|
| `payloadSize` | 分销摘要| 测量所有传入有效载荷大小的分布。|
| `requestProcessDuration` | 计时器| 衡量请求处理的持续时间。

