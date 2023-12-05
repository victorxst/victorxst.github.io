---
layout: default
title: otel_trace_group
parent: 处理器
grand_parent: 管道
nav_order: 73
---

# otel_trace_group

这`otel_trace_group` 处理器完成缺失跟踪-团体-集合中的相关领域[跨度](https://github.com/opensearch-project/data-prepper/blob/834f28fdf1df6d42a6666e91e6407474b88e7ec6/data-prepper-api/src/main/java/org/opensearch/dataprepper/model/trace/Span.java) 通过查找OpenSearch后端记录。这`otel_trace_group` 处理器确定缺少的跟踪组信息`spanId` 通过查找根源的相关字段`span` 存储在OpenSearch中。

## OpenSearch

当您使用用户名和密码连接到OpenSearch集群时，请使用以下示例`pipeline.yaml` 文件以配置`otel_trace_group` 处理器：

``` YAML
pipeline:
  ...
  processor:
    - otel_trace_group:
        hosts: ["https://localhost:9200"]
        cert: path/to/cert
        username: YOUR_USERNAME_HERE
        password: YOUR_PASSWORD_HERE
```

看[OpenSearch安全]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sinks/opensearch/#opensearch-cluster-security) 对于更详细的说明，需要使用哪些OpenSearch凭据和权限以及如何为Otel Trace Group处理器配置这些凭据。

### Amazon OpenSearch服务

当您使用时[Amazon OpenSearch服务]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sinks/opensearch/#amazon-opensearch-service-domain-security)，使用以下示例`pipeline.yaml` 文件以配置`otel_trace_group` 处理器：

``` YAML
pipeline:
  ...
  processor:
    - otel_trace_group:
        hosts: ["https://your-amazon-opensearch-service-endpoint"]
        aws_sigv4: true
        cert: path/to/cert
        insecure: false
```

## 配置

您可以配置`otel_trace_group` 带有以下选项的处理器。

| 姓名| 描述| 默认值|
| -----| ----| -----------|
| `hosts`| OpenSearch节点的IP地址列表。必需的。| 没有默认值。| 
| `cert` | PEM编码的证书授权（CA）证书。接受.pem或.crt。这使客户能够信任已签署OpenSearch使用的证书的CA。| `null` |
| `aws_sigv4` | 一个布尔标志，用来签署具有AWS凭据的HTTP请求。仅适用于Amazon OpenSearch服务。看[OpenSearch安全](https://github.com/opensearch-project/data-prepper/blob/129524227779ee35a327c27c3098d550d7256df1/data-prepper-plugins/opensearch/security.md) 有关详细信息。| `false`。|
| `aws_region` | 代表Amazon OpenSearch服务域的AWS区域的字符串，例如`us-west-2`。仅适用于Amazon OpenSearch服务。| `us-east-1` |
| `aws_sts_role_arn`| AWS的身份和访问管理（IAM）角色，该插件假定将请求签署到Amazon OpenSearch服务。如果未提供，则插件使用[默认凭据](https://sdk.amazonaws.com/java/api/latest/software/amazon/awssdk/auth/credentials/DefaultCredentialsProvider.html)。| `null` |
| `aws_sts_header_overrides` | 标头的地图覆盖了IAM角色为接收器插件所假定的。| `null` |
| `insecure` | 布尔标志用于关闭SSL证书验证。如果设置为`true`，关闭CA证书验证并发送了不安全的HTTP请求。| `false` |
| `username` | 一个包含用户名的字符串，用于[内部用户](https://opensearch.org/docs/latest/security/access-control/users-roles/) `YAML` OpenSearch集群的配置文件。| `null` |
| `password` | 一个包含密码并在[内部用户](https://opensearch.org/docs/latest/security/access-control/users-roles/) `YAML` OpenSearch集群的配置文件。| `null` |

## 配置选项示例

您可以在`aws_sts_header_overrides` 选项。请参阅以下示例：

```
aws_sts_header_overrides:
  x-my-custom-header-1: my-custom-value-1
  x-my-custom-header-2: my-custom-value-2
```

## 指标

下表描述了特定于`otel_trace_group` 处理器。

| 公制名称| 类型| 描述|
| ------------- | ---- | ----------- |
| `recordsInMissingTraceGroup` | 柜台| 入口记录缺少跟踪组字段。|
| `recordsOutFixedTraceGroup` | 柜台| 带有成功完成的跟踪组字段的出口记录数量。|
| `recordsOutMissingTraceGroup` | 柜台| 出口记录缺少跟踪组字段的数量。

