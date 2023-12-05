---
layout: default
title: 配置数据预备
parent: 管理数据预先
nav_order: 5
redirect_from:
 - /clients/data-prepper/data-prepper-reference/
 - /monitoring-plugins/trace/data-prepper-reference/
---

# 配置数据预备

您可以通过编辑来自定义数据预先配置`data-prepper-config.yaml` 在您的数据中提前安装中文件。以下配置选项独立于管道配置选项。


## 数据预先配置

使用以下选项自定义数据预先配置。

选项| 必需的| 类型| 描述
:--- | :--- |:--- | :---
SSL| 不| 布尔| 指示是否应将TLS用于服务器API。默认为true。
KeystoreFilePath| 不| 细绳| .jks或.p12密钥库文件的路径。如果需要`ssl` 是真的。
KeyStorePassword| 不| 细绳| 密钥库的密码。可选，默认为空字符串。
私有keypassword| 不| 细绳| 密钥店内专用密钥的密码。可选，默认为空字符串。
服务器端口| 不| 整数| 用于服务器API的端口号。默认为4900。
公制| 不| 列表| 发布生成指标的指标注册表。目前支持Prometheus和Amazon CloudWatch。默认为Prometheus。
METRICTAGS| 不| 地图| 密钥地图-价值对作为公制登记处的通用度量标签。最大对数为三个。注意`serviceName` 是一个保留的标签键`DataPrepper` 作为默认标签值。另外，管理员可以通过环境变量设置此值`DATAPREPPER_SERVICE_NAME`。如果`serviceName` 定义在`metricTags`，该值覆盖了通过上述方法设置的值。
验证| 不| 目的| 身份验证配置。有效选项是`http_basic` 和`username` 和`password` 特性。如果未定义，服务器将不执行身份验证。
processorshutdowntimeout| 不| 期间| 给处理器清除任何内容的时间-飞行数据并优雅地关闭。默认值为30。
sinkShutdowntimeout| 不| 期间| 水分的时间清除了-飞行数据并优雅地关闭。默认值为30。
PEER_FORWARDER| 不| 目的| 对等转发器配置。看[对等转发器选项](#peer-forwarder-options) 更多细节。
电流_BREAKERS| 不| [电流_BREAKERS](#circuit-breakers) | 在传入数据上配置断路器。
扩展| 不| 目的| 管道扩展插件配置。看[扩展插件](#extension-plugins) 更多细节。

### 对等转发器选项

以下一节详细介绍了Peer Forwarder的各种配置选项。

#### 同行转发的一般选项

选项| 必需的| 类型| 描述
:--- | :--- | :--- | :---
港口| 不| 整数| 对等转发服务器端口。有效选项在0到65535之间。默认值为4994。
请求超时| 不| 整数| 同伴转发器HTTP服务器的请求超时。默认值为10000。
server_thread_count| 不| 整数| 对等转发器服务器使用的线程数。默认值为200。
client_thread_count| 不| 整数| 对等转发器客户端使用的线程数。默认值为200。
max_connection_count| 不| 整数| PEER Expryer Server的最大开放连接数量。默认值为500。
max_pending_requests| 不| 整数| 在计划中的工作队列中，允许的任务的最大数量。默认值为1024。
Discovery_Mode| 不| 细绳| 使用的同行发现模式。有效的选项是`local_node`，`static`，`dns`， 或者`aws_cloud_map`。默认为`local_node`，哪个在本地处理事件。
static_endpoints| 有条件的| 列表| 包含所有数据预先实例的端点的列表。如果需要`discovery_mode` 设置为静态。
domain_name| 有条件的| 细绳| 一个单个域名以查询DNS。通常，通过为同一域创建多个DNS记录来使用。如果需要`discovery_mode` 设置为DNS。
aws_cloud_map_namespace_name| 有条件的| 细绳| 使用AWS云地图服务发现时，云地图名称空间。如果需要`discovery_mode` 被设定为`aws_cloud_map`。
aws_cloud_map_service_name| 有条件的| 细绳| 使用AWS云地图服务发现时，云地图服务名称。如果需要`discovery_mode` 被设定为`aws_cloud_map`。
aws_cloud_map_query_parameters| 不| 地图| 密钥地图-值对根据附加到实例的自定义属性过滤结果。只有匹配所有指定密钥的实例-返回价值对。
缓冲区大小| 不| 整数| 缓冲区接受的最大未检查记录。未选中的记录的数量是写入缓冲区中的记录的总和，-尚未由检查点API检查的飞行记录。默认值为512。
batch_size| 不| 整数| 最大记录数量缓冲区读取时返回。默认值为48。
aws_region| 有条件的| 细绳| 与ACM，S3或AWS云图一起使用的AWS区域。如果需要`use_acm_certificate_for_ssl` 设置为true或`ssl_certificate_file` 和`ssl_key_file` 是AWS S3路径还是`discovery_mode` 被设定为`aws_cloud_map`。
drain_timeout| 不| 期间| 在关闭之前，同伴转发器完成处理数据的等待时间。默认为`10s`。

#### TLS/SSL选项用于PEER throwder

选项| 必需的| 类型| 描述
:--- | :--- | :--- | :---
SSL| 不| 布尔| 启用TLS/SSL。默认为`true`。
ssl_certificate_file| 有条件的| 细绳| SSL证书链文件路径或AWS S3路径。S3路径示例`s3://<bucketName>/<path>`。如果需要`ssl` 是真的`use_acm_certificate_for_ssl` 是错误的。默认为`config/default_certificate.pem` 这是默认证书文件。阅读有关证书文件的生成方式的更多信息[这里](https://github.com/opensearch-project/data-prepper/tree/main/examples/certificates)。
ssl_key_file| 有条件的| 细绳| SSL密钥文件路径或AWS S3路径。S3路径示例`s3://<bucketName>/<path>`。如果需要`ssl` 是真的`use_acm_certificate_for_ssl` 是错误的。默认为`config/default_private_key.pem` 这是默认的私钥文件。阅读更多有关如何生成默认专用密钥文件的更多信息[这里](https://github.com/opensearch-project/data-prepper/tree/main/examples/certificates)。
ssl_insecure_disable_verification| 不| 布尔| 禁用服务器TLS证书链的验证。默认值为false。
ssl_fingerprint_verification_only| 不| 布尔| 禁用服务器TLS证书链的验证，而仅验证证书指纹。默认值为false。
use_acm_certificate_for_ssl| 不| 布尔| 使用AWS证书经理（ACM）的证书和私钥启用TLS/SSL。默认值为false。
acm_certificate_arn| 有条件的| 细绳| ACM证书Arn。ACM证书优先于S3或本地文件系统证书。如果需要`use_acm_certificate_for_ssl` 设置为true。
acm_private_key_password| 不| 细绳| 解密私钥的ACM私钥密码。如果未提供，则数据PEPPER会生成随机密码。
acm_certificate_timeout_millis| 不| 整数| ACM以毫秒为单位获得证书。默认值为120000。
aws_region| 有条件的| 细绳| 使用ACM，S3或AWS云图的AWS区域。如果需要`use_acm_certificate_for_ssl` 设置为true或`ssl_certificate_file` 和`ssl_key_file` 是AWS S3路径还是`discovery_mode` 被设定为`aws_cloud_map`。

#### 同行转发器的身份验证选项

选项| 必需的| 类型| 描述
:--- | :--- | :--- | :---
验证| 不| 地图| 使用的身份验证方法。有效的选项是`mutual_tls` （使用mtls）或`unauthenticated` （无身份验证）。默认为`unauthenticated`。

### 断路器

Data Prepper提供了断路器，以帮助防止耗尽Java内存。当管道具有状态处理器时，非常有用，因为这些处理器可以在缓冲区之外保留内存使用情况。

当断路器被绊倒时，数据预先拒绝传入的数据路由到缓冲区。


选项| 必需的| 类型| 描述
:--- | :--- |：---| ：---
堆| 不| [堆](#heap-circuit-breaker) | 启用堆断路器。默认情况下，未启用此功能。


#### 堆断路器

当JVM Heap达到指定的使用阈值时，配置数据预先绊倒的断路器。

选项| 必需的| 类型| 描述
：--- |:---	|:---	| ：---
用法| 是的| 字节| 指定JVM堆的用法，以使断路器绊倒。如果当前的Java堆使用超过此值，则断路器将打开。这可以是一个值`6.5gb`。
重置| 不| 期间| 绊倒断路器后，直到这段时间通过后才进行新的检查。这有效地设定了断路器保持打开以允许清除内存的最短时间。默认为`1s`。
check_interval| 不| 期间| 指定堆大小的检查之间的时间。默认为`500ms`。

### 扩展插件

由于Data Prepper 2.5，Data Prepper为用户可配置的扩展插件提供了支持。扩展插件共享常见
跨管道插件共享的配置，例如[来源，缓冲区，处理器和下沉]({{site.url}}{{site.baseurl}}/data-prepper/index/#concepts)。

### AWS扩展插件

要使用AWS扩展插件，请将以下设置添加到您的`data-prepper-config.yaml` 在下面`aws`。

选项| 必需的| 类型| 描述
：--- |:---	|:---	| ：---
AWS| 不| 目的| AWS扩展插件配置。

#### AWS Secrets扩展插件

AWS Secrets扩展插件配置[AWS Secrets经理](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html) 成为
如下所示，在管道插件配置中引用：

```json
extensions:
  aws:
    secrets:
      <YOUR_SECRET_CONFIG_ID_1>:
        secret_id: <YOUR_SECRET_ID_1>
        region: <YOUR_REGION_1>
        sts_role_arn: <YOUR_STS_ROLE_ARN_1>
        refresh_interval: <YOUR_REFRESH_INTERVAL>
      <YOUR_SECRET_CONFIG_ID_2>:
        ...
```

要使用Secrets扩展插件，请将以下设置添加到您的`pipeline.yaml` 在下面`extensions` >`aws`。

选项| 必需的| 类型| 描述
：--- |:---	|:---	| ：---
秘密| 不| 目的| AWS Secrets Manager扩展插件配置。看[秘密](#secrets) 更多细节。

### 秘密

使用以下设置`secrets` 扩展设置。


选项| 必需的| 类型| 描述
：--- |:---	|:---	| ：---
secret_id| 是的| 细绳| AWS秘密名称或Arn。|
地区| 不| 细绳| 秘密的AWS地区。默认为`us-east-1`。
sts_role_arn| 不| 细绳| AWS安全令牌服务（AWS STS）角色要担任向AWS Secrets Manager的请求。默认为`null`，将使用[凭证的标准SDK行为](https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/credentials.html)。
刷新间隔| 不| 期间| AWS Secrets扩展插件的刷新间隔将用于调查新的秘密值。默认为`PT1H`。看[自动清新秘密](#automatically-refreshing-secrets) 有关详细信息。

#### 参考秘密
ß
在`pipelines.yaml`，可以使用以下格式在管道插件中引用秘密值：

* 纯文本：`{% raw %}${{aws_secrets:<YOUR_SECRET_CONFIG_ID>}}{% endraw %}`。
* JSON（钥匙-价值对）：`{% raw %}${{aws_secrets:<YOUR_SECRET_CONFIG_ID>:<YOUR_KEY>}}{% endraw %}`

 
代替`<YOUR_SECRET_CONFIG_ID>` 具有相应的秘密配置ID`/extensions/aws/secrets`。代替`<YOUR_KEY>` 带有秘密JSON值所需的键。可以为以下插件设置数据类型解释秘密值参考字符串格式：

* 细绳
* 数字
* 长的
* 短的
* 整数
* 双倍的
* 漂浮
*布尔
* 特点

以下示例部分`data-prepper-config.yaml` 命名两个秘密配置ID，`host-secret-config` 和`credential-secret-config`：


```json
extensions:
  aws:
    secrets:
      host-secret-config:
        secret_id: <YOUR_SECRET_ID_1>
        region: <YOUR_REGION_1>
        sts_role_arn: <YOUR_STS_ROLE_ARN_1>
        refresh_interval: <YOUR_REFRESH_INTERVAL_1>
      credential-secret-config:
        secret_id: <YOUR_SECRET_ID_2>
        region: <YOUR_REGION_2>
        sts_role_arn: <YOUR_STS_ROLE_ARN_2>
        refresh_interval: <YOUR_REFRESH_INTERVAL_2>
```

后`<YOUR_SECRET_CONFIG_ID>` 已配置，您可以参考您的ID`pipelines.yaml`：

```
sink:
    - opensearch:
        hosts: [ {% raw %}"${{aws_secrets:host-secret-config}}"{% endraw %} ]
        username: {% raw %}"${{aws_secrets:credential-secret-config:username}}"{% endraw %}
        password: {% raw %}"${{aws_secrets:credential-secret-config:password}}"{% endraw %}
        index: "test-migration"
```


#### 自动清新秘密

对于每种单独的秘密配置，最新的秘密价值会定期进行轮询，以支持AWS Secrets Manager中的清爽秘密。某些管道插件可利用刷新的秘密值来刷新其组件，例如连接和与后端服务的身份验证。

对于多种秘密配置，请抖动`60s` 在初始秘密进行轮询期间，将在所有配置中应用。

