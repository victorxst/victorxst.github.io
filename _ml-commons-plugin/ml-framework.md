---
layout: default
title: 在OpenSearch中使用ML模型
has_children: true
nav_order: 50
redirect_from:
   - /ml-commons-plugin/model-serving-framework/
---

# 在OpenSearch中使用ML模型
**通常可用2.9**
{: .label .label-purple }

要将机器学习（ML）模型集成到您的OpenSearch集群中，您可以在本地上传并为其提供服务。选择以下选项之一：

- **审计的模型由OpenSearch提供**：要了解更多，请参阅[OpenSearch-提供了预告片的模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/pretrained-models/)。有关支持模型的列表，请参见[支持的预贴模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/pretrained-models/#supported-pretrained-models)。

- **自定义模型** 例如Pytorch深度学习模型：要了解更多信息，请参阅[自定义模型](http://localhost:4000/docs/latest/ml-commons-plugin/custom-local-models/)。

## GPU加速度

为了获得更好的性能，您可以利用ML节点上的GPU加速度。有关更多信息，请参阅[GPU加速度]({{site.url}}{{site.baseurl}}/ml-commons-plugin/gpu-acceleration/)。


