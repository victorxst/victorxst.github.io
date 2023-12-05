---
layout: default
title: opensearch
parent: 来源
grand_parent: 管道
nav_order: 30
---

# OpenSearch

这`opensearch` 源插件用于从OpenSearch群集，Legacy Elasticsearch集群，Amazon OpenSearch Service Service域或Amazon OpenSearch无服务器集合中读取索引。

该插件支持OpenSearch 2.X和Elasticsearch 7.x。

## 用法

使用`opensearch` 源具有最低要求的设置，将以下配置添加到您的`pipeline.yaml` 文件：

```yaml
opensearch-source-pipeline:
 source:
  opensearch:
    hosts: [ "https://localhost:9200" ]
    username: "username"
    password: "password"
 ...
```

使用`opensearch` 所有配置设置，包括`indices`，，，，`scheduling`，，，，`search_options`， 和`connection`，将以下示例添加到您的`pipeline.yaml` 文件：

```yaml
opensearch-source-pipeline:
  source:
    opensearch:
      hosts: [ "https://localhost:9200" ]
      username: "username"
      password: "password"
      indices:
        include:
          - index_name_regex: "test-index-.*"
        exclude:
          - index_name_regex: "\..*"
      scheduling:
        interval: "PT1H"
        index_read_count: 2
        start_time: "2023-06-02T22:01:30.00Z"
      search_options:
        search_context_type: "none"
        batch_size: 1000
      connection:
        insecure: false
        cert: "/path/to/cert.crt"
  ...
```

## Amazon OpenSearch服务

这`opensearch` 可以通过通过一个`sts_role_arn` 随着访问域的访问，如以下示例所示：

```yaml
opensearch-source-pipeline:
  source:
    opensearch:
      hosts: [ "https://search-my-domain-soopywaovobopgs8ywurr3utsu.us-east-1.es.amazonaws.com" ]
      aws:
        region: "us-east-1"
        sts_role_arn: "arn:aws:iam::123456789012:role/my-domain-role"
  ...
```

## 使用元数据

当。。。的时候`opensource` 源构造数据从集群中的文档中的PEPPPER事件，文档索引存储在EventMetadata中`opensearch-index` 键，并且document_id存储在`EventMetadata` 与`opensearch-document_id` 作为钥匙。这允许根据索引或`document_id`。以下示例管道配置将事件发送到`opensearch` 下沉并使用相同的索引和`document_id` 从源群集中，如目标群集中的那样：


```yaml
opensearch-migration-pipeline:
  source:
    opensearch:
      hosts: [ "https://source-cluster:9200" ]
      username: "username"
      password: "password"
  sink:
    - opensearch:
        hosts: [ "https://sink-cluster:9200" ]
        username: "username"
        password: "password"
        document_id: "${getMetadata(\"opensearch-document_id\")}"
        index: "${getMetadata(\"opensearch-index\"}"
```

## 配置选项


下表描述了您可以配置的选项`opensearch` 来源。

选项| 必需的| 类型| 描述
：--- | :--- |:--------| ：---
`hosts` | 是的| 列表| openSearch主机的列表，例如`["https://localhost:9200", "https://remote-cluster:9200"]`。
`username` | 不| 细绳| HTTP基本身份验证的用户名。由于Data Prepper 2.5，因此可以在运行时刷新此设置[AWS秘密参考]({{site.url}}{{site.baseurl}}/data-prepper/managing-data-prepper/configuring-data-prepper/#reference-secrets) 被申请;被应用。
`password` | 不| 细绳| HTTP基本身份验证的密码。由于Data Prepper 2.5，因此可以在运行时刷新此设置[AWS秘密参考]({{site.url}}{{site.baseurl}}/data-prepper/managing-data-prepper/configuring-data-prepper/#reference-secrets) 被申请;被应用。
`disable_authentication` | 不| 布尔| 身份验证是否被禁用。默认为`false`。
`aws` | 不| 目的| AWS配置。有关更多信息，请参阅[AWS](#aws)。
`acknowledgments` | 不| 布尔| 什么时候`true`，启用`opensearch` 接收的来源[结尾-到-结束致谢]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/pipelines/#end-to-end-acknowledgments) 当通过OpenSearch水槽收到事件时。默认为`false`。
`connection` | 不| 目的| 连接配置。有关更多信息，请参阅[联系](#connection)。
`indices` | 不| 目的| 处理哪些索引的配置。默认为所有索引，包括系统索引。有关更多信息，请参阅[索引](#indices)。
`scheduling` | 不| 目的| 调度配置。有关更多信息，请参阅[调度](#scheduling)。
`search_options` | 不| 目的| 来源执行的搜索选项列表。有关更多信息，请参阅[搜索选项](#search_options)。

### 调度

这`scheduling` 配置允许用户根据源来配置如何在源中重新处理索引`index_read_count` 和叙述时间`interval`。

例如，设置`index_read_count` 到`3` 与`interval` 的`1h` 将导致所有索引被重新处理3次，相距1小时。默认情况下，索引仅处理一次。

使用以下选项`scheduling` 配置。

选项| 必需的| 类型| 描述
：--- | :--- |:----------------| ：---
`index_read_count` | 不| 整数| 每个索引将处理的次数。默认为`1`。
`interval` | 不| 细绳| 确定重新处理之间时间的时间间隔。支持ISO 8601符号字符串，例如"PT20.345S" 或者"PT15M"，以及几秒钟的简单符号字符串（"60s"）和毫秒（"1500ms"）。默认为`8h`。
`start_time` | 不| 细绳| 应开始处理的时间。源直到这段时间才开始处理。字符串必须采用ISO 8601格式，例如`2007-12-03T10:15:30.00Z`。默认选项立即开始处理。


### 指数

以下选项有助于`opensearch` 源确定使用REGEX模式从源群集处理哪些索引。仅当索引匹配其中一个时，才会处理索引`index_name_regex` 下面的模式`include` 设置，不匹配任何
下面的模式`exclude` 环境。

选项| 必需的| 类型| 描述
：--- | :--- |:-----------------| ：---
`include` | 不| 对象数组| 指定将处理索引的索引配置模式的列表。
`exclude` | 不| 对象数组| 指定索引的索引配置模式的列表将不处理索引。例如，您可以指定`index_name_regex` 模式`\..*` 排除系统索引。


使用以下设置`include` 和`exclude` 指示索引的正则方式的选项。

选项| 必需的| 类型| 描述
:--- |:----|：-----------------| ：---
`index_name_regex` | 是的| REGEX字符串| 将索引与索引相匹配的正则模式。

### search_options

使用以下设置`search_options` 配置。

选项| 必需的| 类型| 描述
:--- |:---------|：--------| ：---
`batch_size` | 不| 整数| 从OpenSearch进行拨号时要读取的文档数量。默认为`1000`。
`search_context_type` | 不| 枚举| 用于在索引上使用的搜索/分页类型的替代。可[时间点]({{site.url}}{{site.baseurl}}/search-plugins/searching-data/paginate/#point-in-time-with-search_after)），[滚动]({{site.url}}{{site.baseurl}}/search-plugins/searching-data/paginate/#scroll-search)， 或者`none`。这`none` 选项将使用[search_after]({{site.url}}{{site.baseurl}}/search-plugins/searching-data/paginate/#the-search_after-parameter) 范围。有关更多信息，请参阅[默认搜索行为](#default-search-behavior)。

### 默认搜索行为

默认情况下，`opensearch` 来源将查找群集版本和分发以确定
哪个`search_context_type` 使用。用于支持的版本和发行版[时间点](https://opensearch.org/docs/latest/search-plugins/searching-data/paginate/#point-in-time-with-search_after)，，，，`point_in_time` 将会被使用。
如果`point_in_time` 不受集群的支持，然后[滚动](https://opensearch.org/docs/latest/search-plugins/searching-data/paginate/#scroll-search) 将会被使用。对于Amazon OpenSearch无服务器集合，[search_after](https://opensearch.org/docs/latest/search-plugins/searching-data/paginate/#the-search_after-parameter) 将使用，因为都不`point_in_time` 也不`scroll` 由收藏支持。

### 联系

使用以下设置`connection` 配置。

选项| 必需的| 类型| 描述
：--- | :--- |:--------| ：---
`cert` | 不| 细绳| 例如，安全证书的路径`"config/root-ca.pem"`，当群集使用OpenSearch安全插件时。
`insecure` | 不| 布尔| 是否要验证SSL证书。如果设置为`true`，证书机构（CA）证书验证被禁用，并发送了不安全的HTTP请求。默认为`false`。


### AWS

设置身份验证时，请使用以下选项`aws` 服务。

选项| 必需的| 类型| 描述
：--- | :--- |:--------| ：---
`region` | 不| 细绳| 用于凭证的AWS区域。默认为[标准SDK行为以确定该区域](https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/region-selection.html)。
`sts_role_arn` | 不| 细绳| AWS安全令牌服务（AWS STS）角色要为Amazon OpenSearch Service和Amazon OpenSearch无服务器的请求担任。默认为`null`，将使用[凭证的标准SDK行为](https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/credentials.html)。
`serverless` | 不| 布尔| 应该设置为`true` 从Amazon OpenSearch无服务器集合进行处理时。默认为`false`。


## OpenSearch集群安全性

以使用`opensearch` 源插件，您必须在管道配置中指定用户名和密码。以下示例`pipeline.yaml` 文件演示了如何指定默认管理安全凭据：

```yaml
source:
  opensearch:
    username: "admin"
    password: "admin"
  ...
```

### Amazon OpenSearch服务域安全性

这`opensearch` 源插件可以从[Amazon OpenSearch服务](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/what-is.html) 域，使用AWS身份和访问管理（IAM）进行安全性。该插件使用默认的Amazon OpenSearch服务凭证链。跑步`aws configure` 使用[AWS命令线接口（AWS CLI）](https://aws.amazon.com/cli/) 设置您的凭据。

确保您配置的凭据具有所需的IAM权限。以下域访问策略显示了所需的最低权限：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<AccountId>:user/data-prepper-user"
      },
      "Action": "es:ESHttpGet",
      "Resource": [
        "arn:aws:es:us-east-1:<AccountId>:domain/<domain-name>/",
        "arn:aws:es:us-east-1:<AccountId>:domain/<domain-name>/_cat/indices",
        "arn:aws:es:us-east-1:<AccountId>:domain/<domain-name>/_search",
        "arn:aws:es:us-east-1:<AccountId>:domain/<domain-name>/_search/scroll",
        "arn:aws:es:us-east-1:<AccountId>:domain/<domain-name>/*/_search"
      ]
    },
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<AccountId>:user/data-prepper-user"
      },
      "Action": "es:ESHttpPost",
      "Resource": [
        "arn:aws:es:us-east-1:<AccountId>:domain/<domain-name>/*/_search/point_in_time",
        "arn:aws:es:us-east-1:<AccountId>:domain/<domain-name>/*/_search/scroll"
      ]
    },
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<AccountId>:user/data-prepper-user"
      },
      "Action": "es:ESHttpDelete",
      "Resource": [
        "arn:aws:es:us-east-1:<AccountId>:domain/<domain-name>/_search/point_in_time",
        "arn:aws:es:us-east-1:<AccountId>:domain/<domain-name>/_search/scroll"
      ]
    }
  ]
}
```

有关如何配置域访问策略的说明，请参见[资源-基于政策
]（https://docs.aws.amazon.com/opensearch-服务/最新/developerguide/ac.html#交流-类型-Amazon OpenSearch服务文档中的资源）。

### OpenSearch无服务器集合安全

这`opensearch` 源插件可以从一个[Amazon OpenSearch无服务器](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/serverless.html) 收藏。

您无法从使用虚拟私有云（VPC）访问的集合中读取。必须从公共网络访问该集合。
{： 。警告}

#### 创建管道角色

要使用OpenSearch无服务器集合安全性，请创建管道将假定的IAM角色，以便从集合中阅读。该角色必须具有以下最低权限：

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "aoss:APIAccessAll"
            ],
            "Resource": "arn:aws:aoss:*:<AccountId>:collection/*"
        }
    ]
}
```

#### 创建一个集合

接下来，创建一个具有以下设置的集合：

- 民众[网络访问](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/serverless-network.html) 到OpenSearch Endpoint和OpenSearch仪表板。
- 下列[数据访问策略](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/serverless-data-access.html)，它可以将所需的权限授予管道角色，如以下配置所示：

  ```JSON
  [
   {
      "Rules"：[[
         {
            "Resource"：[[
               "index/collection-name/*"
            ]，，
            "Permission"：[[
               "aoss:ReadDocument"，，，，
               "aoss:DescribeIndex"
            ]，，
            "ResourceType"："index"
         }
      ]，，
      "Principal"：[[
         "arn:aws:iam::<AccountId>:role/PipelineRole"
      ]，，
      "Description"："Pipeline role access"
   }
  这是给出的
  ```

Make sure to replace the Amazon Resource Name (ARN) in the `主要的` 元素具有您在上一步中创建的管道角色的ARN。
{： 。提示}

有关如何创建收藏的说明，请参阅[创建收藏](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/serverless-manage.html#serverless-create) 在Amazon OpenSearch服务文档中。

#### 创建管道

在你内`pipeline.yaml` 文件，指定OpenSearch serverless Collection端点为`hosts` 选项。此外，您必须设置`serverless` 选项`true`。指定管道角色`sts_role_arn` 选项，如下所示：

```yaml
opensearch-source-pipeline:
  source:
    opensearch:
      hosts: [ "https://<serverless-public-collection-endpoint>" ]
      aws:
        serverless: true
        sts_role_arn: "arn:aws:iam::<AccountId>:role/PipelineRole"
        region: "us-east-1"
  processor:
    - date:
        from_time_received: true
        destination: "@timestamp"
  sink:
    - stdout:
```

