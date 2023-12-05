---
layout: default
title: k-NN索引
nav_order: 5
parent: k-NN
has_children: false
---

# k-NN索引

k-NN插件引入了自定义数据类型，`knn_vector`，这使用户可以摄取其K-nn向量成opensearch索引并执行不同种类的k-nn搜索。这`knn_vector` 字段是高度可配置的，可以为许多不同的K服务-NN工作负载。有关更多信息，请参阅[k-nn矢量]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/knn-vector/)。

## Lucene Byte Vector

从k开始-NN插件2.9版，您可以使用`byte` 与`lucene` 发动机以减少所需的存储空间。有关更多信息，请参阅[Lucene Byte Vector]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/knn-vector#lucene-byte-vector)。

## 方法定义

方法定义是指近似K的基础配置-您要使用的NN算法。方法定义用于创建一个`knn_vector` 字段（当方法不需要培训时）或[在训练期间创建模型]({{site.url}}{{site.baseurl}}/search-plugins/knn/api#train-model) 然后可以用来[创建一个`knn_vector` 场地]({{site.url}}{{site.baseurl}}/search-plugins/knn/approximate-knn/#building-a-k-nn-index-from-a-model)。

方法定义将始终包含该方法的名称，space_type该方法的构建，引擎
（库）使用，以及参数地图。

映射参数| 必需的| 默认| 可更新| 描述
:--- | :--- | :--- | :--- | :---
`name` | 真的| N/A。| 错误的| 最近的邻居方法的标识符。
`space_type` | 错误的| L2| 错误的| 用于计算向量之间距离的矢量空间。
`engine` | 错误的| nmslib| 错误的| 大约k-NN库用于索引和搜索。可用的图书馆是Faiss，Nmslib和Lucene。
`parameters` | 错误的| 无效的| 错误的| 用于最近邻居方法的参数。

### 支持的NMSLIB方法

方法名称| 需要培训| 支持的空间| 描述
:--- | :--- | :--- | :---
`hnsw` | 错误的| L2，Interproduct，cosineimil，L1，Linf| 分层接近图近似K-nn搜索。有关算法的更多详细信息，请参阅此信息[抽象的](https://arxiv.org/abs/1603.09320)。

#### HNSW参数

参数名称| 必需的| 默认| 可更新| 描述
:--- | :--- | :--- | :--- | :---
`ef_construction` | 错误的| 512| 错误的| K期间使用的动态列表的大小-NN图创建。较高的值导致更准确的图形，但索引速度较慢。
`m` | 错误的| 16| 错误的| 插件为每个新元素创建的双向链接数量。增加和降低该值可能会对记忆消耗产生很大的影响。将此值保持在2到100之间。

对于nmslib， * ef_search *设置在[索引设置](#index-settings)。
{: .note}

### 支持的faiss方法

方法名称| 需要培训| 支持的空间| 描述
:--- | :--- | :--- | :---
`hnsw` | 错误的| L2，内部生产| 分层接近图近似K-nn搜索。
`ivf` | 真的| L2，内部生产| 在基于群集基于聚类的不同存储室分配不同存储库的水桶方法，在搜索过程中，仅搜索了一个子集。

对于HNSW，"innerproduct" 使用PQ时不可用。
{: .note}

#### HNSW参数

参数名称| 必需的| 默认| 可更新| 描述
:--- | :--- | :--- | :--- | :---
`ef_search` | 错误的| 512| 错误的| K期间使用的动态列表的大小-nn搜索。较高的值会导致更准确但较慢的搜索。
`ef_construction` | 错误的| 512| 错误的| K期间使用的动态列表的大小-NN图创建。较高的值导致更准确的图形，但索引速度较慢。
`m` | 错误的| 16| 错误的| 插件为每个新元素创建的双向链接数量。增加和降低该值可能会对记忆消耗产生很大的影响。将此值保持在2到100之间。
`encoder` | 错误的| 平坦的| 错误的| 编码向量的编码定义。编码器可以减少索引的内存足迹，而以搜索准确性为代价。

#### IVF参数

参数名称| 必需的| 默认| 可更新| 描述
:--- | :--- | :--- | :--- | :---
`nlist` | 错误的| 4| 错误的| 将向量划分成的水桶数量。较高的值可能会导致更准确的搜索，但要花费记忆和训练潜伏期。有关选择正确值的更多信息，请参阅[选择索引的指南](https://github.com/facebookresearch/faiss/wiki/Guidelines-to-choose-an-index)。
`nprobes` | 错误的| 1| 错误的| 查询期间要搜索的存储桶数。较高的值会导致更准确但较慢的搜索。
`encoder` | 错误的| 平坦的| 错误的| 编码向量的编码定义。编码器可以减少索引的内存足迹，而以搜索准确性为代价。

有关设置这些参数的更多信息，请参阅[Faiss文档](https://github.com/facebookresearch/faiss/wiki/Faiss-indexes)。

#### IVF培训要求

IVF算法需要训练步骤。要创建使用IVF的索引，您需要使用
[火车API]({{site.url}}{{site.baseurl}}/search-plugins/knn/api#train-model)，传递IVF方法定义。IVF要求至少应有`nlist` 训练
数据点，但这是[建议您使用更多](https://github.com/facebookresearch/faiss/wiki/Guidelines-to-choose-an-index#how-big-is-the-dataset)。
培训数据可以由要摄入的相同数据或单独的数据集组成。

### 支持的Lucene方法

方法名称| 需要培训| 支持的空间| 描述
:--- | :--- | :--- | :---
`hnsw` | 错误的| L2，cosineimil| 分层接近图近似K-nn搜索。

#### HNSW参数

参数名称| 必需的| 默认| 可更新| 描述
:--- | :--- | :--- | :--- | :---
`ef_construction` | 错误的| 512| 错误的| K期间使用的动态列表的大小-NN图创建。较高的值导致图形更准确但索引速度较慢。<br> Lucene Engine使用专有术语"beam_width" 描述此功能，直接对应于"ef_construction"。为了在整个OpenSearch文档中保持一致，我们保留该术语"ef_construction" 标记此参数。
`m` | 错误的| 16| 错误的| 插件为每个新元素创建的双向链接数量。增加和降低该值可能会对记忆消耗产生很大的影响。将此值保持在2到100之间。<br> Lucene Engine使用专有术语"max_connections" 描述此功能，直接对应于"m"。为了在整个OpenSearch文档中保持一致，我们保留该术语"m" 标记此参数。

Lucene HNSW实施忽略`ef_search`  并将其动态设置为"k" 在搜索请求中。因此，无需为`ef_search` 使用Lucene引擎时。
{: .note}

```json
{
    "type": "knn_vector",
    "dimension": 100,
    "method": {
        "name":"hnsw",
        "engine":"lucene",
        "space_type": "l2",
        "parameters":{
            "m":2048,
            "ef_construction": 245
        }
    }
}
```

### 支持的Faiss编码器

您可以使用编码器来减少K的内存足迹-nn索引以牺牲搜索准确性为代价。Faiss拥有
几种编码器类型，但插件当前仅支持 * Flat *和 * PQ *编码。

以下示例方法定义指定`hnsw` 方法和`pq` 编码器：

```json
"method": {
  "name":"hnsw",
  "engine":"faiss",
  "space_type": "l2",
  "parameters":{
    "encoder":{
      "name":"pq",
      "parameters":{
        "code_size": 8,
        "m": 8
      }
    }
  }
}
```

这`hnsw` 方法支持`pq` OpenSearch版本2.10及以后的编码器。这`code_size` a的参数`pq` 用`hnsw` 方法必须是**8**。
{： 。重要的}

编码器名称| 需要培训| 描述
:--- | :--- | :---
`flat` | 错误的| 编码向量作为浮点数组。此编码不会减少内存足迹。
`pq` | 真的| _product量化_的缩写_，它是一种有损的压缩技术，它使用聚类将矢量编码为固定大小的字节，目的是最大程度地减少k中的下降-NN搜索准确性。在高水平上，向量分解为`m` 子向量，然后每个子向量由`code_size` 从培训期间制作的代码簿获得的代码。有关产品量化的更多信息，请参阅[这篇博客文章](https://medium.com/dotstar/understanding-faiss-part-2-79d90b1e5388)。

#### 例子


以下示例使用`ivf` 未指定编码器的方法（默认情况下，OpenSearch使用`flat` 编码器）：

```json
"method": {
  "name":"ivf",
  "engine":"faiss",
  "space_type": "l2",
  "parameters":{
    "nlist": 4,
    "nprobes": 2
  }
}
```

以下示例使用`ivf` 方法`pq` 编码器：

```json
"method": {
  "name":"ivf",
  "engine":"faiss",
  "space_type": "l2",
  "parameters":{
    "encoder":{
      "name":"pq",
      "parameters":{
        "code_size": 8,
        "m": 8
      }
    }
  }
}
```

以下示例使用`hnsw` 未指定编码器的方法（默认情况下，OpenSearch使用`flat` 编码器）：

```json
"method": {
  "name":"hnsw",
  "engine":"faiss",
  "space_type": "l2",
  "parameters":{
    "ef_construction": 256,
    "m": 8
  }
}
```

#### PQ参数

Paramater名称| 必需的| 默认| 可更新| 描述
:--- | :--- | :--- | :--- | :---
`m` | 错误的| 1| 错误的|  确定多少个子-向量将向量分解成。子-向量彼此独立编码。向量的这个维度必须由`m`。最大值为1024。
`code_size` | 错误的| 8| 错误的| 确定编码子编码的位数-向量进入。最大值为8。**笔记** --- 对于IVF，此值必须小于或等于8。对于HNSW，此值只能为8。

### 选择正确的方法

构建您的时有很多选择`knn_vector` 场地。要确定要选择的正确方法和参数，您应该首先了解您对工作量的要求以及什么交易的要求-您愿意做的。要考虑的因素是（1）查询延迟，（2）查询质量，（3）内存限制，（4）索引延迟。

如果不关心内存，HNSW会提供非常强大的查询延迟/查询质量折衷。

如果您想比HNSW更快地使用内存和索引，同时保持相似的查询质量，则应评估IVF。

如果是令人担忧的，请考虑将PQ编码器添加到您的HNSW或IVF索引中。因为PQ是一个有损的编码，因此查询质量将下降。

### 内存估计

在典型的OpenSearch群中，为JVM堆准备了一定部分RAM。k-NN插件分配
本地图书馆索引剩下的RAM的一部分。此部分的大小由
这`circuit_breaker_limit` 集群设置。默认情况下，限制设置为50％。

具有复制品会使向量的总数加倍。
{: .note }

#### HNSW内存估计

HNSW所需的内存估计为`1.1 * (4 * dimension + 8 * M)` 字节/向量。

例如，假设您有100万个向量，尺寸为256，m为16。可估计的内存要求如下：

```
1.1 * (4 * 256 + 8 * 16) * 1,000,000 ~= 1.267 GB
```

#### IVF内存估计

IVF所需的内存估计为`1.1 * (((4 * dimension) * num_vectors) + (4 * nlist * d))` 字节。

例如，假设您有一百万个矢量，尺寸为256，NLIST为128。内存需求可以估计如下：

```
1.1 * (((4 * 256) * 1,000,000) + (4 * 128 * 256))  ~= 1.126 GB

```

## 索引设置

另外，K-NN插件引入了几种可用于配置K的索引设置-NN结构也是如此。

目前，设置中定义的几个参数处于折旧过程中。这些参数应在映射中设置，而不是索引设置。映射中设置的参数将覆盖索引设置中设置的参数。设置映射中的参数允许索引具有多个`knn_vector` 具有不同参数的字段。

环境| 默认| 可更新| 描述
:--- | :--- | :--- | :---
`index.knn` | 错误的| 错误的| 该索引是否应构建本地库索引`knn_vector` 字段。如果设置为false，`knn_vector` 字段将存储在DOC值中，但近似K-NN搜索功能将被禁用。
`index.knn.algo_param.ef_search` | 512| 真的| K期间使用的动态列表的大小-nn搜索。较高的值会导致更准确但较慢的搜索。仅适用于nmslib。
`index.knn.algo_param.ef_construction` | 512| 错误的| 在1.0.0中弃用。使用[映射参数](https://opensearch.org/docs/latest/search-plugins/knn/knn-index/#method-definitions) 改为设置此值。
`index.knn.algo_param.m` | 16| 错误的| 在1.0.0中弃用。使用[映射参数](https://opensearch.org/docs/latest/search-plugins/knn/knn-index/#method-definitions) 改为设置此值。
`index.knn.space_type` | L2| 错误的| 在1.0.0中弃用。使用[映射参数](https://opensearch.org/docs/latest/search-plugins/knn/knn-index/#method-definitions) 改为设置此值。

