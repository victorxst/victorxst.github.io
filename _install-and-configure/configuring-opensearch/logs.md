---
layout: default
title: 日志
parent: 配置 OpenSearch
nav_order: 120
redirect_from:
  - /opensearch/logs/
  - /monitoring-your-cluster/logs/
---

# 原木

OpenSearch 日志包含用于监控集群操作和解决问题的宝贵信息。日志的位置因安装类型而异：

- 在 Docker 上，OpenSearch 将大部分日志写入控制台，并将其余 `opensearch/logs/` 日志存储在.tarball 安装还使用 `opensearch/logs/`.
- 在大多数 Linux 安装中，OpenSearch 会将日志 `/var/log/opensearch/` 写入.

日志以（纯文本）和 `.json` 文件的形式 `.log` 提供。默认情况下，OpenSearch 日志的权限是 `-rw-r--r--` 默认的，这意味着节点上的任何用户账户都可以读取它们。你可以使用该 `filePermissions` 选项更改 `log4j2.properties` 此行为_对于每种日志类型_。例如，你可以添加 `appender.rolling.filePermissions = rw-r-----` to 更改 JSON 服务器日志的权限。有关详细信息，请参见。[Log4j 2 文档](https://logging.apache.org/log4j/2.x/manual/appenders.html#RollingFileAppender)


## 应用程序日志

对于其应用程序日志，OpenSearch 使用[Apache Log4j 2](https://logging.apache.org/log4j/2.x/)及其内置日志级别（从最不严重到最严重）。日志记录设置说明如下表所示。

| Setting | Data type |描述|
| :--- | :--- | :--- |
| `logger.org.opensearch.discovery` | String |记录器接受 Log4j2 的内置日志级别： `OFF`、、、 `WARN` `ERROR`、 `DEBUG` `FATAL` `INFO` `TRACE`、和。缺省值为 `INFO`。|

你无需更改默认日志级别（ `logger.level`），而是更改各个 OpenSearch 模块的日志级别：

```json
PUT /_cluster/settings
{
  "persistent" : {
    "logger.org.opensearch.index.reindex" : "DEBUG"
  }
}
```
{% include copy-curl.html %}

识别模块的最简单方法不是从日志中识别，日志缩写路径（例如， `o.o.i.r`），而是从[OpenSearch 源代码](https://github.com/opensearch-project/opensearch/tree/master/server/src/main/java/org/opensearch).{: .tip}

在此示例更改之后，OpenSearch 会在重新索引操作期间发出更详细的日志：

```
[2019-10-18T16:52:51,184][DEBUG][o.o.i.r.TransportReindexAction] [node1] [1626]: starting
[2019-10-18T16:52:51,186][DEBUG][o.o.i.r.TransportReindexAction] [node1] executing initial scroll against [some-index]
[2019-10-18T16:52:51,291][DEBUG][o.o.i.r.TransportReindexAction] [node1] scroll returned [3] documents with a scroll id of [DXF1Z==]
[2019-10-18T16:52:51,292][DEBUG][o.o.i.r.TransportReindexAction] [node1] [1626]: got scroll response with [3] hits
[2019-10-18T16:52:51,294][DEBUG][o.o.i.r.WorkerBulkByScrollTaskState] [node1] [1626]: preparing bulk request for [0s]
[2019-10-18T16:52:51,297][DEBUG][o.o.i.r.TransportReindexAction] [node1] [1626]: preparing bulk request
[2019-10-18T16:52:51,299][DEBUG][o.o.i.r.TransportReindexAction] [node1] [1626]: sending [3] entry, [222b] bulk request
[2019-10-18T16:52:51,310][INFO ][o.e.c.m.MetaDataMappingService] [node1] [some-new-index/R-j3adc6QTmEAEb-eAie9g] create_mapping [_doc]
[2019-10-18T16:52:51,383][DEBUG][o.o.i.r.TransportReindexAction] [node1] [1626]: got scroll response with [0] hits
[2019-10-18T16:52:51,384][DEBUG][o.o.i.r.WorkerBulkByScrollTaskState] [node1] [1626]: preparing bulk request for [0s]
[2019-10-18T16:52:51,385][DEBUG][o.o.i.r.TransportReindexAction] [node1] [1626]: preparing bulk request
[2019-10-18T16:52:51,386][DEBUG][o.o.i.r.TransportReindexAction] [node1] [1626]: finishing without any catastrophic failures
[2019-10-18T16:52:51,395][DEBUG][o.o.i.r.TransportReindexAction] [node1] Freed [1] contexts
```

DEBUG 和 TRACE 级别非常详细。如果启用其中任何一个来解决问题，请在完成后将其禁用。

还有其他方法可以更改日志级别：

1. 将行添加到 `opensearch.yml`：

   ```yml
   logger.org.opensearch.index.reindex: debug
   ```
   {% include copy.html %}

   如果要在多个集群中重用日志记录配置或调试单个节点的启动问题，则修改 `opensearch.yml` 最有意义。

2. 修改 `log4j2.properties`：

   ```properties
   # Define a new logger with unique ID of reindex
   logger.reindex.name = org.opensearch.index.reindex
   # Set the log level for that ID
   logger.reindex.level = debug
   ```
   {% include copy.html %}

   这种方法非常灵活，但需要熟悉[Log4j 2 属性文件语法](https://logging.apache.org/log4j/2.x/manual/configuration.html#Properties).通常，其他选项提供更简单的配置体验。

   如果你检查配置目录中的默认 `log4j2.properties` 文件，你可以看到一些特定于 OpenSearch 的变量：

   ```properties
   appender.console.layout.pattern = [%d{ISO8601}][%-5p][%-25c{1.}] [%node_name]%marker %m%n
   appender.rolling_old.fileName = ${sys:opensearch.logs.base_path}${sys:file.separator}${sys:opensearch.logs.cluster_name}.log
   ```

   -  `${sys:opensearch.logs.base_path}` 是日志的目录（例如， `/var/log/opensearch/`）。
   -  `${sys:opensearch.logs.cluster_name}` 是群集的名称。
   -  `[%node_name]` 是节点的名称。


## 慢日志

OpenSearch 有两个*慢日志*日志可帮助你识别性能问题：搜索慢速日志和索引慢速日志。

这些日志依靠阈值来定义什么是“慢速”搜索或“慢速”索引操作。例如，如果查询完成时间超过 15 秒，则可以认为查询速度较慢。与为模块配置的应用程序日志不同，你可以为索引配置慢速日志。默认情况下，两个日志都处于禁用状态（所有阈值都设置为 `-1`）：

```json
GET <some-index>/_settings?include_defaults=true
{
  "indexing": {
    "slowlog": {
      "reformat": "true",
      "threshold": {
        "index": {
          "warn": "-1",
          "trace": "-1",
          "debug": "-1",
          "info": "-1"
        }
      },
      "source": "1000",
      "level": "TRACE"
    }
  },
  "search": {
    "slowlog": {
      "level": "TRACE",
      "threshold": {
        "fetch": {
          "warn": "-1",
          "trace": "-1",
          "debug": "-1",
          "info": "-1"
        },
        "query": {
          "warn": "-1",
          "trace": "-1",
          "debug": "-1",
          "info": "-1"
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

若要启用这些日志，请增加一个或多个阈值：

```json
PUT <some-index>/_settings
{
  "indexing": {
    "slowlog": {
      "threshold": {
        "index": {
          "warn": "15s",
          "trace": "750ms",
          "debug": "3s",
          "info": "10s"
        }
      },
      "source": "500",
      "level": "INFO"
    }
  }
}
```
{% include copy-curl.html %}

在此示例中，OpenSearch 记录在 WARN 级别花费 15 秒或更长时间的索引操作以及花费 10 到 14 秒的操作。*x*秒在 INFO 级别。如果你将阈值设置为 0 秒，OpenSearch 将记录所有操作，这对于测试是否确实启用了慢速日志非常有用。

-  `reformat` 指定是将文档 `_source` 字段记录为单行（ `true`）还是让它跨越多行（ `false`）。
-  `source` 是要记录的文档 `_source` 字段的字符数。
-  `level` 是要包含的最小日志级别。

中的 `opensearch_index_indexing_slowlog.log` 一行可能如下所示：

```
node1 | [2019-10-24T19:48:51,012][WARN][i.i.s.index] [node1] [some-index/i86iF5kyTyy-PS8zrdDeAA] took[3.4ms], took_millis[3], type[_doc], id[1], routing[], source[{"title":"Your Name", "Director":"Makoto Shinkai"}]
```

如果阈值或级别设置得太低，慢速日志可能会占用大量磁盘空间。请考虑暂时启用它们以进行故障排除或性能优化。要禁用慢速日志，请将所有阈值返回到 `-1`。

## 任务日志

启用任务资源使用者后，OpenSearch 可以记录内存消耗最大的 N 个搜索任务的 CPU 时间和内存利用率。默认情况下，任务资源使用者将每隔 60 秒记录前 10 个搜索任务。这些值可以在中配置 `opensearch.yml`。

任务日志记录是通过群集设置 API 动态启用的：

```json
PUT _cluster/settings
{
  "persistent" : {
    "task_resource_consumers.enabled" : "true"
  }
}
```
{% include copy-curl.html %}

启用任务资源使用者可能会对搜索延迟产生影响。{: .tip}

启用后，日志将写入 `logs/opensearch_task_detailslog.json` 和 `logs/opensearch_task_detailslog.log`。

若要配置日志记录间隔和记录的搜索任务数，请将以下行添加到 `opensearch.yml`：

```yaml
# Number of expensive search tasks to log
cluster.task.consumers.top_n.size:100

# Logging interval
cluster.task.consumers.top_n.frequency:30s
```
{% include copy.html %}

## 弃用日志

弃用日志记录客户端何时对集群进行已弃用的 API 调用。这些日志可帮助你在升级到新的主要版本之前识别和修复问题。默认情况下，OpenSearch 会在 WARN 级别记录已弃用的 API 调用，这几乎适用于所有使用案例。如果需要，请使用 `_cluster/settings`、 `opensearch.yml` 或 `log4j2.properties` 进行配置 `logger.deprecation.level`。
