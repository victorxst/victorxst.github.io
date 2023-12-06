---
layout: default
title: 索引编解码器
nav_order: 3
parent: Index settings
---

# 索引编解码器

索引编解码器确定索引的存储字段如何压缩和存储在磁盘上。索引编解码器由指定压缩算法的静态 `index.codec` 设置控制。该设置会影响索引分片大小和索引操作性能。

## 支持的编解码器

OpenSearch 支持四种可用于压缩存储字段的编解码器。每个编解码器在压缩率（存储大小）和索引性能（速度）之间提供不同的权衡：

*  `default` -- 此编解码器使用[LZ4 算法](https://en.wikipedia.org/wiki/LZ4_(compression_algorithm)）和预设字典，该字典优先考虑性能而不是压缩率。与此相比， `best_compression` 它提供了更快的索引和搜索操作，但可能会导致更大的索引/分片大小。如果索引设置中未提供编解码器，则使用 LZ4 作为默认压缩算法。
*  `best_compression` -- 此编解码器用作[zlib](https://en.wikipedia.org/wiki/Zlib)压缩的底层算法。它实现了高压缩比，从而减小了索引大小。但是，这可能会在索引操作期间产生额外的 CPU 使用率，并可能导致索引和搜索延迟过高。

从 OpenSearch 2.9 开始，提供了两个基于的新[Zstandard 压缩算法](https://github.com/facebook/zstd)编解码器。该算法在压缩比和速度之间提供了良好的平衡。

更改现有索引的编解码器设置可能具有挑战性（请参阅[更改索引编解码器](#changing-an-index-codec)），因此在使用新的编解码器设置之前，在非生产环境中测试具有代表性的工作负载非常重要。{：.important}

*  `zstd`（OpenSearch 2.9 及更高版本）-- 与编解码器相比 `default`， `best_compression` 此编解码器提供与编解码器相当的显著压缩，具有合理的 CPU 使用率以及改进的索引和搜索性能。
*  `zstd_no_dict`（OpenSearch 2.9 及更高版本）-- 此编解码器类似于 `zstd` 字典压缩功能，但不包括字典压缩功能。与以稍大的索引大小为代价 `zstd` 相比，它提供了更快的索引和搜索操作。

从 OpenSearch 2.10 开始， `zstd` 和 `zstd_no_dict` 压缩编解码器不能用于[k-NN]({{site.url}}{{site.baseurl}}/search-plugins/knn/index/)或[安全分析]({{site.url}}{{site.baseurl}}/security-analytics/index/)索引。{: .warning}

 `zstd` 对于和 `zstd_no_dict` 编解码器，你可以选择在 `index.codec.compression_level` 设置中指定压缩级别。此设置采用 [1，6] 范围内的整数。压缩级别越高，压缩率越高（存储大小越小），但速度越快（压缩和解压缩速度越慢，索引和搜索延迟越大）。

创建索引段时，它使用当前索引编解码器进行压缩。如果更新索引编解码器，则更新后创建的任何段都将使用新的压缩算法。有关具体操作注意事项，请参见[索引操作的索引编解码器注意事项](#index-codec-considerations-for-index-operations)。{:.note}

## 选择编解码器

索引编解码器的选择会影响存储索引数据所需的磁盘空间量。编解码器（如 `best_compression`、 `zstd`） `zstd_no_dict` 可以实现更高的压缩率，从而产生更小的索引大小。相反，编 `default` 解码器不优先考虑压缩率，导致索引大小更大，但搜索操作速度比 `best_compression` 更快。

## 索引操作的索引编解码器注意事项

以下索引编解码器注意事项适用于各种索引操作。

### 写

每个索引都由分片组成，每个分片都进一步划分为 Lucene 段。在索引写入期间，将根据索引设置中指定的编解码器创建新段。如果更新索引的编解码器，则新段将使用新的编解码器算法。

### 合并

在分段合并期间，OpenSearch 将较小的索引分段合并为较大的分段，以提供最佳资源利用率并提高性能。索引编解码器设置会影响合并操作的速度和效率。索引上发生的合并次数是段大小的一个因素，段大小越小，合并大小越小。如果更新设置 `index.codec`，则在创建合并区段时，新的合并操作将使用新的编解码器。合并的段将具有新编解码器的压缩特性。

### 拆分和收缩

将[Split API]({{site.url}}{{site.baseurl}}/api-reference/index-apis/split/)原始索引拆分为一个新索引，其中每个原始主分片被划分为两个或多个主分片。将[Shrink API]({{site.url}}{{site.baseurl}}/api-reference/index-apis/shrink-index/)现有索引收缩为具有较少主分片的新索引。作为拆分或收缩操作的一部分，任何新创建的区段都将使用最新的编解码器设置。

### 快照

创建时，索引编解码器设置会影响快照的大小和创建[snapshot]({{site.url}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/snapshots/index/)快照所需的时间。如果更新了索引的编解码器，则新创建的快照将使用最新的编解码器设置。生成的快照大小将反映最新编解码器设置的压缩特征。快照中包含的现有区段将保留其原始压缩特征。

将索引从一个集群的快照还原到另一个集群时，请务必验证目标集群是否支持源快照中分段的编解码器。例如，如果源快照包含或 `zstd_no_dict` 编解码器的分段 `zstd`（在 OpenSearch 2.9 中引入），你将无法将快照还原到在较旧的 OpenSearch 版本上运行的集群，因为它不支持这些编解码器。

### 重新索引

从源索引执行[reindex]({{site.url}}{{site.baseurl}}/im-plugin/reindex-data/)操作时，在目标索引中创建的新段将具有目标索引的编解码器设置的属性。

### 索引汇总和转换

索引[rollup]({{site.url}}{{site.baseurl}}/im-plugin/index-rollups/)或[transform]({{site.url}}{{site.baseurl}}/im-plugin/index-transforms/)作业完成后，在目标索引中创建的段将具有在创建目标索引期间指定的索引编解码器的属性，而不考虑源索引编解码器。如果目标索引是通过汇总作业动态创建的，则默认编解码器将用于目标索引的段。

## 更改索引编解码器

无法更改打开索引的编解码器设置。你可以关闭索引，应用新的索引编解码器设置，然后重新打开索引，此时将仅使用新编解码器写入新段。这需要在短时间内停止对索引的所有读取和写入，以更改编解码器，并可能导致段大小和压缩率不一致。或者，你可以使用不同的编解码器设置将源索引中的所有数据重新索引到新索引中，尽管这是一项非常耗费资源的操作。

## 性能调优和基准测试

根据你的特定使用案例，你可能需要尝试不同的索引编解码器设置来微调 OpenSearch 集群的性能。使用不同的编解码器执行基准测试并测量对索引速度、搜索性能和资源利用率的影响，可以帮助你确定工作负载的最佳索引编解码器设置。 `zstd` 使用和 `zstd_no_dict` 编解码器，你还可以微调压缩级别，以确定集群的最佳配置。

### 标杆

下表提供了、 `zstd` 和 `zstd_no_dict` 编解码器与编 `default` 解码器的性能比较 `best_compression`。使用 [ `nyc_taxi`]（https://github.com/topics/nyc-taxi-dataset）数据集进行测试。结果以百分比变化列出，粗体结果表示性能改进。

| | `best_compression`| `zstd` | `zstd_no_dict` |
|：--- |：--- |：--- |：--- | |**写** | | | | 中值延迟 |0% |0% | − 1% | |p90 延迟 |3% |2% |**&minus;5%**	| | 吞吐量 | − 2% |**7%**	|**14%**	| |**读**	| | | | 中值延迟 |0% |1% |0% | |p90 延迟 |1% |1% |**&minus;2%**	| |**磁盘**	| | |
|压缩比|**&minus;34%**	|**&minus;35%**	|**&minus;30%**	|

