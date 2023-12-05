---
layout: default
title: k-nn无痛的扩展
nav_order: 25
parent: k-NN
has_children: false
has_math: true
---

# k-nn无痛的脚本扩展

与K-NN插件的无痛脚本扩展，您可以使用k-NN距离直接在您的无痛脚本中起作用，以执行操作`knn_vector` 字段。无痛的每个上下文都有严格的允许功能和类的清单，以确保其脚本安全。k-NN插件将无痛的脚本扩展添加到使用的一些距离功能[k-NN得分脚本]({{site.url}}{{site.baseurl}}/search-plugins/knn/knn-score-script)，因此您可以使用它们来自定义K-NN工作量。

## 开始K-NN的无痛脚本功能

使用k-NN的无痛脚本功能，首先与`knn_vector` 像在In这样的字段[k-NN得分脚本]({{site.url}}{{site.baseurl}}/search-plugins/knn/knn-score-script#getting-started-with-the-score-script-for-vectors)。创建索引并摄入一些数据后，您可以使用无痛的扩展：

```json
GET my-knn-index-2/_search
{
  "size": 2,
  "query": {
    "script_score": {
      "query": {
        "bool": {
          "filter": {
            "term": {
              "color": "BLUE"
            }
          }
        }
      },
      "script": {
        "source": "1.0 + cosineSimilarity(params.query_value, doc[params.field])",
        "params": {
          "field": "my_vector",
          "query_value": [9.9, 9.9]
        }
      }
    }
  }
}
```

`field` 需要映射到`knn_vector` 字段和`query_value` 需要是一个浮点阵列，其维度与`field`。

## 功能类型
下表描述了可用的无痛功能K-NN插件提供：

功能名称| 功能签名| 描述
:--- | :---
L2平方| `float l2Squared (float[] queryVector, doc['vector field'])` | 此函数计算给定查询向量和文档向量之间的L2距离（欧几里得距离）的平方。距离越短，文档越相关，因此此示例将L2Squared函数的返回值反转。如果文档向量与查询向量匹配，则结果为0，因此此示例还将1添加到距离上，以避免除以零错误。
l1norm| `float l1Norm (float[] queryVector, doc['vector field'])` | 此函数计算给定查询向量和文档向量之间的L2距离（欧几里得距离）的平方。距离越短，文档越相关，因此此示例将L2Squared函数的返回值反转。如果文档向量与查询向量匹配，则结果为0，因此此示例还将1添加到距离上，以避免除以零错误。
余弦| `float cosineSimilarity (float[] queryVector, doc['vector field'])` | 余弦相似性是查询矢量的内部产物，并且将文档向量归一化为两者的长度为1。如果查询向量的幅度在整个查询过程中没有变化，您可以通过查询向量的幅度来提高性能，以提高性能，而不是每次过滤文档每次计算幅度：<br />`float cosineSimilarity (float[] queryVector, doc['vector field'], float normQueryVector)` <br />通常，余弦相似性的范围是[-1，1]。但是，在信息检索的情况下，两个文档的余弦相似性范围为0到1，因为TF-IDF统计量不能为负。因此，K-NN插件添加1.0，以便始终产生正余弦相似性得分。

## 约束

1. 如果文件的`knn_vector` 字段具有与查询不同的尺寸，函数抛出`IllegalArgumentException`。

2. 如果矢量字段没有值，则该函数会引发<code> IllegalstateException </code>。

   您可以通过首先检查文档是否在其字段中有一个值来避免这种情况：

   ```
   "source": "doc[params.field].size() == 0 ? 0 : 1 / (1 + l2Squared(params.query_value, doc[params.field]))",
   ```

   由于得分只能是积极的，因此该脚本对矢量字段的文档对文档进行排名，高于没有矢量的文档。

