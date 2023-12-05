---
layout: default
title: 指标参考
nav_order: 25
has_children: true
parent: OpenSearch基准参考
redirect_from: /benchmark/metrics/index/
---

# 指标

工作量完成后，OpenSearch基准测试将所有度量记录存储在其指标商店中。这些指标可以将其保存在内存中，也可以保存在OpenSearch集群中。

## 存储指标

您可以在运行基准测试时指定指标是存储在内存还是指标存储中[`datastore.type`](https://opensearch.org/docs/latest/benchmark/configuring-benchmark/#results_publishing) 您的参数`benchmark.ini` 文件。

### 在记忆中

如果要在运行基准测试时将指标存储在内存中，请在`results_publishing` 部分`benchmark.ini`：

```ini
[results_publishing]
datastore.type = in-memory
datastore.host = <host-url>
datastore.port = <host-port>
datastore.secure = False
datastore.ssl.verification_mode = <ssl-verification-details>
datastore.user = <username>
datastore.password = <password>
```

### OpenSearch

如果要在运行基准时将指标存储在外部OpenSearch Memory商店中，请在`results_publishing` 部分`benchmark.ini`：

```ini
[results_publishing]
datastore.type = opensearch
datastore.host = <opensearch endpoint>
datastore.port = 443
datastore.secure = true
datastore.ssl.verification_mode = none
datastore.user = <opensearch basic auth username>
datastore.password = <opensearch basic auth password>
datastore.number_of_replicas = 
datastore.number_of_shards = 
```
当两者都不`datastore.number_of_replicas` 也不`datastore.number_of_shards` 已提供，OpenSearch使用默认值：`0` 对于复制品的数量和`1` 用于碎片数。如果在创建数据存储集群之后更改了这些设置，则只有在本月底创建新的结果索引时，新的副本和碎片设置才能适用。

运行OpenSearch基准配置为使用OpenSearch用作数据存储后，OpenSearch基准标准创建了三个索引：

- `benchmark-metrics-YYYY-MM`：持有颗粒公制和遥测数据。
- `benchmark-results-YYYY-MM`：根据最终结果保存数据。
- `benchmark-test-executions-YYYY-MM`：持有有关的数据`execution-ids`。

您可以在OpenSearch仪表板中可视化这些索引中的数据。


## 下一步

- 有关如何设计指标商店的更多信息，请参阅[公制记录]({{site.url}}{{site.baseurl}}/benchmark/metrics/metric-records/)。
- 有关存储哪些指标的更多信息，请参阅[公制键]({{site.url}}{{site.baseurl}}/benchmark/metrics/metric-keys/)。

