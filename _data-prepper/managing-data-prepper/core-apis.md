---
layout: default
title: 核心API
parent: 管理数据预先
nav_order: 15
---

# 核心API

所有数据预先实例都会用某些控制API公开服务器。默认情况下，该服务器在端口4900上运行。一些插件，尤其是源插件，可能会暴露在不同端口上运行的其他服务器。这些插件的配置与核心API无关。例如，要关闭Data Prepper，您可以运行以下curl请求：

```
curl -X POST http://localhost:4900/shutdown
```

## 蜜蜂

下表列出了可用的API。

| 姓名| 描述|
| --- | --- | 
| ```GET /list```<br>```POST /list``` | 返回运行管道的列表。|
| ```POST /shutdown``` | 开始对数据预先的优雅关闭。|
| ```GET /metrics/prometheus```<br>```POST /metrics/prometheus``` | 以普罗米修斯文本格式返回一系列数据预先指标。此API可作为`metricsRegistries` 数据Prepper配置文件中的参数`data-prepper-config.yaml` 并包含`Prometheus` 作为注册表的一部分。
| ```GET /metrics/sys```<br>```POST /metrics/sys``` | 返回Prometheus文本格式的JVM指标。此API可作为`metricsRegistries` 数据Prepper配置文件中的参数`data-prepper-config.yaml` 并包含`Prometheus` 作为注册表的一部分。

## 配置服务器

您可以通过`data-prepper-config.yaml` 文件。

### SSL/TLS连接

该项目的许多入门指南禁用端点上的SSL：

```yaml
ssl: false
```

要启用数据Prepper端点上的SSL，请配置您的`data-prepper-config.yaml` 具有以下选项的文件：

```yaml
ssl: true
keyStoreFilePath: "/usr/share/data-prepper/keystore.p12"
keyStorePassword: "secret"
privateKeyPassword: "secret"
```

有关使用SSL配置数据Prepper服务器的更多信息，请参见[服务器配置](https://github.com/opensearch-project/data-prepper/blob/main/docs/configuration.md#server-configuration)。如果您正在使用自我-签名证书，您可以添加`-k` 标记以使用SSL快速测试核心API的请求。使用以下内容`shutdown` 请求使用SSL测试核心API：


```
curl -k -X POST https://localhost:4900/shutdown 
```

### 验证

数据PEPPER CORE API支持HTTP基本身份验证。您可以在以下配置中设置用户名和密码`data-prepper-config.yaml` 文件：

```yaml
authentication:
  http_basic:
    username: "myuser"
    password: "mys3cr3t"
```

您可以使用以下配置禁用核心端点的身份验证。请谨慎使用它，因为任何具有网络访问您的数据Prepper实例的人都可以访问关闭API和其他API。

```yaml
authentication:
  unauthenticated:
```

### 同行前锋

可以配置对等转发器以启用跨多个数据预先节点的状态聚合。有关配置对等转发器的更多信息，请参见[同行前锋]({{site.url}}{{site.baseurl}}/data-prepper/managing-data-prepper/peer-forwarder/)。它得到了`service_map_stateful`，，，，`otel_traces_raw`， 和`aggregate` 处理器。

### 关闭超时

运行数据预先`shutdown` API，该过程优雅地关闭并清除了这两个数据的剩余数据`ExecutorService` 下沉和`ExecutorService` 处理器。两个进程关闭的默认超时为10秒。您可以使用以下可选来配置超时`data-prepper-config.yaml` 文件参数：

```yaml
processorShutdownTimeout: "PT15M"
sinkShutdownTimeout: 30s
```

这些参数的值被解析为`Duration` 通过[数据预先持续时间供应器](https://github.com/opensearch-project/data-prepper/tree/main/data-prepper-core/src/main/java/org/opensearch/dataprepper/parser/DataPrepperDurationDeserializer.java)。

