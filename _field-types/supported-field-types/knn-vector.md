---
layout: default
title: k-NN 矢量
nav_order: 58
has_children: false
grand_parent: 支持的字段类型
has_math: true
---

# k-nn矢量

这[k-NN插件]({{site.url}}{{site.baseurl}}/search-plugins/knn/index/) 引入自定义数据类型，`knn_vector`，这使用户可以摄取其K-nn向量成opensearch索引并执行不同种类的k-nn搜索。这`knn_vector` 字段是高度可配置的，可以为许多不同的K服务-NN工作负载。通常，`knn_vector` 可以通过提供方法定义或指定模型ID来构建字段。

## 例子

例如，映射`my_vector1` 作为一个`knn_vector`，使用以下请求：

```json
PUT test-index
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
        "dimension": 3,
        "method": {
          "name": "hnsw",
          "space_type": "l2",
          "engine": "lucene",
          "parameters": {
            "ef_construction": 128,
            "m": 24
          }
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

## 方法定义

[方法定义]({{site.url}}{{site.baseurl}}/search-plugins/knn/knn-index#method-definitions) 在基础时使用[近似k-nn]({{site.url}}{{site.baseurl}}/search-plugins/knn/approximate-knn/) 算法不需要培训。例如，以下`knn_vector` 字段指定 *nmslib *的实现 *hnsw *应用于近似k-nn搜索。在索引期间， * nmslib *将构建相应的 * HNSW *段文件。

```json
"my_vector": {
  "type": "knn_vector",
  "dimension": 4,
  "method": {
    "name": "hnsw",
    "space_type": "l2",
    "engine": "nmslib",
    "parameters": {
      "ef_construction": 128,
      "m": 24
    }
  }
}
```

## 型号ID

当基础近似k时使用模型ID-NN算法需要训练步骤。作为先决条件，
必须使用[火车API]({{site.url}}{{site.baseurl}}/search-plugins/knn/api#train-model)。这
模型包含初始化本机库段文件所需的信息。

```json
  "type": "knn_vector",
  "model_id": "my-model"
}
```

但是，如果您打算使用无痛的脚本或K-nn得分脚本，您只需要通过维度即可。
 ```json
   "type": "knn_vector",
   "dimension": 128
 }
 ```

## Lucene Byte Vector

默认情况下，k-nn矢量是`float` 向量，每个维度为4个字节。如果要节省存储空间，则可以使用`byte` 与`lucene` 引擎。在一个`byte` 向量，每个维度是签名的8-在[-128，127]范围。
 
字节向量仅支持`lucene` 引擎。他们不支持`nmslib` 和`faiss` 引擎。
{: .note}}

在[k-NN基准测试](https://github.com/opensearch-project/k-NN/tree/main/benchmarks/perf-tool)， 指某东西的用途`byte` 而不是`float` 向量导致存储和内存使用情况显着减少，并改善了索引吞吐量和减少查询延迟。此外，召回精度没有受到很大的影响（请注意，召回可能取决于各种因素，例如[量化技术](#quantization-techniques) 和数据分布）。

使用时`byte` 向量，与使用相比，召回召回中有一定的精度丧失`float` 向量。字节向量在大的-比例应用程序和用例，优先减少记忆足迹，以换取最小的召回损失。
{： 。重要的}
 
在k中引入-NN插件2.9版，可选`data_type` 参数定义向量的数据类型。此参数的默认值为`float`。

使用一个`byte` 向量，设置`data_type` 参数为`byte` 在为索引创建映射时：

 ```JSON
放测试-指数
{
  "settings"：{
    "index"：{
      "knn"： true，
      "knn.algo_param.ef_search"：100
    }
  }，，
  "mappings"：{
    "properties"：{
      "my_vector1"：{
        "type"："knn_vector"，
        "dimension"：3，
        "data_type"："byte"，
        "method"：{
          "name"："hnsw"，
          "space_type"："l2"，
          "engine"："lucene"，
          "parameters"：{
            "ef_construction"：128，
            "m"：24
          }
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

Then ingest documents as usual. Make sure each dimension in the vector is in the supported [-128, 127] range:

```json
放测试-索引/_doc/1
{
  "my_vector1"：[[-126、28、127]
}
```
{% include copy-curl.html %}

```json
放测试-索引/_doc/2
{
  "my_vector1"：[100，-128，0]
}
```
{% include copy-curl.html %}

When querying, be sure to use a `byte` vector:

```json
进行测试-索引/_Search
{
  "size"：2，
  "query"：{
    "knn"：{
      "my_vector1"：{
        "vector"：[[26，-120，99]，
        "k"：2
      }
    }
  }
}
```
{% include copy-curl.html %}

### Quantization techniques

If your vectors are of the type `float`, you need to first convert them to the `byte` type before ingesting the documents. This conversion is accomplished by _quantizing the dataset_---reducing the precision of its vectors. There are many quantization techniques, such as scalar quantization or product quantization (PQ), which is used in the Faiss engine. The choice of quantization technique depends on the type of data you're using and can affect the accuracy of recall values. The following sections describe the scalar quantization algorithms that were used to quantize the [k-NN benchmarking test](https://github.com/opensearch-project/k-NN/tree/main/benchmarks/perf-tool) data for the [L2](#scalar-quantization-for-the-l2-space-type) and [cosine similarity](#scalar-quantization-for-the-cosine-similarity-space-type) space types. The provided pseudocode is for illustration purposes only.

#### Scalar quantization for the L2 space type

The following example pseudocode illustrates the scalar quantization technique used for the benchmarking tests on Euclidean datasets with the L2 space type. Euclidean distance is shift invariant. If you shift both $$x$$ and $$y$$ by the same $$z$$, then the distance remains the same ($$\lVert x-y\rVert =\lVert (x-z)-(y-z)\rVert$$).

```python
# 随机数据集（创建一个随机数据集的示例）
dataset = np.random.riform（-300，300，（100，10））
# 随机查询集（示例创建一个随机QuerySet）
querySet = np.random.均匀（-350，350，（100，10））
# 值数
B = 256

# 索引：
# 获得最小和最大
dataset_min = np.min（数据集）
dataset_max = np.max（数据集）
# 将坐标转换为非-消极的
数据集-= dataset_min
# 标准化为[0，1]
数据集 *= 1. /（dataset_max- dataset_min）
# 存储到256个值
dataset = np.floor（数据集 *（b- 1））- int（b / 2）

# 查询：
# 剪辑（如果QuerySet范围不超出数据集范围）
querySet = queryset.clip（dataset_min，dataset_max）
# 将坐标转换为非-消极的
queryset-= dataset_min
# 归一化
queryset *= 1. /（dataset_max- dataset_min）
# 存储到256个值
querySet = np.floor（querySet *（b- 1））- int（b / 2）
```
{% include copy.html %}

#### Scalar quantization for the cosine similarity space type

The following example pseudocode illustrates the scalar quantization technique used for the benchmarking tests on angular datasets with the cosine similarity space type. Cosine similarity is not shift invariant ($$cos(x, y) \neq cos(x-z, y-z)$$). 

The following pseudocode is for positive numbers:

```python
# 用于正数

# 索引和查询：

# 获取最大火车数据集
max = np.max(dataset)
min = 0
B = 127

# 标准化为[0,1]
val = (val - min) / (max - min)
val = (val * B)

# 获取int和分数值
int_part = floor(val)
frac_part = val - int_part

if 0.5 < frac_part:
 bval = int_part + 1
else:
 bval = int_part

return Byte(bval)
```
{% include copy.html %}

The following pseudocode is for negative numbers:

```python
# 对于负数

# 索引和查询：

# 获取火车数据集的最小
min = 0
max = -np.min(dataset)
B = 128

# 标准化为[0,1]
val = (val - min) / (max - min)
val = (val * B)

# 获取int和分数值
int_part = floor(var)
frac_part = val - int_part

if 0.5 < frac_part:
 bval = int_part + 1
else:
 bval = int_part

return Byte(bval)
```
{% include copy.html %}
