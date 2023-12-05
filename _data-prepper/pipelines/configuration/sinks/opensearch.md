---
layout: default
title: opensearch 
parent: 下沉
grand_parent: 管道
nav_order: 50
---

# OpenSearch

您可以使用`opensearch` 接收器插件将数据发送到OpenSearch Cluster，Legacy Elasticsearch群集或Amazon OpenSearch Service Service域。

该插件支持OpenSearch 1.0及以后的OpenSearch，以及Elasticsearch 7.3及更高版本。

## 用法

配置一个`opensearch` 下沉，指定`opensearch` 管道配置中的选项：

```yaml
pipeline:
  ...
  sink:
    opensearch:
      hosts: ["https://localhost:9200"]
      cert: path/to/cert
      username: YOUR_USERNAME
      password: YOUR_PASSWORD
      index_type: trace-analytics-raw
      dlq_file: /your/local/dlq-file
      max_retries: 20
      bulk_size: 4
```

配置一个[Amazon OpenSearch服务](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/what-is.html) 接收器，将域端点指定为`hosts` 选项：

```yaml
pipeline:
  ...
  sink:
    opensearch:
      hosts: ["https://your-amazon-opensearch-service-endpoint"]
      aws_sigv4: true
      cert: path/to/cert
      insecure: false
      index_type: trace-analytics-service-map
      bulk_size: 4
```

## 配置选项

下表描述了您可以配置的选项`opensearch` 下沉。

选项| 必需的| 类型| 描述
:--- | :--- | :--- | :---
主持人| 是的| 列表| openSearch主机列表要写入（例如，`["https://localhost:9200", "https://remote-cluster:9200"]`）。
证书| 不| 细绳| 通往安全证书的路径（例如，`"config/root-ca.pem"`）如果群集使用OpenSearch安全插件。
用户名| 不| 细绳| HTTP基本身份验证的用户名。
密码| 不| 细绳| HTTP基本身份验证的密码。
AWS_SIGV4| 不| 布尔| 默认值为false。是否使用AWS身份和访问管理（IAM）签名来连接到Amazon OpenSearch服务域。对于您的访问密钥，秘密密钥和可选会话令牌，数据Prepper使用默认凭证链（环境变量，Java系统属性，`~/.aws/credential`， ETC。）。
aws_region| 不| 细绳| AWS地区（例如，`"us-east-1"`）对于域，如果您连接到Amazon OpenSearch服务。
AWS_STS_ROLE_ARN| 不| 细绳| 插件用来签署发送给Amazon OpenSearch服务的请求的IAM角色。如果未提供此信息，则插件使用默认凭据。
[max_retries](#configure-max_retries) | 不| 整数| OpenSearch水槽的最大次数应尝试将数据推向OpenSearch服务器，然后再将其视为故障。默认为`Integer.MAX_VALUE`。如果未提供，该水槽将尝试将数据无限期地推向OpenSearch Server，因为默认值很高，并且指数退回将增加重试之前的等待时间。
socket_timeout| 不| 整数| 超时，以毫秒为单位，等待数据返回（或两个连续数据包之间的最大不活动时间）。零超时值被解释为无限超时。如果此超时值为负或未设置，那么基础的Apache HTTPClient将依靠操作系统设置来管理套接字超时。
connect_timeout| 不| 整数| 从连接管理器请求连接时使用的毫秒中的超时。零超时值被解释为无限超时。如果此超时值为负或未设置，则基础Apache HTTPCLEINT将依靠操作系统设置来管理连接超时。
不安全| 不| 布尔| 是否要验证SSL证书。如果设置为true，则禁用证书授权（CA）证书验证，而不是发送不安全的HTTP请求。默认值是`false`。
代理人| 不| 细绳| 一个地址[向前http代理服务器](https://en.wikipedia.org/wiki/Proxy_server)。格式为"&lt;host name or IP&gt;:&lt;port&gt;"。例子："example.com:8100"，"http://example.com:8100"，"112.112.112.112:8100"。端口号无法省略。
指数| 有条件的| 细绳| 导出索引的名称。仅在`index_type` 是`custom`。
index_type| 不| 细绳| 该索引类型告诉接收器插件正在处理哪种类型的数据。有效值：`custom`，`trace-analytics-raw`，`trace-analytics-service-map`，`management-disabled`。默认值是`custom`。
template_type| 不| 细绳| 定义要使用哪种类型的OpenSearch模板。可用选项是`v1` 和`index-template`。默认值是`v1`，使用原始的OpenSearch模板可用`_template` API端点。这`index-template` 选项使用可复合的[索引模板]({{site.url}}{{site.baseurl}}/opensearch/index-templates/) 可以通过OpenSearch的`_index_template` API。合并索引类型的灵活性比默认值更高，并且当OpenSearch群集已经存在索引模板时，必不可少。可组合模板可用于所有版本的OpenSearch和一些后来版本的Elasticsearch。什么时候`distribution_version` 被设定为`es6`，数据预先执行`template_type` 作为`v1`。
Template_File| 不| 细绳| json的道路[索引模板]({{site.url}}{{site.baseurl}}/opensearch/index-templates/) 文件，例如`/your/local/template-file.json` 什么时候`index_type` 被设定为`custom`。有关示例模板文件，请参阅[奥特尔-V1-APM-跨度-指数-template.json](https://github.com/opensearch-project/data-prepper/blob/main/data-prepper-plugins/opensearch/src/main/resources/otel-v1-apm-span-index-template.json)。如果提供模板文件，则必须匹配由`template_type` 范围。
document_id_field| 不| 细绳| 从源数据到OpenSearch文档ID的字段（例如，`"my-field"`） 如果`index_type` 是`custom`。
DLQ_FILE| 不| 细绳| 您喜欢的死信队列文件的路径（例如，`/your/local/dlq-file`）。当数据未能在OpenSearch集群上索引文档时，数据Prepper将其写入该文件。
DLQ| 不| N/A。| DLQ配置。看[死信队列]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/dlq/) 有关详细信息。如果是`dlq_file` 选项也可用，水槽将失败。
bulk_size| 不| 整数（长）| 发送到OpenSearch cluster的批量请求的最大大小（以MIB为单位）。低于0表示无限尺寸。如果单个文档超过最大批量请求大小，则数据预先列出了单独发送。默认值为5。
ism_policy_file| 不| 细绳| ISM（索引状态管理）策略JSON文件的绝对文件路径。此策略文件仅在没有构建时有效-在索引类型的策略文件中。例如，`custom` 索引类型当前是唯一没有内置的一种-在策略文件中，如果通过此参数提供，它将在此处使用策略文件。有关更多信息，请参阅[ISM政策]({{site.url}}{{site.baseurl}}/im-plugin/ism/policies/)。
number_of_shards| 不| 整数| 索引在目标OpenSearch服务器上应有的主要碎片数量。此参数仅在`template_file` 要么明确提供在水槽配置中或已建造的-在。如果设置此参数，它将覆盖索引模板文件中的值。有关更多信息，请参阅[创建索引]({{site.url}}{{site.baseurl}}/api-reference/index-apis/create-index/)。
number_of_replicas| 不| 整数| 每个主碎片应在目标OpenSearch服务器上具有的复制碎片数。例如，如果您有4个主碎片并将number_of_replicas设置为3，则该索引具有12个复制碎片。此参数仅在`template_file` 要么明确提供在水槽配置中或已建造的-在。如果设置此参数，它将覆盖索引模板文件中的值。有关更多信息，请参阅[创建索引]({{site.url}}{{site.baseurl}}/api-reference/index-apis/create-index/)。
Distribution_version| 不| 细绳| 指示下沉的后端版本是Elasticsearch 6或更高版本。`es6` 代表Elasticsearch 6。`default` 代表最新兼容的后端版本，例如Elasticsearch 7.x，OpenSearch 1.x或OpenSearch 2.x。默认为`default`。
enable_request_compression| 不| 布尔| 是否在向OpenSearch发送请求时启用压缩。什么时候`distribution_version` 被设定为`es6`，默认为`false`。对于所有其他发行版本，默认版本是`true`。

### 配置max_retries

您可以包括`max_retries` 在管道配置中，选项可以控制源试图以指数向后写入水槽的次数。如果您不包含此选项，则管道将永远重试。

如果指定`max_retries` 管道有一个[死的-字母队列（DLQ）]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/dlq/) 配置后，管道将一直试图写入水槽，直到达到最大重试，此时它开始将失败的数据发送到DLQ。

如果您不指定`max_retries`，只有被水槽拒绝的数据写入DLQ。管道继续尝试将所有其他数据写入水槽。

## OpenSearch集群安全性

为了使用该数据将数据发送到OpenSearch集群`opensearch` 接收器插件，您必须在管道配置中指定用户名和密码。以下示例`pipelines.yaml` 文件演示了如何指定管理员安全凭据：

```yaml
sink:
  - opensearch:
      username: "admin"
      password: "admin"
      ...
```

或者，您可以使用以下各节中列出的最小权限来指定映射到角色的用户的凭据，而不是管理凭据。

### 集群权限

- `cluster_all`
- `indices:admin/template/get`
- `indices:admin/template/put`

### 指数许可

- 指数：`otel-v1*`;索引许可：`indices_all`
- 指数：`.opendistro-ism-config`;索引许可：`indices_all`
- 指数：`*`;索引许可：`manage_aliases`

有关如何将用户映射到角色的说明，请参阅[将用户映射到角色]({{site.url}}{{site.baseurl}}/security/access-control/users-roles/#map-users-to-roles)。

## Amazon OpenSearch服务域安全性

这`opensearch` 接收器插件可以将数据发送到[Amazon OpenSearch服务](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/what-is.html) 域，该域使用IAM进行安全。该插件使用默认凭证链。跑步`aws configure` 使用[AWS命令线接口（AWS CLI）](https://aws.amazon.com/cli/) 设置您的凭据。

确保您配置的凭据具有所需的IAM权限。以下域访问策略证明了所需的最低权限：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<AccountId>:user/data-prepper-user"
      },
      "Action": "es:ESHttp*",
      "Resource": [
        "arn:aws:es:us-east-1:<AccountId>:domain/<domain-name>/otel-v1*",
        "arn:aws:es:us-east-1:<AccountId>:domain/<domain-name>/_template/otel-v1*",
        "arn:aws:es:us-east-1:<AccountId>:domain/<domain-name>/_plugins/_ism/policies/raw-span-policy",
        "arn:aws:es:us-east-1:<AccountId>:domain/<domain-name>/_alias/otel-v1*",
        "arn:aws:es:us-east-1:<AccountId>:domain/<domain-name>/_alias/_bulk"
      ]
    },
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<AccountId>:user/data-prepper-user"
      },
      "Action": "es:ESHttpGet",
      "Resource": "arn:aws:es:us-east-1:<AccountId>:domain/<domain-name>/_cluster/settings"
    }
  ]
}
```

有关如何配置域访问策略的说明，请参见[资源-基于政策
]（https://docs.aws.amazon.com/opensearch-服务/最新/developerguide/ac.html#交流-类型-Amazon OpenSearch服务文档中的资源）。

### 美好的-粒度访问控制

如果您的OpenSearch服务域使用[美好的-粒度访问控制](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/fgac.html)， 这`opensearch` 接收器插件需要一些其他配置。

#### 我作为主用户

如果您使用IAM Amazon资源名称（ARN）作为主用户，请包括`aws_sigv4` 接收器配置中的选项：

```yaml
...
sink:
    opensearch:
      hosts: ["https://your-fgac-amazon-opensearch-service-endpoint"]
      aws_sigv4: true
```

跑步`aws configure` 使用AWS CLI使用Master IAM用户凭据。如果您不想使用主用户，则可以使用该角色指定其他IAM角色`aws_sts_role_arn` 选项。然后，插件将使用此角色签署发送到域接收器的请求。您指定的ARN必须包括在[域访问策略]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sinks/opensearch/#amazon-opensearch-service-domain-security)。

#### 内部用户数据库中的主用户

如果您的域在内部用户数据库中使用主用户，请指定主用户名和密码以及`aws_sigv4` 选项：

```yaml
sink:
    opensearch:
      hosts: ["https://your-fgac-amazon-opensearch-service-endpoint"]
      aws_sigv4: false
      username: "master-username"
      password: "master-password"
```

有关更多信息，请参阅[推荐配置](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/fgac.html#fgac-recommendations) 在Amazon OpenSearch服务文档中。

***笔记***：您可以使用该角色创建新的IAM角色或内部用户数据库用户`all_access` 权限并使用它而不是主用户。

## OpenSearch无服务器集合安全

这`opensearch` 接收器插件可以将数据发送到[Amazon OpenSearch无服务器](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/serverless.html) 收藏。

OpenSearch无服务器集合接收器具有以下限制：

- 您无法写入使用虚拟私有云（VPC）访问的集合。必须从公共网络访问该集合。
- Otel Trace Group处理器当前不支持Collection Clinks。

### 创建管道角色

首先，创建一个IAM角色，该角色将为将其写入集合。该角色必须具有以下最低权限：

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "aoss:BatchGetCollection"
            ],
            "Resource": "*"
        }
    ]
}
```

该角色必须具有以下信任关系，这使管道可以假设：

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::<AccountId>:root"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

### 创建一个集合

接下来，创建一个具有以下设置的集合：

- 民众[网络访问](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/serverless-network.html) 到OpenSearch Endpoint和OpenSearch仪表板。
- 下列[数据访问策略](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/serverless-data-access.html)，授予管道角色所需的权限：

  ```JSON
  [
   {
      "Rules"：[[
         {
            "Resource"：[[
               "index/collection-name/*"
            ]，，
            "Permission"：[[
               "aoss:CreateIndex"，
               "aoss:UpdateIndex"，
               "aoss:DescribeIndex"，
               "aoss:WriteDocument"
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

  ***Important***: Make sure to replace the ARN in the `主要的` 元素具有您在上一步中创建的管道角色的ARN。

  有关如何创建收藏的说明，请参阅[创建收藏](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/serverless-manage.html#serverless-create) 在Amazon OpenSearch服务文档中。

### 创建管道

在你内`pipelines.yaml` 文件，指定OpenSearch serverless Collection端点为`hosts` 选项。此外，您必须设置`serverless` 选项`true`。指定管道角色`sts_role_arn` 选项：

```yaml
log-pipeline:
  source:
    http:
  processor:
    - date:
        from_time_received: true
        destination: "@timestamp"
  sink:
    - opensearch:
        hosts: [ "https://<serverless-public-collection-endpoint>" ]
        index: "my-serverless-index"
        aws:
          serverless: true
          sts_role_arn: "arn:aws:iam::<AccountId>:role/PipelineRole"
          region: "us-east-1"
```

