---
layout: default
title: 死信队列
parent: 管道
nav_order: 13
---

# 死信队列

数据预先管道支持死信队列（DLQ）用于卸载失败的事件并使其可访问以进行分析。

从数据Prepper 2.3开始，只有`s3` 来源支持DLQ。

## 配置DLQ作者

配置DLQ作者`s3` 来源，将以下内容添加到您的pipeline.yaml文件：

```yaml
  sink:
    opensearch:
      dlq:
        s3:
          bucket: "my-dlq-bucket"
          key_path_prefix: "dlq-files/"
          region: "us-west-2"
          sts_role_arn: "arn:aws:iam::123456789012:role/dlq-role"
```

结果DLQ文件作为DLQ对象的JSON数组输出。写在S3 DLQ的任何文件都包含以下名称模式：

```
dlq-v${version}-${pipelineName}-${pluginId}-${timestampIso8601}-${uniqueId}
```
以下名称模式替换以下信息：


- `version`：数据预先版本。
- `pipelineName`：pipeline.yaml中指示的管道名称。
- `pluginId`：与DLQ事件关联的插件的ID。

## 配置

DLQ支持以下配置选项。

选项| 必需的| 类型| 描述
：--- | ：--- | ：--- | ：---
桶| 是的| 细绳| DLQ输出失败记录的存储桶的名称。
key_path_prefix| 不| 细绳| 这`key_prefix` 在S3桶中使用。默认为`""`。支持时间值模式变量，例如`/%{yyyy}/%{MM}/%{dd}`，包括在[Java DateTimeFormatter](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html)。例如，使用`/%{yyyy}/%{MM}/%{dd}` 模式，您可以设置`key_prefix` 作为`/2023/01/24`。
地区| 不| 细绳| S3桶的AWS区域。默认为`us-east-1`。
sts_role_arn| 不| 细绳| DLQ为了写入AWS S3存储桶而扮演的STS角色。默认为`null`，将标准SDK行为用于凭据。要使用此选项，S3存储桶必须具有`S3:PutObject` 配置了权限。

与OpenSearch sink一起使用DLQ时，您可以配置[max_retries]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sinks/opensearch/#configure-max_retries) 当接收器达到最大重试时，将失败数据发送到DLQ的选项。


## 指标

DLQ支持以下指标。

### 柜台

- `dlqS3RecordsSuccess`：测量发送给S3的成功记录的数量。
- `dlqS3RecordsFailed`：测量未能发送到S3的记录数量。
- `dlqS3RequestSuccess`：测量成功的S3请求的数量。
- `dlqS3RequestFailed`：测量失败的S3请求的数量。

### 分销摘要

- `dlqS3RequestSizeBytes`：测量S3请求的有效载荷大小在字节中的分布。

### 计时器

- `dlqS3RequestLatency`：测量发送每个S3请求时的延迟，包括重试。

## DLQ对象

DLQ支持以下DLQ对象：

*`pluginId`：源自发送到DLQ的事件的插件的ID。
*`pluginName`：插件的名称。
*`failedData` ：一个包含失败对象及其选项的对象。此对象是每个插件所独有的。
*`pipelineName`：事件失败的数据PEPPER管道的名称。
*`timestamp`：失败的时间戳`ISO8601` 格式。


