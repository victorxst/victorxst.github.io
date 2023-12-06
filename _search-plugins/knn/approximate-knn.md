---
layout: default
title: 近似k-nn搜索
nav_order: 15
parent: k-NN
has_children: false
has_math: true
---

# 近似k-nn搜索

标准k-NN搜索方法使用蛮族计算相似性-力量方法可以测量查询与许多点之间的最接近距离，从而产生确切的结果。这在许多应用程序中都很好。但是，在具有高维度的极大数据集的情况下，这会产生一个缩放问题，从而降低了搜索效率。近似k-NN搜索方法可以通过采用更有效地重组索引并降低可搜索向量的维度来克服这一问题。使用这种方法需要精确牺牲，但可以显着提高搜索处理速度。

大约k-openSearch利用的nn搜索方法使用近似最近的邻居（ANN）算法[nmslib](https://github.com/nmslib/nmslib)，[faiss](https://github.com/facebookresearch/faiss)， 和[露西恩](https://lucene.apache.org/) 电力库的图书馆-nn搜索。这些搜索方法采用ANN来改善大型数据集的搜索延迟。在三种搜索方法中-NN插件提供，此方法为大型数据集提供了最佳的搜索可扩展性。当数据集达到数十万个向量时，这种方法是首选方法。

有关插件当前支持的算法的详细信息，请参见[k-NN索引文档]({{site.url}}{{site.baseurl}}/search-plugins/knn/knn-index#method-definitions)。
{: .note}

k-NN插件为每个KNN构建向量的本地库索引-索引期间矢量场/卢克内段对，可用于有效找到K-搜索过程中最近的邻居与查询矢量。要了解有关Lucene细分市场的更多信息，请参阅[Apache Lucene文档](https://lucene.apache.org/core/8_9_0/core/org/apache/lucene/codecs/lucene87/package-summary.html#package.description)。这些本机库索引在搜索过程中加载到本机内存中，并由缓存管理。要了解有关将本地库索引预加载到内存中的更多信息，请参阅[热身API]({{site.url}}{{site.baseurl}}/search-plugins/knn/api#warmup-operation)。此外，您还可以查看已在内存中加载了哪些本机库索引。要了解有关此的更多信息，请参阅[Stats API部分]({{site.url}}{{site.baseurl}}/search-plugins/knn/api#stats)。

由于本机库索引是在索引期间构建的，因此不可能在索引上应用过滤器，然后使用此搜索方法。所有过滤器都应用于大约最近的邻居搜索产生的结果。

### 有关引擎和群集节点尺寸的建议

三个用于大约k的发动机中的每一个-NN搜索具有其自身的属性，在给定情况下，它比其他搜索更明智。您可以遵循以下一般信息，以帮助确定哪种引擎最能满足您的要求。

通常，NMSLIB在搜索方面都优于Faiss和Lucene。但是，要优化索引吞吐量，Faiss是一个不错的选择。对于相对较小的数据集（最多几百万个向量），Lucene Engine展示了更好的延迟和回忆。同时，与其他引擎相比，索引的大小最小，这使其可以使用较小的AWS实例进行数据节点。

此外，Lucene Engine使用纯Java实现，并且不共享使用平台引擎的任何限制-本地代码经验。但是，一个例外是Lucene Engine的最大尺寸计数为1,024，而其他发动机的最大尺寸为1,024。请参阅以下部分中的示例映射参数，以查看配置的位置。

在考虑群集节点大小时，一种一般方法是首先在整个集群中建立索引的均匀分布。但是，还有其他考虑因素。为了帮助做出这些选择，您可以参考本节中的OpenSearch托管服务指南[尺寸域](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/sizing-domains.html)。

## 大约k开始-nn

使用k-NN插件的近似搜索功能，您必须首先创建一个K-NN索引`index.knn` 设置`true`。此设置告诉插件为索引创建本机库索引。

接下来，您必须添加一个或多个字段`knn_vector` 数据类型。此示例创建一个用两个
`knn_vector` 字段，一个使用`faiss` 另一个使用`nmslib` 字段：

```json
PUT my-knn-index-1
{
  "settings": {
    "index": {
      "knn": true,
      "knn.algo_param.ef_search": 100
    }
  },
  "mappings": {
    "properties": {
        "my_vector1": {
          "type": "knn_vector",
          "dimension": 2,
          "method": {
            "name": "hnsw",
            "space_type": "l2",
            "engine": "nmslib",
            "parameters": {
              "ef_construction": 128,
              "m": 24
            }
          }
        },
        "my_vector2": {
          "type": "knn_vector",
          "dimension": 4,
          "method": {
            "name": "hnsw",
            "space_type": "innerproduct",
            "engine": "faiss",
            "parameters": {
              "ef_construction": 256,
              "m": 48
            }
          }
        }
    }
  }
}
```

在上面的示例中，两者都`knn_vector` 字段是根据方法定义配置的。此外，`knn_vector` 字段也可以从模型中配置。您可以在[KNN_VECTOR数据类型]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/knn-vector/) 部分。

这`knn_vector` 数据类型支持由尺寸映射参数设置的nmslib和Faiss引擎的浮子的向量，该浮点数最高为16,000。Lucene库的最大尺寸计数为1,024。

在OpenSearch中，编解码器处理索引的存储和检索。k-NN插件使用自定义编解码器将矢量数据写入本机库索引，以便基础k-NN搜索库可以阅读。
{: .tip }

创建索引后，您可以向其添加一些数据：

```json
POST _bulk
{ "index": { "_index": "my-knn-index-1", "_id": "1" } }
{ "my_vector1": [1.5, 2.5], "price": 12.2 }
{ "index": { "_index": "my-knn-index-1", "_id": "2" } }
{ "my_vector1": [2.5, 3.5], "price": 7.1 }
{ "index": { "_index": "my-knn-index-1", "_id": "3" } }
{ "my_vector1": [3.5, 4.5], "price": 12.9 }
{ "index": { "_index": "my-knn-index-1", "_id": "4" } }
{ "my_vector1": [5.5, 6.5], "price": 1.2 }
{ "index": { "_index": "my-knn-index-1", "_id": "5" } }
{ "my_vector1": [4.5, 5.5], "price": 3.7 }
{ "index": { "_index": "my-knn-index-1", "_id": "6" } }
{ "my_vector2": [1.5, 5.5, 4.5, 6.4], "price": 10.3 }
{ "index": { "_index": "my-knn-index-1", "_id": "7" } }
{ "my_vector2": [2.5, 3.5, 5.6, 6.7], "price": 5.5 }
{ "index": { "_index": "my-knn-index-1", "_id": "8" } }
{ "my_vector2": [4.5, 5.5, 6.7, 3.7], "price": 4.4 }
{ "index": { "_index": "my-knn-index-1", "_id": "9" } }
{ "my_vector2": [1.5, 5.5, 4.5, 6.4], "price": 8.9 }

```

然后，您可以使用`knn` 查询类型：

```json
GET my-knn-index-1/_search
{
  "size": 2,
  "query": {
    "knn": {
      "my_vector2": {
        "vector": [2, 3, 5, 6],
        "k": 2
      }
    }
  }
}
```

`k` 是邻居的数量，每个图的搜索将返回。您还必须包括`size` 选项，哪个
指示查询实际返回多少结果。插件返回`k` 每个碎片的结果量
（以及每个细分市场）和`size` 整个查询的结果量。该插件支持最大`k` 价值10,000。

### 建造k-来自模型的NN索引

对于我们支持的某些算法，需要对本机库索引进行培训，然后才能使用。培训每个新创建的细分市场的昂贵，因此，我们介绍了一个 *模型 *的概念，该概念用于在段创建过程中初始化本机库索引。一个 *模型 *是通过调用[火车API]({{site.url}}{{site.baseurl}}/search-plugins/knn/api#train-model)，传递培训数据的来源以及模型的方法定义。培训完成后，该模型将被序列化为K-NN模型系统索引。然后，在索引期间，将模型从该索引提取以初始化段。

要训练模型，我们首先需要使用其中的培训数据进行开发索引。培训数据可能来自
任何`knn_vector` 字段具有与要创建的模型的维度相匹配的尺寸。培训数据可能与您要索引或在单独集合中具有的数据相同。让我们创建一个培训指数：

```json
PUT /train-index
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 0
  },
  "mappings": {
    "properties": {
      "train-field": {
        "type": "knn_vector",
        "dimension": 4
      }
    }
  }
}
```

注意`index.knn` 未在索引设置中设置。这样可以确保您不会为此索引创建本地库索引。

您现在可以在索引中添加一些数据：

```json
POST _bulk
{ "index": { "_index": "train-index", "_id": "1" } }
{ "train-field": [1.5, 5.5, 4.5, 6.4]}
{ "index": { "_index": "train-index", "_id": "2" } }
{ "train-field": [2.5, 3.5, 5.6, 6.7]}
{ "index": { "_index": "train-index", "_id": "3" } }
{ "train-field": [4.5, 5.5, 6.7, 3.7]}
{ "index": { "_index": "train-index", "_id": "4" } }
{ "train-field": [1.5, 5.5, 4.5, 6.4]}
```

索引培训指数完成后，我们可以致电火车API：

```json
POST /_plugins/_knn/models/my-model/_train
{
  "training_index": "train-index",
  "training_field": "train-field",
  "dimension": 4,
  "description": "My model description",
  "method": {
    "name": "ivf",
    "engine": "faiss",
    "space_type": "l2",
    "parameters": {
      "nlist": 4,
      "nprobes": 2
    }
  }
}
```

培训工作开始后，火车API将立即返回。要检查其状态，我们可以使用GET模型API：

```json
GET /_plugins/_knn/models/my-model?filter_path=state&pretty
{
  "state": "training"
}
```

一旦模型进入"created" 状态，您可以创建一个索引，该索引将使用此模型来初始化其本地
库索引：

```json
PUT /target-index
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1,
    "index.knn": true
  },
  "mappings": {
    "properties": {
      "target-field": {
        "type": "knn_vector",
        "model_id": "my-model"
      }
    }
  }
}
```

最后，我们可以添加想要搜索的文档中的索引：
```json
POST _bulk
{ "index": { "_index": "target-index", "_id": "1" } }
{ "target-field": [1.5, 5.5, 4.5, 6.4]}
{ "index": { "_index": "target-index", "_id": "2" } }
{ "target-field": [2.5, 3.5, 5.6, 6.7]}
{ "index": { "_index": "target-index", "_id": "3" } }
{ "target-field": [4.5, 5.5, 6.7, 3.7]}
{ "index": { "_index": "target-index", "_id": "4" } }
{ "target-field": [1.5, 5.5, 4.5, 6.4]}
...
```

摄入数据后，可以像其他任何人一样进行搜索`knn_vector` 场地！

### 使用近似k-nn带过滤器

了解与K一起使用过滤器-nn搜索，请参阅[k-NN搜索过滤器]({{site.url}}{{site.baseurl}}/search-plugins/knn/filter-search-knn/)。

## 空间

一个空间对应于用于测量两个点之间距离以确定K的函数-最近的邻居。来自k-nn透视图，较低的分数等同于更接近，更好的结果。这与OpenSearch分数结果的相反，其中更高的分数等于更好的结果。要将距离转换为OpenSearch分数，我们采用1 /（1 +距离）。k-nn插件插件支撑的空间下方。并非每种方法都支持这些空间。一定要结帐[方法文档]({{site.url}}{{site.baseurl}}/search-plugins/knn/knn-index#method-definitions) 为了确保您感兴趣的空间得到支持。

<table>
  <thead样式="text-align: center">
  <tr>
    <th> spaceType </th>
    <th>距离函数（d）</th>
    <th> OpenSearch分数</th>
  </tr>
  </thead>
  <tr>
    <td> l1 </td>
    <td> \ [D（\ Mathbf {X}，\ Mathbf {y}）= \ sum_ {i = 1}^n|x_i- 义| \] </td>
    <td> \ [score = {1 \ over 1 + d} \] </td>
  </tr>
  <tr>
    <td> l2 </td>
    <td> \ [D（\ Mathbf {x}，\ Mathbf {y}）= \ sum_ {i = 1}^n（x_i- y_i）^2 \] </td>
    <td> \ [score = {1 \ over 1 + d} \] </td>
  </tr>
  <tr>
    <td> linf </td>
    <td> \ [D（\ Mathbf {x}，\ Mathbf {y}）= max（|x_i- 义|）\] </td>
    <td> \ [score = {1 \ over 1 + d} \] </td>
  </tr>
  <tr>
    <td> cosinesimil </td>
    <td> \ [D（\ Mathbf {X}，\ Mathbf {y}）= 1- cos {\ theta} = 1- {\ Mathbf {x}＆middot;\ mathbf {y} \ over \|\ Mathbf {X} \| ＆middot;\ \|\ Mathbf {y} \|} \] \ [= 1- 
    {\ sum_ {i = 1}^n x_i y_i \ over \ sqrt {\ sum_ {i = 1}^n x_i^2}＆middot;\ sqrt {\ sum_ {i = 1}^n y_i^2}}} \]
    在哪里 \（\|\ Mathbf {X} \|\） 和 \（\|\ Mathbf {y} \|\）分别表示向量x和y的规范。</td>
    <td> <b> nmslib </b> and <b> faiss：</b> \ [score = {1 \ over 1 + d} \] <br> <b> <b> lucene：</b> \ [得分= {2- d \ over 2} \] </td>
  </tr>
  <tr>
    <td> Interproduct（不支持Lucene）</td>
    <td> \ [D（\ Mathbf {X}，\ Mathbf {y}）=- {\ Mathbf {x}＆middot;\ Mathbf {y}} =- \ sum_ {i = 1}^n x_i y_i \] </td>
    <td> \ [\ text {if} d \ ge 0，\] \ [score = {1 \ over 1 + d} \] \ [\ text {if text {if} d <0，score =＆minus; d + 1 \] </td>
  </tr>
</table>

余弦的相似性公式不包括`1 -` 字首。但是，因为相似性搜索库等于
得分较小，结果更接近，他们返回`1 - cosineSimilarity` 余弦相似空间---这就是为什么`1 -` 是
包括在距离函数中。
{: .note}

