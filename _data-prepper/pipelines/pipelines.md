---
layout: default
title: 管道
has_children: true
nav_order: 10
redirect_from:
  - /data-prepper/pipelines/
  - /clients/data-prepper/pipelines/
---

# 管道

下图说明了管道的工作原理。

<img src="{{site.url}}{{site.baseurl}}/images/data-prepper-pipeline.png" alt="Data Prepper pipeline">{: .img-fluid}

要使用PATA PEPPPER，您可以在配置YAML文件中定义管道。每个管道都是源，缓冲区，零或更多处理器以及一个或多个下沉的组合。例如：

```yml
simple-sample-pipeline:
  workers: 2 # the number of workers
  delay: 5000 # in milliseconds, how long workers wait between read attempts
  source:
    random:
  buffer:
    bounded_blocking:
      buffer_size: 1024 # max number of records the buffer accepts
      batch_size: 256 # max number of records the buffer drains after each read
  processor:
    - string_converter:
        upper_case: true
  sink:
    - stdout:
```

- 来源定义了您的数据来自何处。在这种情况下，源是一个随机的UUID发生器（`random`）。

- 缓冲区通过管道的数据存储数据。

  默认情况下，Data Prepper使用其唯一的缓冲区，`bounded_blocking` 缓冲区，因此您可以省略此部分，除非您开发了自定义缓冲区或需要调整缓冲区设置。

- 处理器对您的数据执行一些操作：过滤，转换，丰富，等。

  您可以拥有多个处理器，这些处理器从上到下依次运行，而不是并行运行。这`string_converter` 处理器通过使它们的大写通过将其转换。

- 接收器定义数据的去向。在这种情况下，水槽是Stdout。

从数据Prepper 2.0开始，您可以在多个配置yaml文件上定义管道，其中每个文件包含一个或多个管道的配置。这为您提供了更多组织和链条复杂管道配置的自由。要正确加载管道配置，请将您的配置yaml文件放在`pipelines` 您应用程序主目录下的文件夹（例如`/usr/share/data-prepper`）。
{:.note}

## 结尾-到-结束致谢

数据预先确保从源书写的数据的耐用性和可靠性-到-结束（E2E）致谢。E2E确认始于来源，该消息源是在管道内部设置的一批事件，并在成功将这些事件成功推向下沉时等待积极的确认。当管道包含多个接收器（包括设置为其他数据预先管道管道）的水槽时，在管道链中的最终水槽收到事件时，E2E确认会发送。

另外，当事件因任何原因无法传递到水槽时，源会发送负面确认。

当管道的任何组件失败并且无法发送事件时，源未收到确认。在失败的情况下，管道的来源会耗尽。这使您能够采取任何必要的措施来解决源失败，包括重新启动管道或记录故障。


## 条件路由

管道也支持**条件路由**  这使您可以根据特定条件将事件路由到不同的水槽。要将条件路由添加到管道中，请指定下方的指定路由列表`route` 组件并将特定路线添加到下沉`routes` 财产。与`routes` 属性只会接受与至少一个路由条件匹配的事件。

在以下示例中，`application-logs` 是一个条件设置为条件的命名路线`/log_type == "application"`。路线使用[数据预先表达式](https://github.com/opensearch-project/data-prepper/tree/main/examples) 定义条件。Data Prepper仅将满足条件的事件路由到第一次开放搜索渠道。默认情况下，数据将所有事件的所有事件都路由到没有定义路线的水槽。在示例中，所有事件都路由到第三次开放式搜索接收器。

```yml
conditional-routing-sample-pipeline:
  source:
    http:
  processor:
  route:
    - application-logs: '/log_type == "application"'
    - http-logs: '/log_type == "apache"'
  sink:
    - opensearch:
        hosts: [ "https://opensearch:9200" ]
        index: application_logs
        routes: [application-logs]
    - opensearch:
        hosts: [ "https://opensearch:9200" ]
        index: http_logs
        routes: [http-logs]
    - opensearch:
        hosts: [ "https://opensearch:9200" ]
        index: all_logs
```


## 例子

本节提供了一些管道示例，您可以用来开始创建自己的管道。有关更多管道配置，请从每个组件中从以下选项中进行选择：

- [缓冲区]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/buffers/buffers/)
- [处理器]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/processors/processors/)
- [水槽]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sinks/sinks/)
- [来源]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sources/sources/)

数据Prepper存储库有几个[样本应用](https://github.com/opensearch-project/data-prepper/tree/main/examples) 为了帮助您开始。

### 日志摄入管道

以下示例`pipeline.yaml` 使用SSL和基本身份验证的文件启用了`http-source` 演示如何使用HTTP源和Grok Prepper插件来处理非结构化日志数据：


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
        # Since we are Grok matching for Apache logs, it makes sense to send them to an OpenSearch index named apache_logs.
        # You should change this to correspond with how your OpenSearch indexes are set up.
        index: apache_logs
```

此示例使用弱安全性。我们强烈建议您确保在生产环境中打开外部端口的所有插件。
{:.note}

### 跟踪分析管道

以下示例演示了如何构建支持该管道的管道[跟踪分析opensearch仪表板插件]({{site.url}}{{site.baseurl}}/observability-plugin/trace/ta-dashboards/)。该管道从OpenTelemetry Collector中获取数据，并使用其他两个管道作为水槽。这两个单独的管道索引跟踪和仪表板插件的服务映射文档。

从数据Prepper 2.0开始，Data Prepper不再支持`otel_trace_raw_prepper` 由于数据预先内部数据模型演变而引起的处理器。
相反，用户应使用`otel_trace_raw`。

```yml
entry-pipeline:
  delay: "100"
  source:
    otel_trace_source:
      ssl: false
  buffer:
    bounded_blocking:
      buffer_size: 10240
      batch_size: 160
  sink:
    - pipeline:
        name: "raw-pipeline"
    - pipeline:
        name: "service-map-pipeline"
raw-pipeline:
  source:
    pipeline:
      name: "entry-pipeline"
  buffer:
    bounded_blocking:
      buffer_size: 10240
      batch_size: 160
  processor:
    - otel_trace_raw:
  sink:
    - opensearch:
        hosts: ["https://localhost:9200"]
        insecure: true
        username: admin
        password: admin
        index_type: trace-analytics-raw
service-map-pipeline:
  delay: "100"
  source:
    pipeline:
      name: "entry-pipeline"
  buffer:
    bounded_blocking:
      buffer_size: 10240
      batch_size: 160
  processor:
    - service_map_stateful:
  sink:
    - opensearch:
        hosts: ["https://localhost:9200"]
        insecure: true
        username: admin
        password: admin
        index_type: trace-analytics-service-map
```

要保持相似的摄入吞吐量和潜伏期，请扩展`buffer_size` 和`batch_size` 按照客户端请求有效载荷中估计的最大批量大小。
{:.tip}

### 指标管道

数据PEPPER支持使用Otel摄入指标。它目前支持以下指标类型：

* 测量
*总和
* 概括
*直方图

其他类型不支持。数据预先删除所有其他类型，包括指数直方图和摘要。此外，数据预先不支持范围仪器。

设置指标管道：

```yml
metrics-pipeline:
  source:
    otel_metrics_source:
  processor:
    - otel_metrics_raw_processor:
  sink:
    - opensearch:
      hosts: ["https://localhost:9200"]
      username: admin
      password: admin
```

### S3日志摄入管道

以下示例演示了如何使用S3source和Grok处理器插件来处理非结构化日志数据[亚马逊简单存储服务](https://aws.amazon.com/s3/) （亚马逊S3）。此示例使用应用程序负载平衡器日志。当应用程序负载平衡器将日志写入S3时，S3在Amazon SQS中创建通知。数据PEPPER监视这些通知并读取S3对象以获取日志数据并处理它。

```yml
log-pipeline:
  source:
    s3:
      notification_type: "sqs"
      compression: "gzip"
      codec:
        newline:
      sqs:
        queue_url: "https://sqs.us-east-1.amazonaws.com/12345678910/ApplicationLoadBalancer"
      aws:
        region: "us-east-1"
        sts_role_arn: "arn:aws:iam::12345678910:role/Data-Prepper"

  processor:
    - grok:
        match:
          message: ["%{DATA:type} %{TIMESTAMP_ISO8601:time} %{DATA:elb} %{DATA:client} %{DATA:target} %{BASE10NUM:request_processing_time} %{DATA:target_processing_time} %{BASE10NUM:response_processing_time} %{BASE10NUM:elb_status_code} %{DATA:target_status_code} %{BASE10NUM:received_bytes} %{BASE10NUM:sent_bytes} \"%{DATA:request}\" \"%{DATA:user_agent}\" %{DATA:ssl_cipher} %{DATA:ssl_protocol} %{DATA:target_group_arn} \"%{DATA:trace_id}\" \"%{DATA:domain_name}\" \"%{DATA:chosen_cert_arn}\" %{DATA:matched_rule_priority} %{TIMESTAMP_ISO8601:request_creation_time} \"%{DATA:actions_executed}\" \"%{DATA:redirect_url}\" \"%{DATA:error_reason}\" \"%{DATA:target_list}\" \"%{DATA:target_status_code_list}\" \"%{DATA:classification}\" \"%{DATA:classification_reason}"]
    - grok:
        match:
          request: ["(%{NOTSPACE:http_method})? (%{NOTSPACE:http_uri})? (%{NOTSPACE:http_version})?"]
    - grok:
        match:
          http_uri: ["(%{WORD:protocol})?(://)?(%{IPORHOST:domain})?(:)?(%{INT:http_port})?(%{GREEDYDATA:request_uri})?"]
    - date:
        from_time_received: true
        destination: "@timestamp"


  sink:
    - opensearch:
        hosts: [ "https://localhost:9200" ]
        username: "admin"
        password: "admin"
        index: alb_logs
```

## 从logstash迁移

Data Prepper支持一组有限的插件的LogStash配置文件。只需使用LogStash配置即可运行数据预备。

```bash
docker run --name data-prepper \
    -v /full/path/to/logstash.conf:/usr/share/data-prepper/pipelines/pipelines.conf \
    opensearchproject/opensearch-data-prepper:latest
```

此功能受数据预珀的特征均衡限制。在数据Prepper 1.2发布时，支持LogStash配置中的以下插件：

- HTTP输入插件
- Grok过滤器插件
- Elasticsearch输出插件
- Amazon Elasticsearch输出插件

## 配置数据Prepper服务器

数据Prepper本身提供管理HTTP端点，例如`/list` 列出管道和`/metrics/prometheus` 提供普罗米修-兼容指标数据。具有这些端点的端口具有TLS配置，并由单独的YAML文件指定。默认情况下，这些端点是由Data Prepper Docker图像确保的。我们强烈建议您提供自己的配置文件，以保护生产环境。这是一个例子`data-prepper-config.yaml`：

```yml
ssl: true
keyStoreFilePath: "/usr/share/data-prepper/keystore.jks"
keyStorePassword: "password"
privateKeyPassword: "other_password"
serverPort: 1234
```

要配置数据Prepper服务器，请使用附加YAML文件运行Data Prepper。

```bash
docker run --name data-prepper \
    -v /full/path/to/my-pipelines.yaml:/usr/share/data-prepper/pipelines/my-pipelines.yaml \
    -v /full/path/to/data-prepper-config.yaml:/usr/share/data-prepper/data-prepper-config.yaml \
    opensearchproject/data-prepper:latest
```

## 配置对等转发器

Data Prepper提供了HTTP服务，可在数据预先节点之间转发事件以进行聚合。这是在集群部署中操作数据预先操作所必需的。目前，支持同伴转发`aggregate`，`service_map_stateful`， 和`otel_trace_raw` 处理器。同行转发器根据处理器提供的标识密钥将事件组成。为了`service_map_stateful` 和`otel_trace_raw` 它是`traceId` 默认情况下，无法配置。为了`aggregate` 处理器，可以使用`identification_keys` 选项。

PEER Forwarder通过以下三个选项之一支持同行发现：静态列表，DNS记录查找或AWS云映射。可以使用同行发现使用`discovery_mode` 选项。Peer Fewracker还支持SSL进行验证和加密，以及在同伴转发服务中相互认证的MTL。

要配置对等转发器，请将配置选项添加到`data-prepper-config.yaml` 在[配置数据Prepper服务器](#configure-the-data-prepper-server) 部分：

```yml
peer_forwarder:
  discovery_mode: dns
  domain_name: "data-prepper-cluster.my-domain.net"
  ssl: true
  ssl_certificate_file: "<cert-file-path>"
  ssl_key_file: "<private-key-file-path>"
  authentication:
    mutual_tls:
```


## 管道配置

由于数据Prepper 2.5，因此可以在“保留部分”下配置共享管道组件`pipeline_configurations` 当在单个管道配置yaml文件中定义所有管道时。
共享管道配置可以包含某些组件[扩展插件]({{site.url}}{{site.baseurl}}/data-prepper/managing-data-prepper/configuring-data-prepper/#extension-plugins)，如以下示例所示，指的是秘密配置`opensearch` 下沉：

```json
pipeline_configurations:
  aws:
    secrets:
      credential-secret-config:
        secret_id: <YOUR_SECRET_ID>
        region: <YOUR_REGION>
        sts_role_arn: <YOUR_STS_ROLE_ARN>
simple-sample-pipeline:
  ...
  sink:
    - opensearch:
        hosts: [ {% raw %}"${{aws_secrets:host-secret-config}}"{% endraw %} ]
        username: {% raw %}"${{aws_secrets:credential-secret-config:username}}"{% endraw %}
        password: {% raw %}"${{aws_secrets:credential-secret-config:password}}"{% endraw %}
        index: "test-migration"
```

当两者中定义相同的组件时`pipelines.yaml` 和`data-prepper-config.yaml`，在`pipelines.yaml` 将覆盖对应物`data-prepper-config.yaml`。有关共享管道组件的更多信息，请参见[AWS Secrets扩展插件]({{site.url}}{{site.baseurl}}/data-prepper/managing-data-prepper/configuring-data-prepper/#aws-secrets-extension-plugin) 有关详细信息。

