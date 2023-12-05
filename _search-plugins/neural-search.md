---
layout: default
title: 神经搜索
nav_order: 200
has_children: true
has_toc: false
redirect_from: 
  - /neural-search-plugin/index/
---

# 神经搜索

神经搜索将文本转变为矢量，并在摄入时和搜索时间促进矢量搜索。在摄入期间，神经搜索将文档文本转换为矢量嵌入，并将文本及其矢量嵌入在矢量索引中。当您在搜索过程中使用神经查询时，神经搜索将查询文本转换为矢量嵌入，使用矢量搜索来比较查询和文档嵌入，并返回最接近的结果。

在将文档摄入索引之前，文档将通过机器学习（ML）模型传递，该模型生成了文档字段的向量嵌入。当您发送搜索请求时，查询文本或图像也会通过ML模型传递，该模型生成相应的向量嵌入。然后，神经搜索在嵌入式上执行矢量搜索，并返回匹配文档。

## 先决条件

在使用神经搜索之前，您必须设置ML模型。您可以使用OpenSearch提供的验证模型，将自己的模型上传到OpenSearch，或将其连接到托管在外部平台上的基础模型。有关ML模型的更多信息，请参阅[在OpenSearch中使用ML模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/) 和[连接到远程型号]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/index/)。一步-经过-步骤教程，请参阅[语义搜索]({{site.url}}{{site.baseurl}}/ml-commons-plugin/semantic-search/)。

## 设置神经搜索

设置ML模型后，请选择以下神经搜索类型之一，以了解如何使用模型进行神经搜索。

### 神经文本搜索

神经文本搜索使用基于文本嵌入模型的密集检索来搜索文本数据。有关详细的设置步骤，请参阅[文字搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-text-search/)。

### 多模式搜索

多模式搜索使用多模式嵌入模型来搜索文本和图像数据。有关详细的设置步骤，请参阅[多模式搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-multimodal-search/)。

### 神经稀疏搜索

神经稀疏搜索使用基于稀疏嵌入模型的稀疏检索来搜索文本数据。有关详细的设置步骤，请参阅[稀疏搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-sparse-search/)。

