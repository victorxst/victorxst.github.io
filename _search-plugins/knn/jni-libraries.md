---
layout: default
title: JNI 库
nav_order: 35
parent: k-NN
has_children: false
redirect_from:
 - /search-plugins/knn/jni-library/
---

# JNI 库

集成[nmslib](https://github.com/nmslib/nmslib/) 和[faiss](https://github.com/facebookresearch/faiss/) 近似k-NN功能（在C ++中实现）到K-NN插件（在Java中实现），我们创建了一个Java本机接口，该接口可以让K-NN插件致电到本机库。该界面包括三个库：`libopensearchknn_nmslib`，与nmslib接口的JNI库，`libopensearchknn_faiss`，与faiss接口的JNI库，`libopensearchknn_common`，一个包含本机库之间常见共享功能的库。

Lucene库未使用本机库实施。
{: .note}

图书馆`libopensearchknn_faiss` 和`libopensearchknn_nmslib` 首次在插件中调用时，会懒洋洋地加载。这意味着，如果您仅计划使用其中一个库，则该插件永远不会加载另一个库。

要从源构建库，请参阅[developer_guide](https://github.com/opensearch-project/k-NN/blob/main/DEVELOPER_GUIDE.md)。

有关JNI的更多信息，请参阅[Java本机接口](https://en.wikipedia.org/wiki/Java_Native_Interface) 在维基百科。

