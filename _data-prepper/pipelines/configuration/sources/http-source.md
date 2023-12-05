---
layout: default
title: http_source
parent: 来源
grand_parent: 管道
nav_order: 5
---

# http_source

`http_source` 是支持HTTP的源插件。现在，`http_source` 仅支持JSON UTF-8用于传入请求的编解码器，例如`[{"key1": "value1"}, {"key2": "value2"}]`。下表描述了您可以使用的选项来配置`http_source` 来源。

选项| 必需的| 类型| 描述
：--- | ：--- | ：--- | ：---
港口| 不| 整数| 源正在运行的端口。默认值是`2021`。有效的选项是`0` 和`65535`。
health_check_service| 不| 布尔| 启用健康检查服务`/health` 定义端口上的端点。默认值是`false`。
未经授权的_health_check| 不| 布尔| 确定在健康检查端点上是否需要身份验证。如果未定义身份验证，则数据预先忽略此选项。默认值是`false`。
请求超时| 不| 整数| 请求超时，以毫秒为单位。默认值是`10000`。
thread_count| 不| 整数| 将要保留在计划中的线程的线程数。默认值是`200`。
max_connection_count| 不| 整数| 允许的最大开放连接数。默认值是`500`。
max_pending_requests| 不| 整数| 允许的任务数量最大`ScheduledThreadPool` 工作队列。默认值是`1024`。
验证| 不| 目的| 身份验证配置。默认情况下，这将为管道创建一个未经验证的服务器。这使用HTTPS的可插入身份验证。使用基本身份验证定义`http_basic` 带有插件`username` 和`password`。提供客户身份验证，使用或创建实施的插件[Armeriahttpauthentication -Provider](https://github.com/opensearch-project/data-prepper/blob/1.2.0/data-prepper-plugins/armeria-common/src/main/java/com/amazon/dataprepper/armeria/authentication/ArmeriaHttpAuthenticationProvider.java)。
SSL| 不| 布尔| 启用TLS/SSL。默认值为false。
ssl_certificate_file| 有条件的| 细绳| SSL证书链文件路径或亚马逊简单存储服务（Amazon S3）路径。亚马逊S3路径示例`s3://<bucketName>/<path>`。如果需要`ssl` 设置为true和`use_acm_certificate_for_ssl` 设置为false。
ssl_key_file| 有条件的| 细绳| SSL密钥文件路径或Amazon S3路径。亚马逊S3路径示例`s3://<bucketName>/<path>`。如果需要`ssl` 设置为true和`use_acm_certificate_for_ssl` 设置为false。
use_acm_certificate_for_ssl| 不| 布尔| 使用AWS证书经理（ACM）的证书和私钥启用TLS/SSL。默认值为false。
acm_certificate_arn| 有条件的| 细绳| ACM证书亚马逊资源名称（ARN）。ACM证书优先于Amazon S3或本地文件系统证书。如果需要`use_acm_certificate_for_ssl` 设置为true。
acm_private_key_password| 不| 细绳| ACM专用密钥密码可以解密私钥。如果未提供，则数据PEPPER会生成随机密码。
acm_certificate_timeout_millis| 不| 整数| 超时，ACM以毫秒为单位获得证书。默认值为120000。
aws_region| 有条件的| 细绳| ACM或Amazon S3使用的AWS区域。如果需要`use_acm_certificate_for_ssl` 设置为true或`ssl_certificate_file` 和`ssl_key_file` 是亚马逊S3路径。

<！--- ## 配置

内容将添加到本节中。--->

## 指标

这`http_source` 来源包括以下指标。

### 柜台

- `requestsReceived`：衡量该请求总数`/log/ingest` 端点。
- `requestsRejected`：测量HTTP源插件拒绝的请求总数（429响应状态代码）。
- `successRequests`：测量成功处理的请求总数（200个响应状态代码）由HTTP源插件。
- `badRequests`：用HTTP源插件（400响应状态代码）处理的无效内容类型或格式来衡量请求总数。
- `requestTimeouts`：测量HTTP源服务器（415响应状态代码）中超时的请求总数。
- `requestsTooLarge`：测量事件大小大于缓冲区容量（413响应状态代码）的请求总数。
- `internalServerError`：用自定义类型（500响应状态代码）来衡量HTTP源处理的请求总数。

### 计时器

- `requestProcessDuration`：测量HTTP源插件在几秒钟内处理的请求的延迟。

### 分销摘要

- `payloadSize`：测量在字节中的传入请求有效载荷大小。

