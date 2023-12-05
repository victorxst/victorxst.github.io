---
layout: default
title: 远程支持存储
nav_order: 40
has_children: true
parent: 可用性和恢复
redirect_from: 
  - /opensearch/remote/
  - /tuning-your-cluster/availability-and-recovery/remote/
---

# 远程支持存储

引入2.10
{：.label .label-紫色的 }


偏僻的-支持的存储为OpenSearch用户提供了一种新的方法来防止数据丢失，通过自动创建所有索引交易的备份并将其发送到远程存储。为了公开此功能，还必须启用段复制。看[段复制]({{site.url}}{{site.baseurl}}/opensearch/segment-replication/) 有关其他信息。

与远程-后备存储，当写请求降落在主碎片上时，该请求仅在主碎片上向Lucene索引。然后将相应的翻译库上传到远程存储。OpenSearch不会将写入请求发送给复制品，而是执行主要术语验证以确认请求发起人shard仍然是主要碎片。主术语验证可确保代理主要碎片在隔离时会失败，并且不知道集群经理选举新的初级。

作为刷新，齐平和合并流的一部分，在主碎片上创建片段后，将段上传到远程段存储中，然后从同一远程段存储中获得副本。这样可以防止主要碎片必须执行任何写操作。

## 配置遥控器-支持存储

偏僻的-后备存储是集群级别的设置。仅当引导到群集时才能启用它。引导完成后，遥控器-无法启用或禁用后盾存储。这在集群级别提供了耐用性。

与配置的远程群集的通信发生在存储库插件接口中。存储库插件的所有现有实现，例如Azure Blob存储，Google Cloud Storage和Amazon Simple Storage Service（Amazon S3），都与远程兼容-支持存储。

确保在集群中的所有节点上以相同的方式对远程存储设置进行配置。如果没有，则引导程序将失败，其属性与当选的集群管理器节点不同。
{： 。笔记}

启用遥控器-给定群集的后盾存储，将远程存储库详细信息作为节点属性提供`opensearch.yml`，如以下示例所示：

```yml
# Repository name
node.attr.remote_store.segment.repository: my-repo-1
node.attr.remote_store.translog.repository: my-repo-2
node.attr.remote_store.state.repository: my-repo-3

# Segment repository settings
node.attr.remote_store.repository.my-repo-1.type: s3
node.attr.remote_store.repository.my-repo-1.settings.bucket: <Bucket Name 1>
node.attr.remote_store.repository.my-repo-1.settings.base_path: <Bucket Base Path 1>
node.attr.remote_store.repository.my-repo-1.settings.region: us-east-1

# Translog repository settings
node.attr.remote_store.repository.my-repo-2.type: s3
node.attr.remote_store.repository.my-repo-2.settings.bucket: <Bucket Name 2>
node.attr.remote_store.repository.my-repo-2.settings.base_path: <Bucket Base Path 2>
node.attr.remote_store.repository.my-repo-2.settings.region: us-east-1

# Cluster state repository settings
node.attr.remote_store.repository.my-repo-3.type: s3
node.attr.remote_store.repository.my-repo-3.settings.bucket: <Bucket Name 3>
node.attr.remote_store.repository.my-repo-3.settings.base_path: <Bucket Base Path 3>
node.attr.remote_store.repository.my-repo-3.settings.region: us-east-1
```
{％包含副本-curl.html％}

您不必将三个不同的远程存储存储库用于细分，转换和状态。这三家商店都可以共享相同的存储库。

在引导过程中，遥控器-列出的支持存储库`opensearch.yml` 自动注册。在使用`remote_store` 设置，该集群中创建的所有索引都将启动将数据上传到配置的远程存储。

## 相关集群设置

您可以使用以下[集群设置]({{site.url}}{{site.baseurl}}//api-reference/cluster-api/cluster-settings/) 调整如何遥不可及-支持的群集处理每个工作负载。

| 场地| 数据类型| 描述|
| ：--- | ：--- | ：--- |
| cluster.default.index.refresh_interval| 时间单元| 设置刷新间隔`index.refresh_interval` 未提供设置。当您要在集群中的所有索引上设置默认刷新间隔并支持该设置，并且也支持该设置`searchIdle` 环境。您不能将间隔设置为低于`cluster.minimum.index.refresh_interval` 环境。|
| cluster.minimum.index.refresh_interval| 时间单元| 设置最小刷新间隔，并将其应用于集群中的所有索引。这`cluster.default.index.refresh_interval` 设置应高于此设置的值。如果在创建索引期间`index.refresh_interval` 设置低于最小值，索引创建失败。|
| cluster.remote_store.translog.buffer_interval| 时间单元| 执行周期性翻译更新时使用的转换缓冲区间隔的默认值。只有在索引设置时，此设置才有效`index.remote_store.translog.buffer_interval` 不存在。|


## 从备份恢复

要从远程备份恢复索引，例如在节点故障的情况下，请使用以下选项之一：

**仅还原未分配的碎片**

```bash
curl -X POST "https://localhost:9200/_remotestore/_restore" -H 'Content-Type: application/json' -d'
{
  "indices": ["my-index-1", "my-index-2"]
}
'
```

**还原给定索引的所有碎片**

```bash
curl -X POST "https://localhost:9200/_remotestore/_restore?restore_all_shards=true" -ku admin:admin -H 'Content-Type: application/json' -d'
{
  "indices": ["my-index"]
}
'
```

如果启用了安全插件，则用户必须具有`cluster:admin/remotestore/restore` 允许。看[访问控制](/security-plugin/access-control/index/) 有关配置用户权限的信息。
{： 。笔记}

## 潜在用例

您可以使用远程-后备存储至：

- 还原红色簇或索引。
- 恢复所有数据，直到上次确认的写作，无论复制计数如何`index.translog.durability` 被设定为`request`。

## 基准

OpenSearch项目已使用多个工作负载选项运行远程存储[OpenSearch基准测试](https://opensearch.org/docs/latest/benchmark/index/) 工具。本节总结了以下工作负载的基准结果：

- [堆栈溢出](https://github.com/opensearch-project/opensearch-benchmark-workloads/tree/main/so)
- [HTTP日志](https://github.com/opensearch-project/opensearch-benchmark-workloads/tree/main/http_logs)
- [纽约出租车](https://github.com/opensearch-project/opensearch-benchmark-workloads/tree/main/nyc_taxis)，，，，

每个工作负载均可针对多个批量索引客户端配置进行测试，以模拟不同程度的请求并发。

您的结果可能会根据群集拓扑，硬件，碎片计数和合并设置而有所不同。

### 集群，碎片和测试配置

对于这些基准测试，我们使用以下群集，碎片和测试配置：

*节点：三个节点，每个节点都使用数据，摄入和群集管理器角色
*节点实例：Amazon EC2 R6G.xlarge
* OpenSearch基准主机：单个Amazon EC2 M5.2xlarge实例
*碎片配置：三片带一个复制品
* 这`repository-s3` 使用默认的S3设置安装了插件

### 堆栈溢出

下表列出了`so` 远程转换缓冲区间隔的工作量为250毫秒。

||| 8个散装索引客户端（默认）| | | 16个批量索引客户| | | 24个批量索引客户| | |
|---|---|---|---|---| --- | --- | --- | --- | --- | --- |
||| 文档复制| 启用远程| 差异的百分比| 文档复制| 启用远程| 差异的百分比| 文档复制| 启用远程| 差异的百分比|
|索引吞吐量|意思是|29582.5| 40667.4|37.47|31154.9|47862.3|53.63|31777.2|51123.2|60.88|
|索引吞吐量|P50|28915.4|40343.4|39.52|30406.4|47472.5|56.13|30852.1|50547.2|63.84|
|索引延迟|P90|1716.34|1469.5|-14.38|3709.77|2799.82|-24.53|5768.68|3794.13|-34.23|

### HTTP日志

下表列出了`http_logs` 远程翻译缓冲区间隔为200 ms的工作量。

||| 8个散装索引客户端（默认）| | | 16个批量索引客户| | | 24个批量索引客户| | |
|---|---|---|---|---| --- | --- | --- | --- | --- | --- |
||| 文档复制| 启用远程|差异的百分比| 文档复制| 启用远程| 差异的百分比|文档复制| 启用远程| 差异的百分比|
|索引吞吐量|意思是|149062|82198.7|-44.86|134696|148749|10.43|133050|197239|48.24|
|索引吞吐量|P50|148123|81656.1|-44.87|133591|148859|11.43|132872|197455|48.61|
|索引延迟|P90|327.011|610.036|86.55|751.705|669.073|-10.99|1145.19|817.185|-28.64|

### 纽约出租车

下表列出了`http_logs` 远程转换缓冲区间隔的工作量为250毫秒。

||| 8个散装索引客户端（默认）| | | 16个批量索引客户| | | 24个批量索引客户| | |
|---|---|---|---|---| --- | --- | --- | --- | --- | --- |
||| 文档复制| 启用远程|差异的百分比| 文档复制| 启用远程| 差异的百分比|文档复制| 启用远程| 差异的百分比|
|索引吞吐量|意思是|93383.9|94186.1|0.86|91624.8|125770|37.27|93627.7|132006|40.99|
|索引吞吐量|P50|91645.1|93906.7|2.47|89659.8|125443|39.91|91120.3|132166|45.05|
|索引延迟|P90|995.217|1014.01|1.89|2236.33|1750.06|-21.74|3353.45|2472|-26.28|

如结果所示，在索引延迟大于平均远程上传时间的情况下，有一致的收益。当您增加批量索引客户的数量时，遥控器-启用的配置可提供高达60的索引吞吐量收益--65％。有关更详细的结果，请参阅[问题#9790](https://github.com/opensearch-project/OpenSearch/issues/9790)。

## 下一步

跟踪对遥控器的未来增强功能-后备存储，请参阅[问题#10181](https://github.com/opensearch-project/OpenSearch/issues/10181)。


