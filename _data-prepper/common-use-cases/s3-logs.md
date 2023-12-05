---
layout: default
title: S3日志
parent: 常见用例
nav_order: 20
---

# S3日志

Data Prepper允许您从中加载日志[亚马逊简单存储服务](https://aws.amazon.com/s3/) （Amazon S3），包括传统日志，JSON文档和CSV日志。


## 建筑学

数据Prepper可以使用一个[亚马逊简单队列服务（SQS）](https://aws.amazon.com/sqs/) （Amazon SQS）队列和[亚马逊S3事件通知](https://docs.aws.amazon.com/AmazonS3/latest/userguide/NotificationHowTo.html)。

Data Prepper对S3事件通知的Amazon SQS队列进行了调查。当Data Prepper收到一条通知时，就会创建S3对象时，数据预先读取和解析S3对象。

下图显示了所涉及的组件的整体体系结构。

<img src="{{site.url}}{{site.baseurl}}/images/data-prepper/s3-source/s3-architecture.jpg" alt="S3 source architecture">{: .img-fluid}

数据流量如下。

1. 系统将登录到S3存储桶中。
2. S3在SQS队列中创建S3事件通知。
3. Data Prepper对Amazon SQS进行了调查，以获取消息，然后收到消息。
4. Data Prepper从S3对象下载内容。
5. Data Prepper将文档发送给S3对象中内容的文档。


## 管道概述

数据Prepper支持使用S3读取数据[`s3` 来源]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sources/s3/)。

下图显示了S3的PEPEPPER管道读数的概念概述。

<img src="{{site.url}}{{site.baseurl}}/images/data-prepper/s3-source/s3-pipeline.jpg" alt="S3 source architecture">{: .img-fluid}

## 先决条件

在数据Prepper可以读取S3的日志数据之前，您需要以下先决条件：

- S3桶。
- 将日志写入S3的日志生产商。确切的日志生产者将根据您的特定用例有所不同，但可以包括将日志写入S3或Amazon CloudWatch之类的服务。


## 入门

使用以下步骤开始使用Data Prepper从S3加载日志。

1. 创建一个[SQS标准队列](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/step-create-queue.html) 对于您的S3事件通知。
2. 配置[水桶通知](https://docs.aws.amazon.com/AmazonS3/latest/userguide/ways-to-add-notification-config-to-bucket.html) 对于SQS。使用`s3:ObjectCreated:*` 事件类型。
3. 授予[aws iam](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html) 访问SQS和S3的数据预先权限的权限。
4. （建议）创建一个[SQS死了-字母队列](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html) （DLQ）。
5. （建议）配置平方英尺-驱动策略将失败消息移至DLQ。

### 设置数据预先的权限

要查看S3日志，Data Prepper需要访问Amazon SQS和S3。
使用以下示例设置权限：

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "s3-access",
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::<YOUR-BUCKET>/*"
        },
        {
            "Sid": "sqs-access",
            "Effect": "Allow",
            "Action": [
                "sqs:DeleteMessage",
                "sqs:ReceiveMessage"
            ],
            "Resource": "arn:aws:sqs:<YOUR-REGION>:<123456789012>:<YOUR-SQS-QUEUE>"
        },
        {
            "Sid": "kms-access",
            "Effect": "Allow",
            "Action": "kms:Decrypt",
            "Resource": "arn:aws:kms:<YOUR-REGION>:<123456789012>:key/<YOUR-KMS-KEY>"
        }
    ]
}
```

如果您的S3对象或SQS队列不使用KMS，则可以删除`kms:Decrypt` 允许。

### SQS死了-字母队列

这是如何处理处理S3对象产生的错误的两个选项。

- 使用SQS死亡-字母队列（DLQ）跟踪故障。这是推荐的方法。
- 从SQS删除该消息。您必须手动找到S3对象并纠正错误。

下图显示了与DLQ一起使用SQS时的系统体系结构。

<img src="{{site.url}}{{site.baseurl}}/images/data-prepper/s3-source/s3-architecture-dlq.jpg" alt="S3 source architecture with dlq">{: .img-fluid}

使用SQS死亡-字母队列，执行以下步骤：

1. 创建一个新的SQS标准队列以充当您的DLQ。
2. 配置您的SQS的Redrive策略[使用您的DLQ](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-configure-dead-letter-queue.html)。考虑使用低值，例如2或3"Maximum Receives" 环境。
3. 配置数据预先`s3` 使用的来源`retain_messages` 为了`on_error`。这是默认行为。

## 管道设计

创建一个管道以从S3读取日志，从[`s3`]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sources/s3/) 源插件。使用以下示例进行指导。

```yaml
s3-log-pipeline:
   source:
     s3:
       notification_type: sqs
       compression: gzip
       codec:
         newline:
       sqs:
         # Change this value to your SQS Queue URL
         queue_url: "arn:aws:sqs:<YOUR-REGION>:<123456789012>:<YOUR-SQS-QUEUE>"
         visibility_timeout: "2m"
```

根据您的用例配置以下选项：

*`queue_url`：这是SQS队列URL，并且始终是您的管道所独有的。
*`codec`：编解码器确定如何解析传入数据。
*`visibility_timeout`：配置此值足够大，以便数据预先处理10个S3对象。但是，如果您使此值太大，则无法处理的消息至少在数据预先重试之前所需的时间与指定值一样长。

大多数用例的每个选项的默认值工作。对于S3源的所有可用选项，请参阅[`s3`]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sources/s3/)。

```yaml
s3-log-pipeline:
   source:
     s3:
       notification_type: sqs
       compression: gzip
       codec:
         newline:
       sqs:
         # Change this value to your SQS Queue URL
         queue_url: "arn:aws:sqs:<YOUR-REGION>:<123456789012>:<YOUR-SQS-QUEUE>"
         visibility_timeout: "2m"
       aws:
         # Specify the correct region
         region: "<YOUR-REGION>"
         # This shows using an STS role, but you can also use your system's default permissions.
         sts_role_arn: "arn:aws:iam::<123456789012>:role/<DATA-PREPPER-ROLE>"
   processor:
     # You can configure a grok pattern to enrich your documents in OpenSearch.
     #- grok:
     #    match:
     #      message: [ "%{COMMONAPACHELOG}" ]
   sink:
     - opensearch:
         hosts: [ "https://localhost:9200" ]
         # Change to your credentials
         username: "admin"
         password: "admin"
         index: s3_logs
```

## 多个数据预备管道

我们建议您每个数据Prepper管道都有一个平方英尺的队列。此外，您可以从同一SQS队列中的相同群集读取中具有多个节点，这不需要与Data Prepper进行其他配置。

如果您有多个管道，则必须为每个管道创建多个SQS队列，即使两个管道都使用相同的S3存储桶。

## 亚马逊SNS粉丝模式

为了满足S3产生的日志规模，一些用户需要多个SQS队列的日志。您可以使用[亚马逊简单通知服务](https://docs.aws.amazon.com/sns/latest/dg/welcome.html) （Amazon SNS）将事件通知从S3到SQS[粉丝模式](https://docs.aws.amazon.com/sns/latest/dg/sns-common-scenarios.html)。使用SNS，所有S3事件通知直接发送到单个SNS主题，您可以在其中订阅多个SQS队列。

为确保数据预先可以直接从SNS主题解析事件，请配置[原始消息传递](https://docs.aws.amazon.com/sns/latest/dg/sns-large-payload-raw-message-delivery.html) 在sns to sqs订阅上。设置此选项不会影响订阅该SNS主题的其他SQS队列。



