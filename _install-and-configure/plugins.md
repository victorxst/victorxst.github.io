---
layout: default
title: 安装插件
nav_order: 90
redirect_from:
   - /opensearch/install/plugins/
   - /install-and-configure/install-opensearch/plugins/
---

# 安装插件

你可以根据需要为 OpenSearch 安装单独的插件。有关可用插件的信息，请参阅[可用的插件](#available-plugins)。


要使插件与 OpenSearch 正常工作，所有插件都必须能够访问集群中的数据，包括有关集群操作的元数据。因此，为了保护集群的数据并保持其完整性，请先确保你了解插件的功能，然后再将其安装在 OpenSearch 集群上。其次，在选择自定义插件时，请确保插件的来源是可靠的。
{: .warning}

## 管理插件

OpenSearch 使用用于管理插件的 `opensearch-plugin` 命令行工具。此工具允许你：

- [List](#list)已安装的插件。
- [Install](#install)插件。
- [Remove](#remove)已安装的插件。

通过传递 `-h` 或 `--help` 打印帮助文本。根据你的主机配置，你可能还需要使用 `sudo` 权限运行该命令。

如果你在 Docker 容器中运行 OpenSearch，则必须通过修改 Docker 映像来安装、删除和配置插件。有关信息，请参阅[使用插件]({{site.url}}{{site.baseurl}}/install-and-configure/install-opensearch/docker#working-with-plugins) 
{: .note}

## 列表

用于 `list` 查看已安装的插件列表。

#### 用法：
```bash
bin/opensearch-plugin list
```

#### 例：
```bash
$ ./opensearch-plugin list
opensearch-alerting
opensearch-anomaly-detection
opensearch-asynchronous-search
opensearch-cross-cluster-replication
opensearch-geospatial
opensearch-index-management
opensearch-job-scheduler
opensearch-knn
opensearch-ml
opensearch-notifications
opensearch-notifications-core
opensearch-observability
opensearch-performance-analyzer
opensearch-reports-scheduler
opensearch-security
opensearch-sql
```

你还可以使用[CAT API]({{site.url}}{{site.baseurl}}/api-reference/cat/cat-plugins/)列出已安装的插件。

#### 路径和 HTTP 方法

```bash
GET _cat/plugins
```

#### 示例响应

```bash
opensearch-node1 opensearch-alerting                  2.0.1.0
opensearch-node1 opensearch-anomaly-detection         2.0.1.0
opensearch-node1 opensearch-asynchronous-search       2.0.1.0
opensearch-node1 opensearch-cross-cluster-replication 2.0.1.0
opensearch-node1 opensearch-index-management          2.0.1.0
opensearch-node1 opensearch-job-scheduler             2.0.1.0
opensearch-node1 opensearch-knn                       2.0.1.0
opensearch-node1 opensearch-ml                        2.0.1.0
opensearch-node1 opensearch-notifications             2.0.1.0
opensearch-node1 opensearch-notifications-core        2.0.1.0
```

## 安装

有三种方法可以使用 `opensearch-plugin`：

- [按名称安装插件]({{site.url}}{{site.baseurl}}/opensearch/install/plugins#install-a-plugin-by-name)
- [从 zip 文件安装插件]({{site.url}}{{site.baseurl}}/opensearch/install/plugins#install-a-plugin-from-a-zip-file)
- [使用 Maven 坐标安装插件]({{site.url}}{{site.baseurl}}/opensearch/install/plugins#install-a-plugin-using-maven-coordinates)

### 按名称安装插件：

有关可按名称安装的插件列表，请参阅[其他插件]({{site.url}}{{site.baseurl}}/opensearch/install/plugins#additional-plugins)。

#### 用法：
```bash
bin/opensearch-plugin install <plugin-name>
```

#### 例：
```bash
$ sudo ./opensearch-plugin install analysis-icu
-> Installing analysis-icu
-> Downloading analysis-icu from opensearch
[=================================================] 100%   
-> Installed analysis-icu with folder name analysis-icu
```

### 从 zip 文件安装插件：

可以通过替换为 `<zip-file>` 托管文件的 URL 来安装远程 zip 文件。该工具仅支持通过 HTTP/HTTPS 协议下载。对于本地 zip 文件，请替换为 `<zip-file>` `file:` 插件 zip 文件的绝对或相对路径，如下面的第二个示例所示。

#### 用法：
```bash
bin/opensearch-plugin install <zip-file>
```

#### 例：
```bash
# Zip file is hosted on a remote server - in this case, Maven central repository.
$ sudo ./opensearch-plugin install https://repo1.maven.org/maven2/org/opensearch/plugin/opensearch-anomaly-detection/2.2.0.0/opensearch-anomaly-detection-2.2.0.0.zip
-> Installing https://repo1.maven.org/maven2/org/opensearch/plugin/opensearch-anomaly-detection/2.2.0.0/opensearch-anomaly-detection-2.2.0.0.zip
-> Downloading https://repo1.maven.org/maven2/org/opensearch/plugin/opensearch-anomaly-detection/2.2.0.0/opensearch-anomaly-detection-2.2.0.0.zip
[=================================================] 100%   
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@     WARNING: plugin requires additional permissions     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
* java.lang.RuntimePermission accessClassInPackage.sun.misc
* java.lang.RuntimePermission accessDeclaredMembers
* java.lang.RuntimePermission getClassLoader
* java.lang.RuntimePermission setContextClassLoader
* java.lang.reflect.ReflectPermission suppressAccessChecks
* java.net.SocketPermission * connect,resolve
* javax.management.MBeanPermission org.apache.commons.pool2.impl.GenericObjectPool#-[org.apache.commons.pool2:name=pool,type=GenericObjectPool] registerMBean
* javax.management.MBeanPermission org.apache.commons.pool2.impl.GenericObjectPool#-[org.apache.commons.pool2:name=pool,type=GenericObjectPool] unregisterMBean
* javax.management.MBeanServerPermission createMBeanServer
* javax.management.MBeanTrustPermission register
See http://docs.oracle.com/javase/8/docs/technotes/guides/security/permissions.html
for descriptions of what these permissions allow and the associated risks.

Continue with installation? [y/N]y
-> Installed opensearch-anomaly-detection with folder name opensearch-anomaly-detection

# Zip file in a local directory.
$ sudo ./opensearch-plugin install file:/home/user/opensearch-anomaly-detection-2.2.0.0.zip
-> Installing file:/home/user/opensearch-anomaly-detection-2.2.0.0.zip
-> Downloading file:/home/user/opensearch-anomaly-detection-2.2.0.0.zip
[=================================================] 100%   
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@     WARNING: plugin requires additional permissions     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
* java.lang.RuntimePermission accessClassInPackage.sun.misc
* java.lang.RuntimePermission accessDeclaredMembers
* java.lang.RuntimePermission getClassLoader
* java.lang.RuntimePermission setContextClassLoader
* java.lang.reflect.ReflectPermission suppressAccessChecks
* java.net.SocketPermission * connect,resolve
* javax.management.MBeanPermission org.apache.commons.pool2.impl.GenericObjectPool#-[org.apache.commons.pool2:name=pool,type=GenericObjectPool] registerMBean
* javax.management.MBeanPermission org.apache.commons.pool2.impl.GenericObjectPool#-[org.apache.commons.pool2:name=pool,type=GenericObjectPool] unregisterMBean
* javax.management.MBeanServerPermission createMBeanServer
* javax.management.MBeanTrustPermission register
See http://docs.oracle.com/javase/8/docs/technotes/guides/security/permissions.html
for descriptions of what these permissions allow and the associated risks.

Continue with installation? [y/N]y
-> Installed opensearch-anomaly-detection with folder name opensearch-anomaly-detection
```

### 使用 Maven 坐标安装插件：

该 `opensearch-plugin install` 工具还接受上托管[Maven 中心](https://search.maven.org/search?q=org.opensearch.plugin)的可用工件和版本的 Maven 坐标。将解析你提供的 Maven 坐标并构造 URL。 `opensearch-plugin` 因此，主机必须能够直接[Maven 中心](https://search.maven.org/search?q=org.opensearch.plugin)连接到。如果将坐标传递到代理或本地存储库，则插件安装将失败。

#### 用法：
```bash
bin/opensearch-plugin install <groupId>:<artifactId>:<version>
```

#### 例：
```bash
$ sudo ./opensearch-plugin install org.opensearch.plugin:opensearch-anomaly-detection:2.2.0.0
-> Installing org.opensearch.plugin:opensearch-anomaly-detection:2.2.0.0
-> Downloading org.opensearch.plugin:opensearch-anomaly-detection:2.2.0.0 from maven central
[=================================================] 100%   
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@     WARNING: plugin requires additional permissions     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
* java.lang.RuntimePermission accessClassInPackage.sun.misc
* java.lang.RuntimePermission accessDeclaredMembers
* java.lang.RuntimePermission getClassLoader
* java.lang.RuntimePermission setContextClassLoader
* java.lang.reflect.ReflectPermission suppressAccessChecks
* java.net.SocketPermission * connect,resolve
* javax.management.MBeanPermission org.apache.commons.pool2.impl.GenericObjectPool#-[org.apache.commons.pool2:name=pool,type=GenericObjectPool] registerMBean
* javax.management.MBeanPermission org.apache.commons.pool2.impl.GenericObjectPool#-[org.apache.commons.pool2:name=pool,type=GenericObjectPool] unregisterMBean
* javax.management.MBeanServerPermission createMBeanServer
* javax.management.MBeanTrustPermission register
See http://docs.oracle.com/javase/8/docs/technotes/guides/security/permissions.html
for descriptions of what these permissions allow and the associated risks.

Continue with installation? [y/N]y
-> Installed opensearch-anomaly-detection with folder name opensearch-anomaly-detection
```

安装插件后重新启动 OpenSearch 节点。
{: .note}

## 删除

你可以使用该 `remove` 选项删除已安装的插件。

#### 用法：
```bash
bin/opensearch-plugin remove <plugin-name>
```

#### 例：
```bash
$ sudo $ ./opensearch-plugin remove opensearch-anomaly-detection
-> removing [opensearch-anomaly-detection]...
```

删除插件后重新启动 OpenSearch 节点。
{: .note}

## 批处理模式

当安装需要默认未包含的额外权限的插件时，插件将提示用户确认所需的权限。要授予所有请求的权限，请使用批处理模式跳过确认提示。

要在安装插件时强制使用批处理模式，请添加 `-b` or `--batch` 选项：
```bash
bin/opensearch-plugin install --batch <plugin-name>
```

## 可用的插件

主要、次要和补丁插件版本必须与 OpenSearch 主要版本、次要版本和补丁版本匹配才能兼容。例如，插件版本 2.3.0.x 仅适用于 OpenSearch 2.3.0. {: .warning}

### 捆绑插件

以下插件与所有 OpenSearch 分配捆绑在一起，但最小分配包除外。

| Plugin Name | Repository |最早的可用版本|
| :--- | :--- | :--- |
| Alerting |[opensearch-alerting（opensearch-alerting）](https://github.com/opensearch-project/alerting)| 1.0.0 |
| Anomaly Detection |[opensearch-异常检测](https://github.com/opensearch-project/anomaly-detection)| 1.0.0 |
| Asynchronous Search |[opensearch-异步搜索](https://github.com/opensearch-project/asynchronous-search)| 1.0.0 |
| Cross Cluster Replication |[opensearch-cross-cluster-replication](https://github.com/opensearch-project/cross-cluster-replication)| 1.1.0 |
| Custom Codecs |[opensearch-custom-codecs](https://github.com/opensearch-project/custom-codecs)| 2.10.0 |
| Notebooks<sup>1</sup> |[opensearch-notebooks](https://github.com/opensearch-project/dashboards-notebooks)| 1.0.0 to 1.1.0 |
| 通知 | [通知](https://github.com/opensearch-project/notifications) | 2.0.0
| Reports Scheduler |[opensearch-reports-scheduler](https://github.com/opensearch-project/dashboards-reports)| 1.0.0 |
| Geospatial |[opensearch-geospatial](https://github.com/opensearch-project/geospatial)| 2.2.0 |
| Index Management |[opensearch-index-management（opensearch-index-management）](https://github.com/opensearch-project/index-management)| 1.0.0 |
| Job Scheduler |[opensearch-job-scheduler](https://github.com/opensearch-project/job-scheduler)| 1.0.0 |
| k-NN |[opensearch-knn](https://github.com/opensearch-project/k-NN)| 1.0.0 |
| ML Commons |[OpenSearch-ML 的](https://github.com/opensearch-project/ml-commons)| 1.3.0 |
| Neural Search |[神经搜索](https://github.com/opensearch-project/neural-search)| 2.4.0 |
| Observability |[opensearch-可观测性](https://github.com/opensearch-project/observability)| 1.2.0 |
| Performance Analyzer<sup>2</sup> |[opensearch-performance-analyzer](https://github.com/opensearch-project/performance-analyzer)| 1.0.0 |
| Security |[opensearch-安全](https://github.com/opensearch-project/security)| 1.0.0 |
| Security Analytics |[opensearch-安全-分析](https://github.com/opensearch-project/security-analytics)| 2.4.0 |
| SQL |[opensearch-sql](https://github.com/opensearch-project/sql)| 1.0.0 |

_<sup>1</sup>Dashboard Notebooks was merged in to the Observability plugin with the release of OpenSearch 1.2.0._<br>
_<sup>2</sup>Performance Analyzer is not available on Windows._


### 其他插件

OpenSearch 社区的成员为该服务构建了无数插件。尽管无法构建每个插件的详尽列表，但由于许多插件未在 OpenSearch GitHub 存储库中维护，因此可以使用以下插件列表按名称 `bin/opensearch-plugin install <plugin-name>` 安装。

| Plugin Name |最早的可用版本|
| :--- | :--- |
| analysis-icu | 1.0.0 |
| analysis-kuromoji | 1.0.0 |
| analysis-nori | 1.0.0 |
| analysis-phonetic | 1.0.0 |
| analysis-smartcn | 1.0.0 |
| analysis-stempel | 1.0.0 |
| analysis-ukrainian | 1.0.0 |
| discovery-azure-classic | 1.0.0 |
| discovery-ec2 | 1.0.0 |
| discovery-gce | 1.0.0 |
| ingest-attachment | 1.0.0 |
| mapper-annotated-text | 1.0.0 |
| mapper-murmur3 | 1.0.0 |
| mapper-size | 1.0.0 |
| repository-azure | 1.0.0 |
| repository-gcs | 1.0.0 |
| repository-hdfs | 1.0.0 |
| repository-s3 | 1.0.0 |
| store-smb | 1.0.0 |
| transport-nio | 1.0.0 |

## 相关链接

- [关于可观测性]({{site.url}}{{site.baseurl}}/observability-plugin/index/)
- [关于安全分析]({{site.url}}{{site.baseurl}}/security-analytics/index/)
- [关于安全插件]({{site.url}}{{site.baseurl}}/security/index/)
- [Alerting]({{site.url}}{{site.baseurl}}/monitoring-plugins/alerting/index/)
- [异常检测]({{site.url}}{{site.baseurl}}/monitoring-plugins/ad/index/)
- [异步搜索]({{site.url}}{{site.baseurl}}/search-plugins/async/index/)
- [跨集群复制]({{site.url}}{{site.baseurl}}/replication-plugin/index/)
- [索引状态管理]({{site.url}}{{site.baseurl}}/im-plugin/ism/index/)
- [k-NN]({{site.url}}{{site.baseurl}}/search-plugins/knn/index/)
- [ML Commons 插件]({{site.url}}{{site.baseurl}}/ml-commons-plugin/index/)
- [神经搜索]({{site.url}}{{site.baseurl}}/neural-search-plugin/index/)
- [通知]({{site.url}}{{site.baseurl}}/notifications-plugin/index/)
- [OpenSearch 控制面板]({{site.url}}{{site.baseurl}}/dashboards/index/)
- [性能分析器]({{site.url}}{{site.baseurl}}/monitoring-plugins/pa/index/)
- [SQL]({{site.url}}{{site.baseurl}}/search-plugins/sql/index/)
