---
layout: default
title: 调整索引速度
nav_order: 41
has_children: false
---

# 调整群集以获取索引速度

以下配置显示，当吞吐量的改善约为60％。
运行索引-与外部相比，只有工作量-的-这-盒子体验。工作量没有
合并搜索或其他方案。仅在计算机上运行OpenSearch Server进程，
带有基准客户端托管在不同的节点上。

执行环境由AWS云中的Intel EC2实例（R7iz.2xlarge）组成，
使用的工作负载是OpenSearch基准测试的一部分可用的Stackoverflow数据集。

## Java堆尺寸

较大的Java堆尺寸可用于索引。将Java Min和Max Heap尺寸设置为RAM的50％
大小在EC2实例上显示出更好的索引性能。

## 冲洗翻译阈值

的默认值`flush_threshold_size` 为512 MB。这意味着当翻译达到512 MB时，它会冲洗掉。索引负载的重量决定了翻译的频率。当你增加时`index.translog.flush_threshold_size`，节点执行转换操作的频率较低。因为冲洗是资源-降低翻译频率的密集操作可提高索引性能。通过增加冲洗阈值大小，OpenSearch群集还会产生更少的大段，而不是多个小细分市场。大细分市场合并的频率较低，更多的线程用于索引而不是合并。

对于纯索引工作负载，请考虑增加`flush_threshold_size` 例如，约有25％的Java堆尺寸以提高索引性能。

增加`index.translog.flush_threshold_size` 还可以增加翻译完成所需的时间。如果碎片失败，则恢复需要更多时间，因为翻译较大。
{: .note}

在增加之前`index.translog.flush_threshold_size`，致电以下API操作以获取当前的冲洗操作统计信息：

```json
curl -XPOST "os-endpoint/index-name/_stats/flush?pretty"
```
{% include copy.html %}


更换`os-endpoint` 和`index-name` 带有您的端点和索引名称。

在输出中，请注意冲洗次数和总时间。下面的示例输出表明有124次冲洗，耗时17,690毫秒：

```json
{
     "flush": {
          "total": 124,
          "total_time_in_millis": 17690
     }
}
```

为了增加冲洗阈值大小，请致电以下API操作：

```json
curl -XPUT "os-endpoint/index-name/_settings?pretty" -d "{"index":{"translog.flush_threshold_size" : "1024MB"}}"
```
{% include copy.html %}

在此示例中，齐平阈值大小设置为1024 MB，这是具有超过32 GB内存的实例的理想选择。

为群集选择适当的阈值大小。
{: .note}

再次运行STATS API操作，以查看冲洗活动是否更改：

```json
curl -XGET "os-endpoint/index-name/_stats/flush?pretty"
```
{% include copy.html %}

这是增加的最佳实践`index.translog.flush_threshold_size` 仅针对当前索引。确认结果后，将更改应用于索引模板。
{: .note}

## 索引刷新间隔

默认情况下，OpenSearch刷新每秒索引。OpenSearch仅刷新具有
在过去30秒内至少收到了一个搜索请求。

当您增加刷新间隔时，数据节点会更少的API调用。阻止[429错误](https://repost.aws/knowledge-center/opensearch-resolve-429-error)，这是增加刷新间隔的最佳实践。

如果您的申请可以忍受增加索引索引和何时之间的时间
变得可见，您可以增加`index.refresh_interval` 例如，更大的价值`30s`，甚至禁用它
纯索引方案，以提高索引速度。

## 索引缓冲区大小

如果节点执行重型索引，请确保索引缓冲区大小足够大。您可以将索引缓冲区大小设置为
Java堆尺寸或字节数。在大多数情况下，JVM内存的10％的默认值就足够了。你可以试试
将其提高到25％以进一步改善。

## 并发合并

并发合并的最大数量被指定为`max_merge_count`。并发剂控制器控制执行
在需要时合并操作。合并以不同的线程和最大数量
达到线程，进一步的合并将等到合并线程可用。
如果索引节流是一个问题，请考虑增加合并线程的数量
默认值。

## 碎片分布

为确保碎片在要摄入的索引的数据节点上均匀分布，请使用以下公式确认碎片均匀分布：

索引= k *的碎片数（数据节点的数量），其中k是每个节点的碎片数

例如，如果索引中有24张碎片，并且有8个数据节点，则OpenSearch为每个节点分配了3个碎片。

## 将副本计数设置为零

如果您预计索引重量，请考虑设置`index.number_of_replicas` 价值`0`。每个副本都复制索引过程。结果，禁用复制品可以改善您的群集性能。重量索引完成后，重新激活复制的索引。

如果在禁用副本时节点失败，则可能会丢失数据。仅当您可以在短时间内忍受数据丢失时，才禁用复制品。
{: .important }

## 实验以找到最佳的批量请求尺寸

从散装要求大小为5 MIB到15 MIB开始。然后慢慢增加请求大小，直到索引性能停止改善。

## 使用具有SSD实例存储量的实例类型（例如i3）

i3实例提供快速和本地内存Express（NVME）存储。I3实例比使用通用SSD（GP2）Amazon Elastic Block Store（Amazon EBS）卷的实例提供了更好的摄入性能。有关更多信息，请参阅[Amazon OpenSearch Service的PETABYTE量表](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/petabyte-scale.html)。

## 减少响应尺寸

为了减少OpenSearch响应的大小，请使用`filter_path` 参数排除不必要的字段。确保您不会过滤识别或重试失败的请求所需的任何字段。这些字段可能因客户端而异。

在下面的示例中，`index-name`，`type-name`， 和`took` 田地被排除在响应之外：

```json
curl -XPOST "es-endpoint/index-name/type-name/_bulk?pretty&filter_path=-took,-items.index._index,-items.index._type" -H 'Content-Type: application/json' -d'
{ "index" : { "_index" : "test2", "_id" : "1" } }
{ "user" : "testuser" }
{ "update" : {"_id" : "1", "_index" : "test2"} }
{ "doc" : {"user" : "example"} }
```
{% include copy.html %}

## 压缩编解码器

在OpenSearch 2.9及以后，有两个新的编解码器：`zstd` 和`zstd_no_dict`。您可以选择在`index.codec.compression_level` 设置为值[1，6](range. [Benchmark]({{site.url}}{{site.baseurl}}/im-plugin/index-codecs/#benchmarking) 数据显示`zstd` 提供7％的写入吞吐量和`zstd_no_dict` 提供14％的吞吐量，并提高30％的存储`default` 编解码器。有关压缩的更多信息，请参阅[索引编解码器]({{site.url}}{{site.baseurl}}/im-plugin/index-codecs/)

