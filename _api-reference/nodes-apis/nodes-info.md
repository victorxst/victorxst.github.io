---
layout: default
title: 节点信息
parent: 节点API
nav_order: 10
---

# 节点信息
**引入1.0**
{: .label .label-purple }

节点信息API主要表示有关群集节点的静态信息，包括但不限于：

- 主机系统信息
- JVM
- 处理器类型
- 节点设置
- 线程池设置
- 已安装的插件

## 例子

要获取集群中所有节点的信息，请使用以下查询：

```json
GET /_nodes
```
{% include copy-curl.html %}

要获取有关群集管理器节点的线程池信息，请使用以下查询：

```json
GET /_nodes/master:true/thread_pool
```
{% include copy-curl.html %}

## 路径和HTTP方法

```bash
GET /_nodes
GET /_nodes/<nodeId>
GET /_nodes/<metrics>
GET /_nodes/<nodeId>/<metrics>
# or full path equivalent
GET /_nodes/<nodeId>/info/<metrics>
```

## 路径参数

下表列出了可用路径参数。所有路径参数都是可选的。

范围| 类型| 描述
:--- |:-------| :---
nodeid| 细绳| 逗号-用于过滤结果的节点的分离列表。支持[节点过滤器]({{site.url}}{{site.baseurl}}/api-reference/nodes-apis/index/#node-filters)。默认为`_all`。
指标| 细绳| 逗号-将包括在响应中的公制组的分开列表。例如，`jvm,thread_pool`。默认为所有指标。

下表列出了所有可用的公制组。

公制| 描述
:--- |:----
设置| 节点的设置。这是默认设置的组合，从[配置文件]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/#configuration-file)和动态[更新的设置]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/#updating-cluster-settings-using-the-api)。
操作系统| 有关主机OS的静态信息，包括版本，处理器体系结构以及可用/分配的处理器。
过程| 包含过程ID。
JVM| 有关运行JVM的详细静态信息，包括参数。
thread_pool| 为所有单个线程池配置的选项。
运输| 主要是有关运输层的静态信息。
http| 主要是有关HTTP层的静态信息。
插件| 有关已安装插件和模块的信息。
摄取| 有关摄入管道和可用摄入处理器的信息。
聚合| 有关可用的信息[聚合]({{site.url}}{{site.baseurl}}/opensearch/aggregations)。
指数| 在节点级别配置的静态索引设置。

## 查询参数

您可以在请求中包含以下查询参数。所有查询参数都是可选的。

范围| 类型| 描述
:--- |:-------| :---
flat_settings| 布尔| 指定是否返回`settings` 响应的对象以平面格式。默认为`false`。
暂停| 时间| 设置节点响应的时间限制。默认值是`30s`。

#### 示例请求

以下查询请求`process` 和`transport` 集群管理器节点的指标：

```json
GET /_nodes/cluster_manager:true/process,transport
```
{% include copy-curl.html %}

#### 示例响应

响应包含在`<metrics>` 请求参数（在这种情况下，`process` 和`transport`）：

```json
{
  "_nodes": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "cluster_name": "opensearch",
  "nodes": {
    "VC0d4RgbTM6kLDwuud2XZQ": {
      "name": "node-m1-23",
      "transport_address": "127.0.0.1:9300",
      "host": "127.0.0.1",
      "ip": "127.0.0.1",
      "version": "1.3.1",
      "build_type": "tar",
      "build_hash": "c4c0672877bf0f787ca857c7c37b775967f93d81",
      "roles": [
        "data",
        "ingest",
        "master",
        "remote_cluster_client"
      ],
      "attributes": {
        "shard_indexing_pressure_enabled": "true"
      },
      "process" : {
        "refresh_interval_in_millis": 1000,
        "id": 44584,
        "mlockall": false
      },
      "transport": {
        "bound_address": [
          "[::1]:9300",
          "127.0.0.1:9300"
        ],
        "publish_address": "127.0.0.1:9300",
        "profiles": { }
      }
    }
  }
}
```

## 响应字段

响应包含基本节点标识并为每个节点构建匹配的信息`<nodeId>` 请求参数。下表列出了响应字段。

场地| 描述
:--- |:----
姓名| 节点的名称。
transport_address| 节点的传输地址。
主持人| 节点的主机地址。
IP| 节点的主机IP地址。
版本| 节点的OpenSearch版本。
build_type| 节点的构建类型`rpm`，`docker`，`tar`， ETC。
build_hash| 构建的git提交。
total_indexing_buffer| 字节中的最大堆大小用于保存新索引的文档。一旦超出了堆的大小，文档就会写入磁盘。
角色| 节点角色列表。
属性| 节点的属性。
操作系统| 有关操作系统的信息，包括名称，版本，体系结构，刷新间隔以及可用的处理器和分配的处理器的数量。
过程| 有关当前运行过程的信息，包括PID，刷新间隔和`mlockall`，指定过程地址空间是否已成功锁定在内存中。
JVM| 有关JVM的信息，包括PID，版本，内存信息，垃圾收集器信息和参数。
thread_pool| 有关线程池的信息。
运输| 有关运输地址的信息，包括约束地址，发布地址和个人资料。
http| 有关HTTP地址的信息，包括界限地址，发布地址和最大内容长度，以字节为单位。
插件| 有关已安装插件的信息，包括名称，版本，OpenSearch版本，Java版本，描述，类名称，自定义文件夹名称，扩展插件列表，以及`has_native_controller`，指定插件是否具有本机控制器过程。
模块| 有关模块的信息，包括名称，版本，OpenSearch版本，Java版本，描述，类名称，自定义文件夹名称，扩展插件列表，以及`has_native_controller`，指定插件是否具有本机控制器过程。模块与插件不同，因为将模块自动加载到OpenSearch中，而必须手动安装插件。
摄取| 有关摄入管道和处理器的信息。
聚合| 有关可用聚合类型的信息。


## 所需的权限

如果使用安全插件，请确保拥有适当的权限：`cluster:monitor/nodes/info`。

