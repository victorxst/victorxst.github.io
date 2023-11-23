---
layout: default
title: 索引设置
parent: 配置 OpenSearch
nav_order: 70
redirect_from:
  - /im-plugin/index-settings/
---

# 索引设置

索引设置可以有两种类型：[群集级别设置](#cluster-level-index-settings)影响集群中的所有索引和[索引级设置](#index-level-index-settings)影响单个索引的索引。

要了解有关静态和动态设置的详细信息，请参阅[配置 OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)。

## 集群级索引设置

OpenSearch 支持以下集群级索引设置。此列表中的所有设置都是动态的：

-  `action.auto_create_index`（Boolean）：如果索引尚不存在，则自动创建索引。还应用配置的任何索引模板。缺省值为 `true`。

-  `action.destructive_requires_name`（Boolean）：设置为 `true` 时，必须指定索引名称才能删除索引。不能删除所有索引或使用通配符。缺省值为 `true`。

-  `cluster.default.index.refresh_interval`（时间单位）：设置未提供设置时 `index.refresh_interval` 的刷新间隔。当你希望在群集中的所有索引中设置默认刷新间隔并支持该设置时， `searchIdle` 此设置非常有用。不能将间隔设置为低于设置 `cluster.minimum.index.refresh_interval`。

-  `cluster.minimum.index.refresh_interval`（时间单位）：设置最小刷新间隔，并将其应用于集群中的所有索引。该 `cluster.default.index.refresh_interval` 设置应高于此设置的值。如果在索引创建期间， `index.refresh_interval` 该设置低于最小设置，则索引创建将失败。

-  `cluster.indices.close.enable`（Boolean）：允许关闭 OpenSearch 中的打开索引。缺省值为 `true`。

-  `indices.recovery.max_bytes_per_sec`（字符串）：限制每个节点的入站和出站恢复流量总量。这适用于对等恢复和快照恢复。缺省值为 `40mb`。如果将恢复流量值设置为小于或等于 `0mb`，则将禁用速率限制，这会导致恢复数据以尽可能高的速率传输。

-  `indices.recovery.max_concurrent_file_chunks`（整数）：每个恢复操作并行发送的文件块数。缺省值为 `2`。

-  `indices.recovery.max_concurrent_operations`（整数）：每次恢复并行发送的操作数。缺省值为 `1`。

-  `indices.recovery.max_concurrent_remote_store_streams`（整数）：恢复远程存储索引时可以并行打开的远程存储库流数。缺省值为 `20`。

-  `indices.time_series_index.default_index_merge_policy`（字符串）：此设置允许你为时序索引指定默认合并策略，特别是对于具有 `@timestamp` 字段的索引，例如数据流。两个可用选项是 `tiered`（默认）和 `log_byte_size`。建议使用 `log_byte_size` for time-series 索引来增强字段 `@timestamp` 范围查询的性能。若要基于每个索引覆盖合并策略，可以使用 `index.merge.policy` 索引设置。

-  `indices.fielddata.cache.size`（String）：字段数据缓存的最大大小。可以指定为绝对值（例如，）或节点堆的百分比（例如， `8GB` `50%`）。此值是静态的，因此必须在文件中指定它 `opensearch.yml`。如果未指定此设置，则最大大小不受限制。此值应小于 `indices.breaker.fielddata.limit`.有关详细信息，请参阅[现场数据断路器]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/circuit-breaker/#field-data-circuit-breaker-settings)。

## 索引级索引设置

你可以在创建索引时指定索引设置。有两种类型的索引设置：

- [静态索引级索引设置](#static-index-level-index-settings)是在索引打开时无法更新的设置。若要更新静态设置，必须关闭索引，更新设置，然后重新打开索引。
- [动态索引级索引设置](#dynamic-index-level-index-settings)是你可以随时更新的设置。

### 创建索引时指定设置

创建索引时，可以按如下方式指定其静态或动态设置：

```json
PUT /testindex
{
  "settings": {
    "index.number_of_shards": 1,
    "index.number_of_replicas": 2
  }
}
```
{% include copy-curl.html %}

### 静态索引级索引设置

OpenSearch 支持以下静态索引级索引设置：

-  `index.number_of_shards`（Integer）：索引中主分片的数量。默认值为 1。

-  `index.number_of_routing_shards`（整数）：用于拆分索引的路由分片数。

-  `index.shard.check_on_startup`（Boolean）：是否应检查索引的分片是否损坏。可用选项包括 `false`（不检查损坏）、 `checksum`（检查物理损坏）和（检查物理和 `true` 逻辑损坏）。缺省值为 `false`。

-  `index.codec`（字符串）：确定索引的存储字段如何压缩并存储在磁盘上。此设置会影响索引分片的大小和索引操作的性能。
    
    有效值为：
        
    -  `default`
    -  `best_compression`
    -  `zstd`（OpenSearch 2.9 及更高版本）
    -  `zstd_no_dict`（OpenSearch 2.9 及更高版本）
        
    对于 `zstd` 和 `zstd_no_dict`，可以在设置中 `index.codec.compression_level` 指定压缩级别。有关详细信息，请参阅[索引编解码器设置]({{site.url}}{{site.baseurl}}/im-plugin/index-codecs/)。自选。缺省值为 `default`。

-  `index.codec.compression_level`（整数）：压缩级别设置提供了压缩比和速度之间的权衡。压缩级别越高，压缩率越高（存储大小越小），但速度越快（压缩和解压缩速度越慢，索引和搜索延迟越大）。仅当 `index.codec` 在 OpenSearch 2.9 及更高版本中设置为 `zstd` 和 `zstd_no_dict` 压缩级别时，才能指定。有效值为 [1，6] 范围内的整数。有关详细信息，请参阅[索引编解码器设置]({{site.url}}{{site.baseurl}}/im-plugin/index-codecs/)。自选。默认值为 3。

-  `index.routing_partition_size`（整数）：自定义路由值可以转到的分片数。路由通过将值重新定位到分片子集而不是单个分片来帮助不平衡的集群。要启用路由，请将此值设置为大于 1 但小于 `index.number_of_shards`。默认值为 1。

-  `index.soft_deletes.retention_lease.period`（时间单位）：保留分片操作历史记录的最长时间。缺省值为 `12h`。

-  `index.load_fixed_bitset_filters_eagerly`（布尔值）：OpenSearch 是否应预加载缓存的筛选条件。可用选项包括 `true` 和 `false`。缺省值为 `true`。

-  `index.hidden`（Boolean）：是否应隐藏索引。隐藏索引不会作为具有通配符的查询的一部分返回。可用选项包括 `true` 和 `false`。缺省值为 `false`。

-  `index.merge.policy`（字符串）：此设置控制 Lucene 段的合并策略。可用选项为 `tiered` 和 `log_byte_size`。默认值为 `tiered`，但对于时序数据（例如日志事件），我们建议你使用 `log_byte_size` 合并策略，该策略可以提高对 `@timestamp` 字段进行范围查询时的查询性能。建议你不要更改现有索引的合并策略。相反，请在创建新索引时配置此设置。

### 更新静态索引设置

你只能在已关闭的索引上更新静态索引设置。下面的示例演示如何更新索引编解码器设置。

首先，关闭索引：

```json
POST /testindex/_close
```
{% include copy-curl.html %}

然后，通过向 `_settings` 终结点发送请求来更新设置：

```json
PUT /testindex/_settings
{
  "index": {
    "codec": "zstd_no_dict",
    "codec.compression_level": 3
  }
}
```
{% include copy-curl.html %}

最后，重新打开索引以启用读取和写入操作：

```json
POST /testindex/_open
```
{% include copy-curl.html %}

有关更新设置（包括支持的查询参数）的详细信息，请参阅[更新设置]({{site.url}}{{site.baseurl}}/api-reference/index-apis/update-settings/)。

### 动态索引级索引设置

OpenSearch 支持以下动态索引级索引设置：

-  `index.number_of_replicas`（整数）：每个主分片应具有的副本分片数。例如，如果你有 4 个主分片并设置为 `index.number_of_replicas` 3，则索引有 12 个副本分片。默认值为 1。

-  `index.auto_expand_replicas`（String）：集群是否根据数据节点数自动添加副本分片。指定下限和上限（例如，0--9）或 `all` 上限。例如，如果你有 5 个数据节点并设置为 `index.auto_expand_replicas` 0--3，则集群不会自动添加另一个副本分片。但是，如果将此值设置为 `0-all` 并再添加 2 个节点，总共 7 个节点，则集群将扩展到现在有 6 个副本分片。默认值为禁用。

-  `index.search.idle.after`（时间单位）：分片在空闲之前应等待搜索或获取请求的时间。缺省值为 `30s`。

-  `index.refresh_interval`（时间单位）：索引应多久刷新一次，这会发布其最新更改并使其可供搜索。可以设置为 `-1` 禁用刷新。缺省值为 `1s`。

-  `index.max_result_window`（整数）：索引搜索的最大值 `from` + `size`。是要从中搜索的起始索引，并且是 `size` 要返回的结果数。 `from` 默认值为 10000。

-  `index.max_inner_result_window`（整数）：+ `size` 的最大值 `from`，指定返回的嵌套搜索命中数和查询期间聚合的最相关文档数。是要从中搜索的起始索引，并且是 `size` 要返回的热门命中数。 `from` 默认值为 100。

-  `index.max_rescore_window`（Integer）：对索引的重新评分请求的最大值 `window_size`。重新评分请求对索引的文档进行重新排序，并返回一个新分数，该分数可能更精确。默认值与默认值相同 `index.max_inner_result_window` 或 10000。

-  `index.max_docvalue_fields_search`（整数）：查询中允许的最大数量 `docvalue_fields`。默认值为 100。

-  `index.max_script_fields`（整数）：查询中允许的最大数量 `script_fields`。默认值为 32。

-  `index.max_ngram_diff`（整数）：和 `max_gram` `NGramTokenizer` `NGramTokenFilter` 之间的最大差 `min_gram` 值。默认值为 1。

-  `index.max_shingle_diff`（整数）：输入 `shingle` 令牌筛选器的和 `min_shingle_size` 之间的 `max_shingle_size` 最大差值。默认值为 3。

-  `index.max_refresh_listeners`（整数）：每个分片允许拥有的最大刷新侦听器数。

-  `index.analyze.max_token_count`（Integer）：API 操作可返回 `_analyze` 的最大令牌数。默认值为 10000。

-  `index.highlight.max_analyzed_offset`（整数）：突出显示请求可以分析的字符数。默认值为 1000000。

-  `index.max_terms_count`（整数）：术语查询可以接受的最大术语数。默认值为 65536。

-  `index.max_regex_length`（整数）：正则表达式查询中可以包含的正则表达式的最大字符长度。默认值为 1000。

-  `index.query.default_field`（列表）：OpenSearch 在查询中使用的字段或字段列表，以防参数中未指定字段。

-  `index.routing.allocation.enable`（String）：指定索引分片分配的选项。可用选项包括 `all`（允许分配所有分片）、（仅允许分配主分片）、 `primaries` `new_primaries`（仅允许分配新的主分片）和 `none`（不允许分配）。缺省值为 `all`。

-  `index.routing.rebalance.enable`（String）：为索引启用分片重新平衡。可用选项包括 `all`（允许对所有分片进行重新平衡）、（仅允许对主分片进行重新平衡）、 `replicas` `primaries`（仅允许对副本进行重新平衡）和 `none`（不允许重新平衡）。缺省值为 `all`。

-  `index.gc_deletes`（时间单位）：保留已删除文档版本号的时间量。缺省值为 `60s`。

-  `index.default_pipeline`（String）：索引的默认引入节点管道。如果设置了默认管道，但该管道不存在，则索引请求将失败。管道名称 `_none` 指定索引没有引入管道。

-  `index.final_pipeline`（String）：索引的最终引入节点管道。如果设置了最终管道，但该管道不存在，则索引请求将失败。管道名称 `_none` 指定索引没有引入管道。

### 更新动态索引设置

你可以随时通过 API 更新动态索引设置。例如，若要更新刷新间隔，请使用以下请求：

```json
PUT /testindex/_settings
{
  "index": {
    "refresh_interval": "2s"
  }
}
```
{% include copy-curl.html %}

有关更新设置（包括支持的查询参数）的详细信息，请参阅[更新设置]({{site.url}}{{site.baseurl}}/api-reference/index-apis/update-settings/)。
