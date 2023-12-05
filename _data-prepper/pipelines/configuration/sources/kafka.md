---
layout: default
title: kafka
parent: 来源
grand_parent: 管道
nav_order: 6
---

# 卡夫卡

您可以使用Apache Kafka源（`kafka`）在Data Prepper中读取一个或多个Kafka的记录[主题](https://kafka.apache.org/intro#intro_concepts_and_terms)。这些记录保存了您的数据预先管道可以吸收的事件。这`kafka` 来源使用kafka的[消费者API](https://kafka.apache.org/documentation/#consumerapi) 为了消耗KAFKA经纪人的消息，然后创建数据预先事件，以通过数据Prepper管道进行进一步处理。

## 用法

以下示例显示`kafka` 数据预先管道中的来源：

```json
kafka-pipeline:
  source:
    kafka:
      bootstrap_servers:
        - 127.0.0.1:9093
      topics:
        - name: Topic1
          group_id: groupID1
        - name: Topic2
          group_id: groupID1
```

## 配置

使用以下配置选项与`kafka` 来源。

选项| 必需的| 类型| 描述
：--- | ：--- | ：--- | ：---
`bootstrap_servers` | 是的，当不使用Amazon托管流媒体流媒体时，Apache Kafka（Amazon MSK）作为群集。| IP地址| 与Kafka群集的初始连接的主机或端口。您可以通过为每个经纪人使用IP地址或端口号来配置多个KAFKA经纪人。使用时[亚马逊MSK](https://aws.amazon.com/msk/) 作为您的KAFKA群集，使用配置中提供的MSK Amazon资源名称（ARN）从MSK获得了Bootstrap服务器信息。
`topics` | 是的| Json Array| 数据预先数据的Kafka主题`kafka` 源用于读取消息。您最多可以配置10个主题。有关有关的更多信息`topics` 配置选项，请参阅[主题](#topics)。
`schema` | 不| JSON对象| 模式注册表配置。有关更多信息，请参阅[模式](#schema)。
`authentication` | 不| JSON对象| 为管道和Kafka设置身份验证选项。有关更多信息，请参阅[验证](#authentication)。
`encryption` | 不| JSON对象| 加密配置。有关更多信息，请参阅[加密](#encryption)。
`aws` | 不| JSON对象| AWS配置。有关更多信息，请参阅[AWS](#aws)。
`acknowledgments` | 不| 布尔| 如果`true`，启用`kafka` 接收的来源[结尾-到-结束致谢]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/pipelines/#end-to-end-acknowledgments) 当通过OpenSearch水槽收到事件时。默认为`false`。
`client_dns_lookup` | 是的，当使用DNS别名时。| 细绳| 设定卡夫卡的`client.dns.lookup` 选项。默认为`default`。

### 主题

使用以下选项`topics` 大批。

选项| 必需的| 类型| 描述
：--- | ：--- | ：--- | ：---
`name` | 是的| 细绳| 每个Kafka主题的名称。
`group_id` | 是的| 细绳| 设定卡夫卡的`group.id` 选项。
`workers` | 不| 整数| 与每个主题相关的多线程消费者的数量。默认为`2`。最大值是`200`。
`serde_format` | 不| 细绳| 指示主题中消息的序列化和避免格式。默认为`plaintext`。
`auto_commit` | 不| 布尔| 什么时候`false`，消费者的偏见不会定期承诺在背景中对Kafka进行。默认为`false`。
`commit_interval` | 不| 整数| 什么时候`auto_commit` 被设定为`true`，设置多久，几秒钟内，消费者偏移是自动的-通过卡夫卡致力于卡夫卡`auto.commit.interval.ms` 选项。默认为`5s`。
`session_timeout` | 不| 整数| 使用KAFKA的组管理功能时，源检测到客户端失败的时间，可用于平衡数据流。默认为`45s`。
`auto_offset_reset` | 不| 细绳| 自动将偏置重置为较早或通过kafka的偏移量`auto.offset.reset` 选项。默认为`latest`。
`thread_waiting_time` | 不| 整数| 线程等待前一个线程完成其任务并发出下一个线程的时间。KAFKA消费者API投票超时值设置为此设置的一半。默认为`5s`。
`max_partition_fetch_bytes` | 不| 整数| 设置Megabytes的最大限制，以通过Kafka的每个分区从每个分区返回最大数据`max.partition.fetch.bytes` 环境。默认为`1mb`。
`heart_beat_interval` | 不| 整数| 通过Kafka的Kafka的小组管理设施，对消费者协调员进行心跳之间的预期时间`heartbeat.interval.ms` 环境。默认为`5s`。
`fetch_max_wait` | 不| 整数| 当没有足够数据满足的数据时，服务器阻止提取请求的最长时间`fetch_min_bytes` 通过卡夫卡的要求`fetch.max.wait.ms` 环境。默认为`500ms`。
`fetch_max_bytes` | 不| 整数| 经纪人通过Kafka接受的最大记录大小`fetch.max.bytes` 环境。默认为`50mb`。
`fetch_min_bytes` | 不| 整数| 服务器通过Kafka的提取请求期间返回的最小数据量`retry.backoff.ms` 环境。默认为`1b`。
`retry_backoff` | 不| 整数| 在尝试重试给定主题分区的失败请求之前等待的时间。默认为`10s`。
`max_poll_interval` | 不| 整数| A之间的最大延迟`poll()` 通过Kafka的小组管理`max.poll.interval.ms` 选项。默认为`300s`。
`consumer_max_poll_records` | 不| 整数| 单个返回的最大记录数`poll()` 打电话`max.poll.records` 环境。默认为`500`。
`key_mode` | 不| 细绳| 指示如何处理KAFKA消息的密钥字段。默认设置为`include_as_field`，其中包括关键`kafka_key` 事件。这`include_as_metadata` 设置包括事件的元数据中的密钥。这`discard` 设置丢弃密钥。

### 模式

需要以下选项`schema` 配置。

选项| 类型| 描述
：--- | ：--- | ：---
`type` | 细绳| 根据您的注册表设置模式类型，要么是AWS胶模式注册表`aws_glue`，或汇合模式注册表，`confluent`。使用时`aws_glue` 注册表，设置任何[AWS](#aws) 配置选项。

仅在使用一个时才需要以下配置选项`confluent` 注册表。

选项| 类型| 描述
：--- | ：--- | ：---
`registry_url` | 细绳| 从一个`bytearray` 进入字符串。默认为`org.apache.kafka.common.serialization.StringDeserializer`。
`version` | 细绳| 从一个符合记录键`bytearray` 进入字符串。默认为`org.apache.kafka.common.serialization.StringDeserializer`。
`schema_registry_api_key` | 细绳| 模式注册表API密钥。
`schema_registry_api_secret` | 细绳| 模式注册表API秘密。

### 验证

需要以下选项`authentication` 目的。

选项| 类型| 描述
：--- | ：--- | ：---
`sasl` | JSON对象| 简单的身份验证和安全层（SASL）身份验证配置。

### sasl

配置SASL身份验证时，请使用以下选项之一。


选项| 类型| 描述
：--- | ：--- | ：---
`plaintext` | JSON对象| 这[纯文本](#sasl-plaintext) 身份验证配置。
`aws_msk_iam` | 细绳| Amazon MSK AWS身份和访问管理（IAM）配置。如果设置为`role`， 这`sts_role_arm` 设置在`aws` 使用配置。默认为`default`。



#### SASL明文

使用时需要以下选项[SASL明文](https://kafka.apache.org/10/javadoc/org/apache/kafka/common/security/auth/SecurityProtocol.html) 协议。

选项| 类型| 描述
：--- | ：--- | ：---
`username` | 细绳| 明文Auth的用户名。
`password` | 细绳| 明文Auth的密码。

#### 加密

设置SSL加密时，请使用以下选项。

选项| 必需的| 类型| 描述
：--- | ：--- | ：--- | ：---
`type` | 不| 细绳| 加密类型。使用`none` 禁用加密。默认为`ssl`。
`Insecure` | 不| 布尔| 布尔标志用于关闭SSL证书验证。如果设置为`true`，证书授权（CA）证书验证已关闭，并发送了不安全的HTTP请求。默认为`false`。


#### AWS

设置身份验证时，请使用以下选项`aws` 服务。

选项| 必需的| 类型| 描述
：--- | ：--- | ：--- | ：---
`region` | 不| 细绳| 用于凭证的AWS区域。默认为[标准SDK行为以确定该区域](https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/region-selection.html)。
`sts_role_arn` | 不| 细绳| AWS安全令牌服务（AWS STS）角色要担任亚马逊简单队列服务（Amazon SQS）和Amazon Simple Storage Service（Amazon S3）的请求。默认为`null`，将使用[凭证的标准SDK行为](https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/credentials.html)。
`msk` | 不| JSON对象| 这[MSK](#msk) 配置设置。

#### MSK

使用以下选项`msk` 目的。

选项| 必需的| 类型| 描述
：--- | ：--- | ：--- | ：---
`arn` | 是的| 细绳| 这[MSK ARN](https://docs.aws.amazon.com/msk/1.0/apireference/configurations-arn.html) 使用。
`broker_connection_type` 不| 细绳| 要么与MSK经纪人一起使用的连接器类型`public`，，，，`single_vpc`， 或者`multip_vpc`。默认为`single_vpc`。


