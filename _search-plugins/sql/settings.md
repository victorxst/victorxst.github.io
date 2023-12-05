---
layout: default
title: 设置
parent: SQL和PPL
nav_order: 77
redirect_from:
  - /search-plugins/sql/settings/
---

# SQL设置

SQL插件将一些设置添加到标准OpenSearch集群设置中。大多数是动态的，因此您可以在不重新启动群集的情况下更改插件的默认行为。要了解有关静态和动态设置的更多信息，请参阅[配置OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)。

可以独立禁用处理`PPL` 或者`SQL` 查询。

您可以像其他任何集群设置一样更新这些设置：

```json
PUT _cluster/settings
{
  "transient" : {
    "plugins.sql.enabled" : false
  }
}
```

另外，您可以使用以下请求格式：

```json
PUT _cluster/settings
{
  "transient": {
    "plugins": {
      "ppl": {
        "enabled": "false"
      }
    }
  }
}
```

同样，您可以通过将请求发送到`_plugins/_query/settings` 端点：

```json
PUT _plugins/_query/settings
{
  "transient" : {
    "plugins.sql.enabled" : false
  }
}
```

另外，您可以使用以下请求格式：

```json
PUT _plugins/_query/settings
{
  "transient": {
    "plugins": {
      "ppl": {
        "enabled": "false"
      }
    }
  }
}
```

请求`_plugins/_ppl` 和`_plugins/_sql` 端点在请求主体中包含索引名称，因此它们具有与`bulk`，`mget`， 和`msearch` 运营。设置`rest.action.multi.allow_explicit_index` 参数为`false` 禁用两者`SQL` 和`PPL` 端点。
{: .note}

## 可用设置

环境| 默认| 描述
:--- | :--- | :---
`plugins.sql.enabled` | 真的| 改成`false` 禁用`SQL` 在插件中支持。
`plugins.ppl.enabled` | 真的| 改成`false` 禁用`PPL` 在插件中支持。
`plugins.sql.slowlog` | 2秒| 为缓慢查询配置时间限制（以秒为单位）。插件将慢速查询记录为`Slow query: elapsed=xxx (ms)` 在`opensearch.log`。
`plugins.sql.cursor.keep_alive` | 1分钟| 配置光标上下文的打开时间。光标上下文是资源-密集型，因此我们建议低价值。
`plugins.query.memory_limit` | 85％| 为查询引擎的断路器配置堆内存使用限制。
`plugins.query.size_limit` | 200| 设置查询引擎从OpenSearch获取的索引的默认大小。

## 火花连接器设置

SQL插件支持[Apache Spark](https://spark.apache.org/) 作为增强的计算源。当数据源定义为Apache Spark中的表时，OpenSearch可以使用这些表。这使您可以针对OpenSearch Dashboard的外部来源运行SQL查询[发现]({{site.url}}{{site.baseurl}}/dashboards/discover/index-discover/) 和可观察性日志。

首先，启用以下设置将SPARK添加为数据源并启用正确的权限。

环境| 描述
:--- | :---
`spark.uri` | 火花数据源的标识符。
`spark.auth.type` | 用于身份验证的授权类型。
`spark.auth.username` | 您的火花数据源的用户名。
`spark.auth.password` | 火花数据源的密码。
`spark.datasource.flint.host` | 火花数据源的主机。默认为`localhost`。
`spark.datasource.flint.port` | 火花的端口号。默认为`9200`。
`spark.datasource.flint.scheme` | 火花查询中使用的数据方案。有效值是`http` 和`https`。
`spark.datasource.flint.auth` | 访问火花数据源所需的授权。有效值是`false` 和`sigv4`。
`spark.datasource.flint.region` | 您的OpenSearch集群所在的AWS区域。仅在何时使用`auth` 被设定为`sigv4`。默认值是`us-west-2``.
`spark.datasource.flint.write.id_name` | 火花连接器写入的索引的名称。
`spark.datasource.flint.ignore.id_column` | 不包括`id` 列在查询中导出数据时。默认为`true`。
`spark.datasource.flint.write.batch_size` | 写入火花时设置批处理大小-连接索引。默认为`1000`。
`spark.datasource.flint.write.refresh_policy` | 在连接器未能将数据写入OpenSearch时，设置Spark连接的刷新策略。不要刷新（`false`），立即刷新（`true`），或等待的时间，`wait_for: X`。默认值是`false`。
`spark.datasource.flint.read.scroll_size` | 设置使用Spark运行的查询返回的结果数。默认为`100`。
`spark.flint.optimizer.enabled` | 使OpenSearch可以针对Spark连接进行优化。默认为`true`。
`spark.flint.index.hybridscan.enabled` | 启用OpenSearch可以扫描对非非-来自数据源的分区设备。默认为`false`。

配置后，您可以使用以下API调用测试Spark连接：

```json
POST /_plugins/_ppl
content-type: application/json

{
   "query": "source = my_spark.sql('select * from alb_logs')"
}
```

