---
layout: default
title: 精确k-NN带有评分脚本
nav_order: 10
parent: k-NN
has_children: false
has_math: true
---

# 精确k-NN带有评分脚本

k-NN插件实现了OpenSearch分数脚本插件，您可以使用该插件来找到确切的K-最近的邻居到给定的查询点。使用k-NN得分脚本，您可以在执行最近的邻居搜索之前在索引上应用过滤器。这对于索引主体可能会根据其他条件变化的动态搜索案例很有用。

由于得分脚本方法执行蛮力搜索，因此它不像[近似方法]({{site.url}}{{site.baseurl}}/search-plugins/knn/approximate-knn)。在某些情况下，最好考虑重构工作流程或索引结构以使用近似方法而不是分数脚本方法。

## 开始使用矢量的分数脚本

类似于近似最近的邻居搜索，为了在矢量体上使用得分脚本，您必须首先创建一个或多个索引`knn_vector` 字段。

如果您打算仅使用分数脚本方法（而不是大致方法），则可以设置`index.knn` 到`false` 并且不设置`index.knn.space_type`。您可以在搜索过程中选择空间类型。看[空间](#spaces) 对于k的空间-NN得分脚本支持。

此示例创建一个用两个`knn_vector` 字段：

```json
PUT my-knn-index-1
{
  "mappings": {
    "properties": {
      "my_vector1": {
        "type": "knn_vector",
        "dimension": 2
      },
      "my_vector2": {
        "type": "knn_vector",
        "dimension": 4
      }
    }
  }
}
```

如果您 *仅 *想使用分数脚本，则可以省略`"index.knn": true`。这种方法的好处是更快的索引速度和较低的内存使用量，但是您失去了执行标准k的能力-nn查询索引。
{： 。提示}

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

最后，您可以使用`knn` 脚本：
```json
GET my-knn-index-1/_search
{
 "size": 4,
 "query": {
   "script_score": {
     "query": {
       "match_all": {}
     },
     "script": {
       "source": "knn_score",
       "lang": "knn",
       "params": {
         "field": "my_vector2",
         "query_value": [2.0, 3.0, 5.0, 6.0],
         "space_type": "cosinesimil"
       }
     }
   }
 }
}
```

所有参数都是必需的。

- `lang` 是脚本类型。这个值通常是`painless`，但是在这里您必须指定`knn`。
- `source` 是脚本的名称，`knn_score`。

  该脚本是K的一部分-NN插件，标准不可用`_scripts` 小路。获取请求`_cluster/state/metadata` 也不返回它。

- `field` 是包含您的向量数据的字段。
- `query_value` 是您想找到最近的邻居的观点。对于欧几里得和余弦相似性空间，该值必须是与磁场映射中设置的尺寸相匹配的浮子数组。对于锤击位距离，此值可以是签名长的类型或base64-编码字符串（分别用于长和二进制字段类型）。
- `space_type` 对应于距离函数。看到[空格部分](#spaces)。

这[大约方法中的过滤示例]({{site.url}}{{site.baseurl}}/search-plugins/knn/approximate-knn#using-approximate-k-nn-with-filters) 显示搜索返回的搜索少于`k` 结果。如果您想避免这种情况，那么得分脚本方法可以从本质上倒转事件顺序。换句话说，您可以过滤以执行k的文档集-最近的邻居搜索。

此示例显示了一个pre-k的过滤方法-使用分数脚本方法进行搜索。首先，创建索引：

```json
PUT my-knn-index-2
{
  "mappings": {
    "properties": {
      "my_vector": {
        "type": "knn_vector",
        "dimension": 2
      },
      "color": {
        "type": "keyword"
      }
    }
  }
}
```

然后添加一些文档：

```json
POST _bulk
{ "index": { "_index": "my-knn-index-2", "_id": "1" } }
{ "my_vector": [1, 1], "color" : "RED" }
{ "index": { "_index": "my-knn-index-2", "_id": "2" } }
{ "my_vector": [2, 2], "color" : "RED" }
{ "index": { "_index": "my-knn-index-2", "_id": "3" } }
{ "my_vector": [3, 3], "color" : "RED" }
{ "index": { "_index": "my-knn-index-2", "_id": "4" } }
{ "my_vector": [10, 10], "color" : "BLUE" }
{ "index": { "_index": "my-knn-index-2", "_id": "5" } }
{ "my_vector": [20, 20], "color" : "BLUE" }
{ "index": { "_index": "my-knn-index-2", "_id": "6" } }
{ "my_vector": [30, 30], "color" : "BLUE" }

```

最后，使用`script_score` 查询预先-在识别最近的邻居之前过滤您的文档：

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
        "lang": "knn",
        "source": "knn_score",
        "params": {
          "field": "my_vector",
          "query_value": [9.9, 9.9],
          "space_type": "l2"
        }
      }
    }
  }
}
```

## 二进制数据的分数脚本入门
k-NN得分脚本还允许您运行k-nn搜索您的二进制数据，并使用锤距离空间进行搜索。
为了使用锤距，感兴趣的领域必须具有`binary` 或者`long` 字段类型。如果您正在使用`binary` 类型，数据必须是base64-编码字符串。

此示例显示了如何使用锤子距离空间`binary` 字段类型：

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "my_binary": {
        "type": "binary",
        "doc_values": true
      },
      "color": {
        "type": "keyword"
      }
    }
  }
}
```

然后添加一些文档：

```json
POST _bulk
{ "index": { "_index": "my-index", "_id": "1" } }
{ "my_binary": "SGVsbG8gV29ybGQh", "color" : "RED" }
{ "index": { "_index": "my-index", "_id": "2" } }
{ "my_binary": "ay1OTiBjdXN0b20gc2NvcmluZyE=", "color" : "RED" }
{ "index": { "_index": "my-index", "_id": "3" } }
{ "my_binary": "V2VsY29tZSB0byBrLU5O", "color" : "RED" }
{ "index": { "_index": "my-index", "_id": "4" } }
{ "my_binary": "SSBob3BlIHRoaXMgaXMgaGVscGZ1bA==", "color" : "BLUE" }
{ "index": { "_index": "my-index", "_id": "5" } }
{ "my_binary": "QSBjb3VwbGUgbW9yZSBkb2NzLi4u", "color" : "BLUE" }
{ "index": { "_index": "my-index", "_id": "6" } }
{ "my_binary":  "TGFzdCBvbmUh", "color" : "BLUE" }

```

最后，使用`script_score` 查询预先-在识别最近的邻居之前过滤您的文档：

```json
GET my-index/_search
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
        "lang": "knn",
        "source": "knn_score",
        "params": {
          "field": "my_binary",
          "query_value": "U29tZXRoaW5nIEltIGxvb2tpbmcgZm9y",
          "space_type": "hammingbit"
        }
      }
    }
  }
}
```

同样，您可以用`long` 字段并进行搜索：

```json
GET my-long-index/_search
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
        "lang": "knn",
        "source": "knn_score",
        "params": {
          "field": "my_long",
          "query_value": 23,
          "space_type": "hammingbit"
        }
      }
    }
  }
}
```

## 空间

一个空间对应于用于测量两个点之间距离以确定K的函数-最近的邻居。来自k-nn透视图，较低的分数等同于更接近，更好的结果。这与OpenSearch分数结果的相反，其中更高的分数等于更好的结果。下表说明了OpenSearch如何将空间转换为得分：

<表>
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
    <td> \ [得分= 2- D \] </td>
  </tr>
  <tr>
    <TD> Interproduct（不支持Lucene）</td>
    <td> \ [D（\ Mathbf {X}，\ Mathbf {y}）=- {\ Mathbf {x}＆middot;\ Mathbf {y}} =- \ sum_ {i = 1}^n x_i y_i \] </td>
    <td> \ [\ text {if} d \ ge 0，\] \ [score = {1 \ over 1 + d} \] \ [\ text {if text {if} d <0，score =＆minus; d + 1 \] </td>
  </tr>
  <tr>
    <td> Hammingbit </td>
    <td> \ [D（\ MathBf {X}，\ Mathbf {y}）= \ text {CountsetBits}（\ MathBf {x} \ oplus \ oplus \ mathbf {y}）
    <td> \ [score = {1 \ over 1 + d} \] </td>
  </tr>
</table>


余弦相似性返回-1和1，并且由于OpenSearch相关性分数不能低于0，所以K-NN插件添加了1个以获得最终分数。

