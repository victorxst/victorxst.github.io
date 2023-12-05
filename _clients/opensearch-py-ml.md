---
layout: default
title: Opensearch-py-ml
nav_order: 11
---

# opensearch-py-ml

`opensearch-py-ml` 是Python客户端，提供一套数据分析和自然语言处理（NLP）支持工具的套件。它为数据分析师提供了：

- 调用OpenSearch索引并使用OpenSearch操纵它们-py-ML[数据框架](https://opensearch-project.github.io/opensearch-py-ml/reference/dataframe.html) 蜜蜂。OpenSearch-py-ML DataFrame将OpenSearch索引包装到类似的API中[熊猫](https://pandas.pydata.org/)，使您能够从jupyter笔记本中处理大量数据。
- 上传NLP[vencentansformer](https://www.sbert.net/) 使用模型使用[ML Commons插件]({{site.url}}{{site.baseurl}}/ml-commons-plugin/index/)。
- 带有合成查询的训练和调整录音词反应器模型。

## 先决条件

使用`opensearch-py-ml`，安装[OpenSearch Python客户端]({{site.url}}{{site.baseurl}}/clients/python-low-level#setup)。Python客户端允许OpenSearch使用在运行数据范围所需的Python语法`opensearch-py-ml`。

## 安装`opensearch-py-ml`

要将客户端添加到您的项目中，请使用[pip](https://pip.pypa.io/)：

```bash
pip install opensearch-py-ml
```
{% include copy.html %}

然后像其他任何模块一样将客户端导入到OpenSearch中：

```python
from opensearchpy import OpenSearch
import opensearch_py_ml as oml
```
{% include copy.html %}

## API参考

有关所有OpenSearch的信息-py-ML对象，功能和方法，请参阅[OpenSearch-py-ML API参考](https://opensearch-project.github.io/opensearch-py-ml/reference/index.html)。

## 下一步

如果您想跟踪或为发展`opensearch-py-ml` 客户，请参阅[OpenSearch-py-ML GitHub存储库](https://github.com/opensearch-project/opensearch-py-ml)。

例如，与客户一起使用的Python笔记本电脑，请参阅[例子](https://opensearch-project.github.io/opensearch-py-ml/examples/index.html)。

