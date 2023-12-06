---
layout: default
title: 安装 OpenSearch
nav_order: 2
has_children: true
redirect_from:
  - /opensearch/install/
  - /opensearch/install/compatibility/
  - /opensearch/install/important-settings/
  - /install-and-configure/index/
  - /opensearch/install/index/
---

# 安装 OpenSearch

本节详细介绍了如何在你的主机上安装 OpenSearch，包括哪些操作系统是[与 OpenSearch 兼容](#operating-system-compatibility)、哪些操作系统以及要在主机上配置哪些[要打开的端口](#network-requirements)[重要设置](#important-settings)操作系统。

## 操作系统兼容性

OpenSearch 和 OpenSearch 控制面板与使用 [ `systemd`]（https://en.wikipedia.org/wiki/Systemd）的 Red Hat Enterprise Linux（RHEL）和基于 Debian 的 Linux 发行版兼容，例如 CentOS、Amazon Linux 2 和 Ubuntu 长期支持（LTS）。虽然 OpenSearch 和 OpenSearch 控制面板应该适用于大多数 Linux 发行版，但我们只测试了一部分。

下表列出了我们当前支持的操作系统版本。

操作系统 | 版本
:---------- | :-------- 
RHEL/CentOS 操作系统 |	7/8
洛基 Linux |	8
Ubuntu 的 |16.04/18.04/20.04
Windows 服务器 |2019


## 文件系统建议

避免在生产工作流中使用网络文件系统进行节点存储。由于网络条件（如延迟或吞吐量有限）或读/写速度等因素，使用网络文件系统进行节点存储可能会导致集群中出现性能问题。应尽可能使用安装在主机上的固态硬盘（SSD）进行节点存储。

## Java 兼容性

适用于 Linux 的 OpenSearch 发行版在目录中附带了兼容[采用 JDK](https://adoptium.net/)的 Java `jdk` 版本。要查找 JDK 版本，请运行 `./jdk/bin/java -version`。例如，OpenSearch 1.0.0 tarball 随 Java 15.0.1+9（非 LTS）一起提供，OpenSearch 1.3.0 随 Java 11.0.14.1+1（LTS）一起提供，OpenSearch 2.0.0 随 Java 17.0.2+8（LTS）一起提供。OpenSearch 已针对所有兼容的 Java 版本进行了测试。

OpenSearch 版本 | 兼容的 Java 版本 | 捆绑的 Java 版本
:---------- | :-------- | :-----------
1.0 - 1.2.x |11、15 |15.0.1+9
1.3. x |8，11，14 |11.0.14.1+1
2.0.0       | 11, 17    | 17.0.2+8

要使用其他 Java 安装，请将 `OPENSEARCH_JAVA_HOME` or `JAVA_HOME` 环境变量设置为 Java 安装位置。例如：
```bash
export OPENSEARCH_JAVA_HOME=/path/to/opensearch-{{site.opensearch_version}}/jdk
```

## 网络要求

需要为 OpenSearch 组件打开以下端口。

端口号 |OpenSearch 组件
:--- | :--- 
443 |AWS OpenSearch Service 中具有传输中加密（TLS）的 OpenSearch 控制面板
5601 |OpenSearch 控制面板
9200 |OpenSearch REST API
9250 | 跨集群搜索
9300 | 节点通信和传输
9600 | 性能分析器

## 重要设置

对于生产工作负载，请确保将设置为[Linux 设置](https://www.kernel.org/doc/Documentation/sysctl/vm.txt) `vm.max_map_count` 至少 262144. 即使使用 Docker 映像，也请在*主机*.若要检查当前值，请运行以下命令：

```bash
cat /proc/sys/vm/max_map_count
```

若要增加该值，请将以下行添加到 `/etc/sysctl.conf`：

```
vm.max_map_count=262144
```

然后运行 `sudo sysctl -p` 重新加载。

该文件[示例 docker-compose.yml]({{site.url}}{{site.baseurl}}/install-and-configure/install-opensearch/docker/#sample-docker-composeyml)还包含几个关键设置：

-  `bootstrap.memory_lock=true`

  禁用交换（以及 `memlock`）。交换会显著降低性能和稳定性，因此应确保在生产群集上禁用交换。

  启用该 `bootstrap.memory_lock` 设置将导致 JVM 保留它需要的任何内存。文档[Java SE 热点 VM 垃圾回收调优指南](https://docs.oracle.com/javase/9/gctuning/other-considerations.htm#JSGCT-GUID-B29C9153-3530-4C15-9154-E74F44E3DAD9)默认的 1 千兆字节（GB）类元数据本机内存预留。结合 Java 堆，这可能会导致错误，因为内存少于这些要求的 VM 上缺少本机内存。为防止错误，请使用 `-XX:CompressedClassSpaceSize` 或 `-XX:MaxMetaspaceSize` 限制保留的内存大小，并设置 Java 堆的大小，以确保你有足够的系统内存。

-  `OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m`

  设置 Java 堆的大小（建议使用一半的系统 RAM）。

-  `nofile 65536`

  为 OpenSearch 用户设置 65536 个打开文件的限制。

-  `port 9600`

  允许你在端口 9600 上访问性能分析器。

请勿在多个位置声明相同的 JVM 选项，因为这可能会导致意外行为或 OpenSearch 服务无法启动。如果使用环境变量（如 `OPENSEARCH_JAVA_OPTS=-Xms3g -Xmx3g`）声明 JVM 选项，那么应注释掉中对该 JVM 选项 `config/jvm.options` 的任何引用。相反，如果在中 `config/jvm.options` 定义 JVM 选项，则不应使用环境变量定义这些 JVM 选项。
{:.note}

## 重要的系统属性

OpenSearch 具有许多系统属性（如下表所示），你可以使用命令行参数表示法或使用 `-D` 命令行参数表示法来指定 `config/jvm.options` `OPENSEARCH_JAVA_OPTS` 这些属性。

物业 | 描述
:---------- | :-------- 
 `opensearch.xcontent.string.length.max=<value>` | 默认情况下，OpenSearch 不会对 JSON 字符串字段的最大长度施加任何限制。为了保护你的集群免受潜在的分布式拒绝服务（DDoS）或内存问题的影响，你可以将系统属性设置为 `opensearch.xcontent.string.length.max` 合理的限制（最大值为 2,147,483,647），例如 `-Dopensearch.xcontent.string.length.max=5000000`。|
 `opensearch.xcontent.fast_double_writer=[true|false]` | 默认情况下，OpenSearch 使用 Java 运行时环境提供的默认实现序列化浮点数。将此值设置为 `true` 使用 Schubfach 算法，该算法速度更快，但可能会导致精度差异很小。默认值为 `false`. |
