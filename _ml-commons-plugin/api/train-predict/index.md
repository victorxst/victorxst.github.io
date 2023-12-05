---
layout: default
title: 训练和预测API
parent: ML Commons API
has_children: true
nav_order: 30
---

# 训练和预测API

ML Commons API使您可以同步和异步训练机器学习（ML）算法，使用该训练的模型进行预测，并使用相同的数据集进行训练和预测。

要通过API训练任务，需要三个输入：

- 算法名称：必须是[函数名称](https://github.com/opensearch-project/ml-commons/blob/1.3/common/src/main/java/org/opensearch/ml/common/parameter/FunctionName.java)。这确定了ML引擎运行的算法。要添加一个新功能，请参阅[如何添加新功能](https://github.com/opensearch-project/ml-commons/blob/main/docs/how-to-add-new-function.md)。
- 模型超参数：调整这些参数以提高模型精度。
- 输入数据：训练ML模型或将ML模型应用于预测的数据。您可以通过两种方式输入数据，对索引查询或使用数据框架。

ML Commons支持以下火车并预测API：

- [火车]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/train-predict/train/)
- [预测]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/train-predict/predict/)
- [训练和预测]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/train-predict/train-and-predict/)

