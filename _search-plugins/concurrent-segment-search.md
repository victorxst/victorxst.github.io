---
layout: default
title: 并发段搜索
nav_order: 53
---

# 并发段搜索

这是一个实验特征，不建议在生产环境中使用。有关功能进度或要留下反馈的更新，请参阅关联[Github问题](https://github.com/opensearch-project/OpenSearch/issues/2587) 或者[项目委员会](https://github.com/orgs/opensearch-project/projects/117/views/1)。
{： 。警告}

在查询阶段使用并发段搜索并行搜索段。并发搜索改善搜索延迟的情况包括：

- 发送长时间-例如，运行请求，例如包含聚合或大范围的请求
- 作为武力的替代-将细分市场合并为一个细分市场，以提高性能

## 背景

在OpenSearch中，每个搜索请求都遵循分散-收集协议。协调节点接收搜索请求，评估需要哪些碎片来服务此请求，然后发送碎片-对每个碎片的级别搜索请求。接收该请求的每个碎片使用Lucene在本地执行请求并返回结果。协调节点合并了从所有碎片中收到的响应，并将搜索响应发送回客户端。可选的是，如果响应的一部分，客户端请求任何文档字段或整个文档，则可以在将最终结果退还给客户端之前执行提取阶段。

## 同时搜索段

如果没有并发段搜索，Lucene在查询阶段在每个碎片上的所有片段中依次执行请求。然后，查询阶段为搜索请求收集最高点击。通过并发段搜索，每个碎片-级别请求将在查询阶段并行搜索段。对于每个碎片，将段分为多个_slices_。每个切片都是可以在单独线程上并行执行的工作单位，因此切片计数确定碎片的最大并行度-级别请求。一旦所有切片完成工作，Lucene都会在切片上进行减少操作，将它们合并并创建此碎片的最终结果-级别请求。切片是使用新的`index_searcher` 线池，这与`search` 处理碎片的线池-级别请求。

## 启用功能标志

有几种可以启用并发段搜索的方法，具体取决于安装类型。

### 在OpenSearch.yml中启用

如果您正在运行OpenSearch集群并想要在配置文件中启用并发段搜索，请将以下行添加到`opensearch.yml`：

```yaml
opensearch.experimental.feature.concurrent_segment_search.enabled: true
```
{％include copy.html％}

### 使用Docker容器启用

如果您正在运行Docker，请将以下行添加到`docker-compose.yml` 在下面`opensearch-node` >`environment` 部分：

```bash
OPENSEARCH_JAVA_OPTS="-Dopensearch.experimental.feature.concurrent_segment_search.enabled=true"
```
{％include copy.html％}

### 使用TARBALL安装在节点上启用

要在Tarball安装上启用并发段搜索，请在此中提供新的JVM参数`config/jvm.options` 或者`OPENSEARCH_JAVA_OPTS`。

#### 选项1：修改JVM.Options

将以下行添加到`config/jvm.options` 在开始`opensearch` 使功能及其依赖性的过程：

```bash
-Dopensearch.experimental.feature.concurrent_segment_search.enabled=true
```
{％include copy.html％}

然后运行OpenSearch：

```bash
./bin/opensearch
```
{％include copy.html％}

#### 选项2：使用环境变量启用

作为直接修改的替代方案`config/jvm.options`，您可以使用环境变量来定义属性。当您启动openSearch或通过使用`export`。

要在启动OpenSearch时添加这些标志，请运行以下命令：

```bash
OPENSEARCH_JAVA_OPTS="-Dopensearch.experimental.feature.concurrent_segment_search.enabled=true" ./opensearch-{{site.opensearch_version}}/bin/opensearch
```
{％include copy.html％}

如果要在运行OpenSearch之前单独定义环境变量，请运行以下命令：

```bash
export OPENSEARCH_JAVA_OPTS="-Dopensearch.experimental.feature.concurrent_segment_search.enabled=true"
```
{％include copy.html％}

```bash
./bin/opensearch
```
{％include copy.html％}

## 在索引或群集级别禁用并发搜索

启用实验功能标志后，所有搜索请求将在查询阶段使用并发段搜索。要禁用所有索引的并发段搜索，请设置以下动态群集设置：

```json
PUT _cluster/settings
{
   "persistent":{
      "search.concurrent_segment_search.enabled": false
   }
}
```
{％包含副本-curl.html％}

要禁用并发段搜索特定索引，请在端点中指定索引名称：

```json
PUT <index-name>/_settings
{
    "index.search.concurrent_segment_search.enabled": false
}
```
{％包含副本-curl.html％}

## 切片机制

您可以选择将段分配给切片的两种可用机制之一：默认[Lucene机制](#the-lucene-mechanism) 或者[最大切片计数机制](#the-max-slice-count-mechanism)。

### Lucene机制

默认情况下，Lucene最多将250K文档或5个片段（以先到者为准）分配给碎片中的每个切片。例如，考虑一个带有11个片段的碎片。前5个段每个都有250k文档，接下来的6个部分每个都有20K文档。前5个段将分配给1个切片，因为它们每个包含切片允许的最大文档数量。然后，由于切片的最大允许段计数，接下来的5个片段将全部分配给另一个单个切片。第11片将分配给单独的切片。

### 最大切片计数机制

_max slice count_机制是一种替代切片机制，它使用静态配置的最大切片数，并将片段之间的切片分配-罗宾时尚。当已经太多顶部时，这很有用-级别的碎片请求，您想限制每个请求的切片数，以减少切片之间的竞争。

### 设置切片机制

默认情况下，并发段搜索使用Lucene机制来计算每个碎片的切片数-级别请求。要使用最大切片计数机制，请配置`search.concurrent.max_slice_count` 静态设置`opensearch.yml` 配置文件：

```yaml
search.concurrent.max_slice_count: 2
```
{％include copy.html％}

这`search.concurrent.max_slice_count` 设置可以采用以下有效值：
- `0`：使用默认的Lucene机制。
- 正整数：使用最大目标切片计数机制。通常，2至8之间的值就足够了。

## 这`terminate_after` 搜索参数

这[`terminate_after` 搜索参数]({{site.url}}{{site.baseurl}}/api-reference/search/#url-parameters) 一旦收集了指定数量的文档，用于终止搜索请求。如果您包括`terminate_after` 请求中的参数，并发段搜索被禁用，并在非-并发方式。

通常，查询与较小`terminate_after` 值并因此快速完成，因为搜索是在还原的数据集上执行的。因此，在这种情况下，并发搜索可能无法进一步提高性能。而且，什么时候`terminate_after` 与其他搜索请求参数一起使用，例如`track_total_hits` 或者`size`，它增加了复杂性并改变了预期的查询行为。回到非-搜索请求的并发路径包括`terminate_after` 确保并发和非-并发请求。

## API改变

如果启用并发段搜索功能标志，则以下统计信息API响应将包含其他有关SLICES统计信息的其他字段：

- [索引统计]({{site.url}}{{site.baseurl}}/api-reference/index-apis/stats/)
- [节点统计]({{site.url}}{{site.baseurl}}/api-reference/nodes-apis/nodes-stats/)

有关添加字段的描述，请参阅[索引统计API]({{site.url}}{{site.baseurl}}/api-reference/index-apis/stats#concurrent-segment-search)。

另外，有些[配置文件API]({{site.url}}{{site.baseurl}}/api-reference/profile/) 响应字段将被修改并添加其他响应字段。有关更多信息，请参阅[配置文件API的并发段搜索部分]({{site.url}}{{site.baseurl}}/api-reference/profile#concurrent-segment-search)。

## 限制

父母的聚合[加入]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/join/) 字段不支持并发搜索模型。因此，如果搜索请求包含父的聚合，则将使用非-即使在集群级别启用了并发段搜索，并发路径也是如此。

## 开发人员信息：聚合范围更改

由于实现详细信息，并非所有聚合器类型都可以支持并发段搜索。为了适应这个，我们引入了[`supportsConcurrentSegmentSearch()`](https://github.com/opensearch-project/OpenSearch/blob/bb38ed4836496ac70258c2472668325a012ea3ed/server/src/main/java/org/opensearch/search/aggregations/AggregatorFactory.java#L121) 方法中的方法`AggregatorFactory` 班级指示给定的聚合类型是否支持并发段搜索。默认情况下，此方法返回`false`。任何需要支持并发段搜索的聚合器都必须在其自己的工厂实施中覆盖此方法。

确保自定义插件-基于`Aggregator` 实现可与并发搜索路径一起使用，插件开发人员可以通过启用并发搜索验证其实现，然后更新插件以覆盖[`supportsConcurrentSegmentSearch()`](https://github.com/opensearch-project/OpenSearch/blob/bb38ed4836496ac70258c2472668325a012ea3ed/server/src/main/java/org/opensearch/search/aggregations/AggregatorFactory.java#L121) 返回的方法`true`。

