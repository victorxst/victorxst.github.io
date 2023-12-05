---
layout: default
title: k-NN
nav_order: 50
has_children: true
has_toc: false
redirect_from:
  - /search-plugins/knn/
---

# k-nn

*k的缩写-最近的邻居*，K-NN插件使用户可以搜索K-最近的邻居到一个向量索引的查询点。要确定邻居，您可以指定要使用的空间（距离函数）来测量点之间的距离。

用例包括建议（例如"other songs you might like" 音乐应用程序中的功能），图像识别和欺诈检测。有关K的更多背景信息-nn搜索，请参阅[维基百科](https://en.wikipedia.org/wiki/Nearest_neighbor_search)。

该插件支持获得K的三种不同方法-矢量索引最近的邻居：

1. **近似k-nn**

    第一种方法采用大约最近的邻居方法---它使用几种算法之一返回近似k-最近的邻居与查询矢量。通常，这些算法牺牲了索引速度和搜索精度，以换取性能益处，例如较低的延迟，较小的记忆足迹和更可扩展的搜索。要了解有关算法的更多信息，请参阅[*nmslib*](https://github.com/nmslib/nmslib/blob/master/manual/README.md)'沙[*faiss*](https://github.com/facebookresearch/faiss/wiki)的文档。

    近似k-NN是需要低潜伏期的大型索引（即数十万个或更多的矢量）进行搜索的最佳选择。您不应该使用近似K-nn如果要在k之前在索引上应用过滤器-NN搜索，大大减少了要搜索的向量的数量。在这种情况下，您应该使用脚本评分方法或无痛的扩展。

    有关此方法的更多详细信息，包括有关使用哪种引擎的建议，请参阅[近似k-nn搜索]({{site.url}}{{site.baseurl}}/search-plugins/knn/approximate-knn/)。

2. **脚本得分k-nn**

    第二种方法扩展了OpenSearch的脚本评分功能以执行蛮力，精确的K-nn搜索"knn_vector" 可以代表二进制对象的字段或字段。使用这种方法，您可以运行k-nn搜索索引中的向量子集（有时称为pre-过滤器搜索）。

    使用此方法进行搜索较小的文档机构或何时搜索-需要过滤器。在大索引上使用这种方法可能会导致较高的潜伏期。

    有关此方法的更多详细信息，请参阅[精确k-NN带有评分脚本]({{site.url}}{{site.baseurl}}/search-plugins/knn/knn-score-script/)。

3. **无痛的扩展**

    第三种方法添加了距离的功能，可以作为无痛的扩展，您可以在更复杂的组合中使用。类似于k-nn脚本得分，您可以使用此方法执行蛮力，精确k-NN搜索跨索引，这也支持PRE-过滤。

    与K相比，这种方法的查询性能略慢。-NN脚本得分。如果您的用例需要在最终分数上进行更多的自定义，则应使用此方法对脚本分数k进行使用-nn。

    有关此方法的更多详细信息，请参阅[无痛的脚本功能]({{site.url}}{{site.baseurl}}/search-plugins/knn/painless-functions/)。


总体而言，对于较大的数据集，您通常应该选择大约最近的邻居方法，因为它的缩放率明显更好。对于较小的数据集（您可能想应用过滤器），您应该选择自定义评分方法。如果您有一个更复杂的用例，其中需要将距离函数用作其评分方法的一部分，则应使用无痛的脚本方法。

