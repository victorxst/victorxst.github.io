---
layout: default
title: 段复制 
nav_order: 70
has_children: true
parent: 可用性和恢复
datatable: true
redirect_from:
  - /opensearch/segment-replication/
  - /opensearch/segment-replication/index/
---

# 段复制

细分复制涉及在碎片上复制细分文件，而不是在每个碎片副本上索引文档。这种方法增强了索引吞吐量并降低了资源利用率，但增加了网络利用率。细分复制是一系列功能中的第一个功能，旨在将读取和写入以降低计算成本。

当主碎片将检查点发送到刷新上的复制碎片时，在复制片上触发了一个新的细分复制事件。有时候是这样的：

- 当将新的复制碎片添加到集群中时。
- 当主碎片上有段文件更改时。
- 在同行恢复期间，例如复制碎片恢复和碎片重新定位（使用`move` 分配命令或自动碎片重新平衡）。

## 用例

段复制可以在多种情况下应用，包括：

- 没有高搜索要求且刷新时间更长的高写入负载。
- 当经历非常高的负载时，您需要添加新节点，但不想立即为所有数据索引。
- OpenSearch群集部署具有低复制品计数，例如用于日志分析的群集。

## 偏僻的-支持存储

从OpenSearch 2.10开始，您可以使用两种方法进行细分复制：

- **偏僻的-支持存储**，一个持久的存储解决方案：主碎片将细分文件发送到遥控器-支持的存储空间，复制品碎片从同一商店中获取副本。有关使用远程的更多信息-后备存储，请参阅[偏僻的-支持存储]({{site.url}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/remote-store/index/)。
- 节点-到-节点通信：主碎片使用节点直接将细分文件发送到副本碎片-到-节点通信。

## 段复制配置

设置集群的默认复制类型会影响所有新创建的索引。但是，在创建索引时，您可以指定不同的复制类型。指数-等级设置覆盖集群-级别设置。

### 创建具有段复制的索引

要使用段复制作为索引的复制策略，请与`replication.type` 参数设置为`SEGMENT` 如下：

```json
PUT /my-index1
{
  "settings": {
    "index": {
      "replication.type": "SEGMENT" 
    }
  }
}
```
{% include copy-curl.html %}

如果您正在使用远程-支持存储，添加`remote_store` 属性到索引请求主体。

使用节点时-到-节点复制，主要碎片会消耗更多的网络带宽，因为它将细分文件推向所有复制碎片。因此，在节点之间平均分配初级碎片是有益的。为确保平衡的主碎片分布，设置动态`cluster.routing.allocation.balance.prefer_primary` 设置为`true`。有关更多信息，请参阅[集群设置]({{site.url}}{{site.baseurl}}/api-reference/cluster-api/cluster-settings/)。

为了获得最佳性能，建议您启用以下设置：

1. [段复制背压]({{site.url}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/segment-replication/backpressure/) 
2. 使用以下命令：平衡的主碎片分配：

```json
PUT /_cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.balance.prefer_primary": true,
    "segrep.pressure.enabled": true
  }
}
```
{% include copy-curl.html %}

### 为集群设置复制类型

您可以在新创建的集群索引中设置默认复制类型`opensearch.yml` 文件如下：

```yaml
cluster.indices.replication.strategy: 'SEGMENT'
```
{% include copy.html %}



### 使用文档复制创建索引

即使将默认复制类型设置为段复制，您也可以创建一个索引，该索引通过设置文档复制来设置`replication.type` 到`DOCUMENT` 如下：

```json
PUT /my-index1
{
  "settings": {
    "index": {
      "replication.type": "DOCUMENT" 
    }
  }
}
```
{% include copy-curl.html %}

## 考虑因素

使用片段复制时，请考虑以下内容：

1. 启用现有索引的细分复制需要[重新索引](https://github.com/opensearch-project/OpenSearch/issues/3685)。
1. [叉-集群复制](https://github.com/opensearch-project/OpenSearch/issues/4090) 当前不使用片段复制来在簇之间复制。
1. 段复制导致使用节点上碎片上的网络拥塞增加-到-节点复制是因为副本shards从主碎片中获取更新。与远程-支持存储，主碎片可以上传段，并且复制品可以从远程获取更新-支持存储。这有助于卸载从主要碎片到遥控器的职责-支持存储。
1. 读-后-写保证：细分复制当前不支持将刷新策略设置为`wait_for` 或者`true`。如果您设置`refresh` 查询参数为`wait_for` 或者`true` 然后摄入文档，只有在主节点刷新并使这些文档可以搜索之后，您才能得到响应。复制碎片只有在写信给本地翻译后才会做出回应。如果是真实的-需要时间读取，请考虑使用[`get`]({{site.url}}{{site.baseurl}}/api-reference/document-apis/get-documents/) 或者[`mget`]({{site.url}}{{site.baseurl}}/api-reference/document-apis/multi-get/) API操作。
1. 从OpenSearch 2.10开始，系统索引支持段复制。
1. 通过将请求路由到主要碎片，获取，多列，术语向量和多点矢量请求可提供强大的读取。将更多请求路由到主碎片中可能会降低性能，而不是在主和复制碎片上分发请求。提高阅读的性能-重型集群，我们建议设置`realtime` 这些请求中的参数`false`。有关更多信息，请参阅[问题#8700](https://github.com/opensearch-project/OpenSearch/issues/8700)。

## 基准

在初始基准测试中，细分复制用户报告的吞吐量比使用同一群集设置的文档复制时高40％。

收集以下基准[OpenSearch-基准]({{site.url}}{{site.baseurl}}/benchmark/index/) 使用[`stackoverflow`](https://www.kaggle.com/datasets/stackoverflow/stackoverflow) 和[`nyc_taxi`](https://github.com/topics/nyc-taxi-dataset) 数据集。

基准测试证明了以下配置对片段复制的影响：

- [工作量大小](#increasing-the-workload-size)
- [主要碎片的数量](#increasing-the-number-of-primary-shards)
- [副本的数量](#increasing-the-number-of-replicas)

您的结果可能会根据群集拓扑，使用的硬件，碎片计数和合并设置而有所不同。
{:.note}

### 增加工作量大小

下表列出了基准测试结果`nyc_taxi` 具有以下配置的数据集：

- 10 m5.xlarge数据节点
- 40个主要碎片，每个复制品1个（总计80片）
- 每个节点4个主要碎片和4个复制碎片

<table>
    <th colspan ="2" > </th>
    <th colspan ="3" > 40 GB主碎片，80 GB总计</th>
    <th colspan ="3"> 240 GB主碎片，480 GB总计</th>
    <tr>
        <td></td>
        <td></td>
        <td>文档复制</td>
        <td>段复制</td>
        <td>百分比差</td>
        <td>文档复制</td>
        <td>段复制</td>
        <td>百分比差</td>
    </tr>
    <tr>
        <td>商店大小</td>
        <td></td>
        <td> 85.2781 </td>
        <td> 91.2268 </td>
        <td> n/a </td>
        <td> 515.726 </td>
        <td> 558.039 </td>
        <td> n/a </td>
    </tr>
    <tr>
        <td rowspan ="3">索引吞吐量（每秒请求数）</td>
        <td>最小</td>
        <td> 148,134 </td>
        <td> 185,092 </td>
        <td> 24.95％</td>
        <td> 100,140 </td>
        <td> 168,335 </td>
        <td> 68.10％</td>
    </tr>
    <tr>
        <td class ="td-custom">中值</td>
        <td> 160,110 </td>
        <td> 189,799 </td>
        <td> 18.54％</td>
        <td> 106,642 </td>
        <td> 170,573 </td>
        <td> 59.95％</td>
    </tr>
    <tr>
        <td class ="td-custom">最大</td>
        <td> 175,196 </td>
        <td> 190,757 </td>
        <td> 8.88％</td>
        <td> 108,583 </td>
        <td> 172,507 </td>
        <td> 58.87％</td>
    </tr>
    <tr>
        <td>错误率</td>
        <td></td>
        <td> 0.00％</td>
        <td> 0.00％</td>
        <td> 0.00％</td>
        <td> 0.00％</td>
        <td> 0.00％</td>
        <td> 0.00％</td>
    </tr>
</table>

随着工作负载的大小增加，段复制的好处被放大，因为复制品不需要索引较大的数据集。通常，在所有群集配置中，段复制在资源成本低的吞吐量中会导致更高的吞吐量，而不是复制滞后。

### 增加主要碎片的数量

下表列出了基准测试结果`nyc_taxi` 40和100个主要碎片的数据集。

{::nomarkdown}
<table>
    <th colspan ="2"> </th>
    <th colspan ="3"> 40个主碎片，1个复制</th>
    <th colspan ="3"> 100个主要碎片，1个副本</th>
    <tr>
        <td></td>
        <td></td>
        <td>文档复制</td>
        <td>段复制</td>
        <td>百分比差</td>
        <td>文档复制</td>
        <td>段复制</td>
        <td>百分比差</td>
    </tr>
    <tr>
        <td rowspan ="3">索引吞吐量（每秒请求数）</td>
        <td>最小</td>
        <td> 148,134 </td>
        <td> 185,092 </td>
        <td> 24.95％</td>
        <td> 151,404 </td>
        <td> 167,391 </td>
        <td> 9.55％</td>
    </tr>
    <tr>
        <td class ="td-custom">中值</td>
        <td> 160,110 </td>
        <td> 189,799 </td>
        <td> 18.54％</td>
        <td> 154,796 </td>
        <td> 172,995 </td>
        <td> 10.52％</td>
    </tr>
    <tr>
        <td class ="td-custom">最大</td>
        <td> 175,196 </td>
        <td> 190,757 </td>
        <td> 8.88％</td>
        <td> 166,173 </td>
        <td> 174,655 </td>
        <td> 4.86％</td>
    </tr>
    <tr>
        <td>错误率</td>
        <td></td>
        <td> 0.00％</td>
        <td> 0.00％</td>
        <td> 0.00％</td>
        <td> 0.00％</td>
        <td> 0.00％</td>
        <td> 0.00％</td>
    </tr>
</table>
{:/}

随着主要碎片数量的增加，段复制的好处比文档复制的益处减少了。尽管细分复制仍然有益于大量的主要碎片，但性能差异变得不那么明显，因为每个节点的主碎片更多，必须在整个群集上复制段文件。

### 增加复制品的数量

下表列出了基准测试结果`stackoverflow` 1和9副本的数据集。

{::nomarkdown}
<table>
    <th colspan ="2"  > </th>
    <th colspan ="3"  > 10个主要碎片，1个复制品</th>
    <th colspan ="3"> 10个主要碎片，9个复制品</th>
    <tr>
        <td></td>
        <td></td>
        <td>文档复制</td>
        <td>段复制</td>
        <td>百分比差</td>
        <td>文档复制</td>
        <td>段复制</td>
        <td>百分比差</td>
    </tr>
    <tr>
        <td rowspan ="2">索引吞吐量（每秒请求数）</td>
        <td>中值</td>
        <td> 72,598.10 </td>
        <td> 90,776.10 </td>
        <td> 25.04％</td>
        <td> 16,537.00 </td>
        <td> 14,429.80 </td>
        <td>＆minus; 12.74％</td>
    </tr>
    <tr>
        <td class ="td-custom">最大</td>
        <td> 86,130.80 </td>
        <td> 96,471.00 </td>
        <td> 12.01％</td>
        <td> 21,472.40 </td>
        <td> 38,235.00 </td>
        <td> 78.07％</td>
    </tr>
    <tr>
        <td rowspan ="4"> CPU用法（％）</td>
        <td> p50 </td>
        <td> 17 </td>
        <td> 18.857 </td>
        <td> 10.92％</td>
        <td> 69.857 </td>
        <td> 8.833 </td>
        <td>＆minus; 87.36％</td>
    </tr>
    <tr>
        <td class ="td-custom"> p90 </td>
        <td> 76 </td>
        <td> 82.133 </td>
        <td> 8.07％</td>
        <td> 99 </td>
        <td> 86.4 </td>
        <td>＆minus; 12.73％</td>
    </tr>
    <tr>
        <td class ="td-custom"> p99 </td>
        <td> 100 </td>
        <td> 100 </td>
        <td> 0％</td>
        <td> 100 </td>
        <td> 100 </td>
        <td> 0％</td>
    </tr>
    <tr>
        <td class ="td-custom"> p100 </td>
        <td> 100 </td>
        <td> 100 </td>
        <td> 0％</td>
        <td> 100 </td>
        <td> 100 </td>
        <td> 0％</td>
    </tr>
    <tr>
        <td rowspan ="4">内存使用（％）</td>
        <td> p50 </td>
        <td> 35 </td>
        <td> 23 </td>
        <td>＆minus; 34.29％</td>
        <td> 42 </td>
        <td> 40 </td>
        <td>＆minus; 4.76％</td>
    </tr>
    <tr>
        <td class ="td-custom"> p90 </td>
        <td> 59 </td>
        <td> 57 </td>
        <td>＆minus; 3.39％</td>
        <td> 59 </td>
        <td> 63 </td>
        <td> 6.78％</td>
    </tr>
    <tr>
        <td class ="td-custom"> p99 </td>
        <td> 69 </td>
        <td> 61 </td>
        <td>＆minus; 11.59％</td>
        <td> 66 </td>
        <td> 70 </td>
        <td> 6.06％</td>
    </tr>
    <tr>
        <td class ="td-custom"> p100 </td>
        <td> 72 </td>
        <td> 62 </td>
        <td>＆minus; 13.89％</td>
        <td> 69 </td>
        <td> 72 </td>
        <td> 4.35％</td>
    </tr>
    <tr>
        <td>错误率</td>
        <td></td>
        <td> 0.00％</td>
        <td> 0.00％</td>
        <td> 0.00％</td>
        <td> 0.00％</td>
        <td> 2.30％</td>
        <td> 2.30％</td>
    </tr>
</table>
{:/}

随着副本数量的增加，将shards保持最新的主碎片所需的时间（称为_replication lag_）也增加了。这是因为细分复制将段文件直接从主碎片复制到副本。

基准测试结果表明-零错误率随着副本数量的增加。错误率表明[段复制背压]({{site.urs}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/segment-replication/backpressure/) 当复制品无法跟上主碎片时，就会启动机制。但是，错误率被片段复制提供的显着CPU和内存增益所抵消。

## 下一步

1. 追踪[对细分复制的未来增强功能](https://github.com/orgs/opensearch-project/projects/99)。
1. 读[有关细分复制的博客文章](https://opensearch.org/blog/segment-replication/)。

