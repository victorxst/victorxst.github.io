---
layout: default
title: 关于ML Commons
nav_order: 1
has_children: false
has_toc: false
nav_exclude: true
---

# ML Commons插件

用于OpenSearch的ML Commons通过通过运输和REST API调用提供一组ML算法来简化机器学习（ML）功能的开发。这些调用为每个ML请求选择正确的节点和资源，并监视ML任务以确保正常运行时间。这使您可以使用现有的打开-来源ML算法并减少开发新的ML功能所需的精力。

与ML Commons插件的相互作用是通过[REST API]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/index/) 或者[`ad`]({{site.url}}{{site.baseurl}}/search-plugins/sql/ppl/functions#ad) 和[`kmeans`]({{site.url}}{{site.baseurl}}/search-plugins/sql/ppl/functions#kmeans) 管道处理语言（PPL）命令。

[训练有素的模型]({{site.url}}{{site.baseurl}}//ml-commons-plugin/api/train-predict/train/) 通过ML Commons插件支持模型-基于k的算法，例如k-方法。在训练了您的精确要求的模型后，请使用该模型[作出预测]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/train-predict/predict/)。

如果您不想使用模型，则可以使用[训练和预测API]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/train-predict/train-and-predict/) 要测试您的模型而无需评估模型的性能。

## 使用ML CONSONS

1. 确保您适当地设置了所描述的群集设置[ML Commons集群设置]({{site.url}}{{site.baseurl}}/ml-commons-plugin/cluster-settings/)。
2. 如图所述设置模型访问[模型访问控制]({{site.url}}{{site.baseurl}}/ml-commons-plugin/model-access-control/)。
3. 开始使用模型：
  - [在OpenSearch集群中运行自定义模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/)。
  - [集成在外部平台上托管的模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/index/)。

