---
layout: default
title: 日志分析
parent: 常见用例
nav_order: 10
---

# 日志分析

Data Prepper是一种可扩展，可配置和可扩展的解决方案，可将日志摄入到OpenSearch和Amazon OpenSearch服务中。数据预先支持从[流利的位](https://fluentbit.io/) 通过[HTTP源](https://github.com/opensearch-project/data-prepper/blob/main/data-prepper-plugins/http-source/README.md) 并用[Grok处理器](https://github.com/opensearch-project/data-prepper/blob/main/data-prepper-plugins/grok-processor/README.md) 在将它们摄入之前[OpenSearch水槽](https://github.com/opensearch-project/data-prepper/blob/main/data-prepper-plugins/opensearch/README.md)。

以下图像显示了用于对数分析的所有组件，并显示了流利的位，数据预先搜索和opensearch。

![日志分析组件]({{site.url}}{{site.baseurl}}/images/data-prepper/log-analytics/log-analytics-components.jpg)

在应用程序环境中，运行流利的位。可以通过Kubernetes，Docker或Amazon弹性容器服务（Amazon ECS）来容器。您也可以在Amazon Elastic Compute Cloud（Amazon EC2）上作为代理运行流利的位。配置[流利的位HTTP输出插件](https://docs.fluentbit.io/manual/pipeline/outputs/http) 将日志数据导出到数据PEPPPER。然后将DATA PREPPER部署为中间组件，然后将其配置为将丰富的日志数据发送到您的OpenSearch cluster。从那里，使用OpenSearch仪表板进行更深入的可视化和分析。

## 日志分析管道

数据预珀中的日志分析管道非常可定制。下图显示了一个简单的管道。

![日志分析组件]({{site.url}}{{site.baseurl}}/images/data-prepper/log-analytics/log-ingestion-pipeline.jpg)

### HTTP源

这[HTTP源](https://github.com/opensearch-project/data-prepper/blob/main/data-prepper-plugins/http-source/README.md) 接受来自Fluent位的日志数据。该来源以JSON阵列格式接受日志数据并支持行业-TLS/HTTPS和HTTP基本身份验证的形式的标准加密。

### 处理器

数据Prepper 1.2及以上带有[Grok处理器](https://github.com/opensearch-project/data-prepper/blob/main/data-prepper-plugins/grok-processor/README.md)。Grok处理器是从日志中构造和提取重要字段的宝贵工具，使其更可查询。

Grok处理器带有各种各样的[默认模式](https://github.com/thekrakken/java-grok/blob/master/src/main/resources/patterns/patterns) 匹配通用日志格式（例如Apache Logs或Syslogs），但它可以轻松接受迎合您特定日志格式的任何自定义模式。

有关Grok功能的更多信息，请参见文档。

### 下沉

有一个通用的水槽将数据写入OpenSearch作为目的地。这[OpenSearch水槽](https://github.com/opensearch-project/data-prepper/blob/main/data-prepper-plugins/opensearch/README.md) 具有与OpenSearch集群相关的配置选项，例如端点，SSL/用户名，索引名称，索引模板和索引状态管理。

## 管道配置

以下各节讨论管道配置。

### 带有SSL和基本身份验证的示例管道

此示例管道配置随附SSL，并启用了基本身份验证`http-source`：

```yaml
log-pipeline:
  source:
    http:
      ssl_certificate_file: "/full/path/to/certfile.crt"
      ssl_key_file: "/full/path/to/keyfile.key"
      authentication:
        http_basic:
          username: "myuser"
          password: "mys3cret"
  processor:
    - grok:
        match:
          # This will match logs with a "log" key against the COMMONAPACHELOG pattern (ex: { "log": "actual apache log..." } )
          # You should change this to match what your logs look like. See the grok documenation to get started.
          log: [ "%{COMMONAPACHELOG}" ]
  sink:
    - opensearch:
        hosts: [ "https://localhost:9200" ]
        # Change to your credentials
        username: "admin"
        password: "admin"
        # Add a certificate file if you are accessing an OpenSearch cluster with a self-signed certificate  
        #cert: /path/to/cert
        # If you are connecting to an Amazon OpenSearch Service domain without
        # Fine-Grained Access Control, enable these settings. Comment out the
        # username and password above.
        #aws_sigv4: true
        #aws_region: us-east-1
        # Since we are grok matching for apache logs, it makes sense to send them to an OpenSearch index named apache_logs.
        # You should change this to correspond with how your OpenSearch indexes are set up.
        index: apache_logs
```

该管道配置是Apache日志摄入的示例。不要忘记，您可以轻松地为自己的自定义日志配置Grok处理器。您需要修改OpenSearch群集的配置。

以下是您需要进行的主要更改：

*`hosts`  - 设置为您的主机。
*`index`  - 将其更改为要发送日志的OpenSearch索引。
*`username`  - 提供您的OpenSearch用户名。
*`password`  - 提供您的OpenSearch密码。
*`aws_sigv4`  - 如果您使用AMAZON OPENSEARCH服务与AWS签名，请将其设置为true。它将与默认AWS凭据提供商签署请求。
*`aws_region`  - 如果您使用AMAZON OPENSEARCH服务与AWS签名，请将此值设置为托管群集的AWS区域。

## 流利的位

您需要在服务环境中流利。看[稍有刻度开始](https://docs.fluentbit.io/manual/installation/getting-started-with-fluent-bit) 用于安装说明。确保您可以配置[流利的位HTTP输出插件](https://docs.fluentbit.io/manual/pipeline/outputs/http) 您的数据PEPPER HTTP源。以下是一个例子`fluent-bit.conf` 尾随一个名为的日志文件`test.log` 并将其转发到本地运行的数据Prepper HTTP源，该源默认在端口2021上运行。

请注意，您应该调整文件`path`， 输出`Host`， 和`Port` 根据您的方式和位置，您的位和数据预先运行。

### 示例：不带SSL的Fluent Bit文件，并启用了基本身份验证

以下是一个例子`fluent-bit.conf` 在HTTP源上启用了没有SSL的文件，并启用了基本身份验证：

```
[INPUT]
  name                  tail
  refresh_interval      5
  path                  test.log
  read_from_head        true

[OUTPUT]
  Name http
  Match *
  Host localhost
  Port 2021
  URI /log/ingest
  Format json
```

如果您的HTTP源具有SSL和基本身份验证，则需要添加`http_User`，，，，`http_Passwd`，，，，`tls.crt_file`， 和`tls.key_file` 到`fluent-bit.conf` 文件，如下示例所示。

### 示例：使用SSL和基本身份验证的流利位文件

以下是一个例子`fluent-bit.conf` 在HTTP源上启用了SSL和基本身份验证的文件：

```
[INPUT]
  name                  tail
  refresh_interval      5
  path                  test.log
  read_from_head        true

[OUTPUT]
  Name http
  Match *
  Host localhost
  http_User myuser
  http_Passwd mys3cret
  tls On
  tls.crt_file /full/path/to/certfile.crt
  tls.key_file /full/path/to/keyfile.key
  Port 2021
  URI /log/ingest
  Format json
```

# 下一步

看到[数据预先日志摄入演示指南](https://github.com/opensearch-project/data-prepper/blob/main/examples/log-ingestion/README.md) 对于Apache日志摄入的特定示例`FluentBit -> Data Prepper -> OpenSearch` 穿过Docker。

将来，数据Prepper将提供其他来源和处理器，以使更复杂的日志分析管道可用。查看[数据预先项目路线图](https://github.com/opensearch-project/data-prepper/projects/1) 看看即将发生的事情。

如果您想在日志分析工作流中包含的特定来源，处理器或水槽，并且目前不在路线图上，请通过创建GitHub问题引起我们的注意。此外，如果您有兴趣为Data Prepper做出贡献，请参阅我们的[贡献准则](https://github.com/opensearch-project/data-prepper/blob/main/CONTRIBUTING.md) 以及我们的[开发人员指南](https://github.com/opensearch-project/data-prepper/blob/main/docs/developer_guide.md) 和[插件开发指南](https://github.com/opensearch-project/data-prepper/blob/main/docs/plugin_development.md)。

