---
layout: default
title: otel_metrics_source
parent: 来源
grand_parent: 管道
nav_order: 10
---

# otel_metrics_source

`otel_metrics_source` 是收集度量数据的OpenTelemetry收集器来源。下表描述了您可以使用的选项来配置`otel_metrics_source` 来源。

选项| 必需的| 类型| 描述
：--- | ：--- | ：--- | ：---
港口| 不| 整数| OpenTelemtry指标源运行的端口。默认值是`21891`。
请求超时| 不| 整数| 请求超时，以毫秒为单位。默认值是`10000`。
health_check_service| 不| 布尔| 在下面启用GRPC健康检查服务`grpc.health.v1/Health/Check`。默认值是`false`。
proto_reflection_service| 不| 布尔| 启用Protobuf服务的反射服务（请参阅[GRPC反射](https://github.com/grpc/grpc/blob/master/doc/server-reflection.md) 和[GRPC服务器反射教程](https://github.com/grpc/grpc-java/blob/master/documentation/server-reflection-tutorial.md) 文档）。默认值是`false`。
unframed_requests| 不| 布尔| 启用不使用GRPC线协议构成的请求。
thread_count| 不| 整数| 要将线程保持在`ScheduledThreadPool`。默认值是`200`。
max_connection_count| 不| 整数| 允许的最大开放连接数。默认值是`500`。
SSL| 不| 布尔| 在TLS/SSL上启用与OpenTelemetry源端口的连接。默认值是`true`。
sslkeycertchainfile| 有条件的| 细绳| 文件-系统路径或亚马逊简单存储服务（Amazon S3）安全证书的路径（例如，`"config/demo-data-prepper.crt"` 或者`"s3://my-secrets-bucket/demo-data-prepper.crt"`）。如果需要`ssl` 被设定为`true`。
sslkeyfile| 有条件的| 细绳| 文件-系统路径或Amazon S3通往安全密钥的路径（例如，`"config/demo-data-prepper.key"` 或者`"s3://my-secrets-bucket/demo-data-prepper.key"`）。如果需要`ssl` 被设定为`true`。
USEACMCERTFORSSL| 不| 布尔| 是否使用AWS证书经理（ACM）的证书和私钥启用TLS/SSL。默认值是`false`。
acmcertificatearn| 有条件的| 细绳| 代表ACM证书ARN。ACM证书优先考虑S3或本地文件系统证书。如果需要`useAcmCertForSSL` 被设定为`true`。
awsregion| 有条件的| 细绳| 代表ACM或Amazon S3使用的AWS区域。如果需要`useAcmCertForSSL` 被设定为`true` 或者`sslKeyCertChainFile` 和`sslKeyFile` 是亚马逊S3路径。
验证| 不| 目的| 身份验证配置。默认情况下，为管道创建了一个未经身份验证的服务器。这使用HTTPS的可插入身份验证。要使用基本身份验证，请定义`http_basic` 带有插件`username` 和`password`。提供客户身份验证，使用或创建实施的插件[grpCauthentication -provider](https://github.com/opensearch-project/data-prepper/blob/1.2.0/data-prepper-plugins/armeria-common/src/main/java/com/amazon/dataprepper/armeria/authentication/GrpcAuthenticationProvider.java)。

<！--- ## 配置

内容将添加到本节中。--->

## 指标

这`otel_metrics_source` 来源包括以下指标。

### 柜台

- `requestTimeouts`：测量超时的请求总数。
- `requestsReceived`：测量OpenTelemetry指标来源收到的请求总数。


