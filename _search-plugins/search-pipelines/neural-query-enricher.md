---
layout: default
title: 神经查询浓度
nav_order: 12
has_children: false
parent: 搜索处理器
grand_parent: 搜索管道
---

# 神经查询富集处理器

这`neural_query_enricher` 搜索请求处理器旨在将默认的机器学习（ML）模型ID设置为索引或字段级别[神经搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-search/) 查询。要了解有关ML模型的更多信息，请参阅[在OpenSearch中使用ML模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/) 和[连接到远程型号]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/index/)。

## 请求字段

下表列出了所有可用的请求字段。

场地| 数据类型| 描述
：--- | ：--- | ：---
`default_model_id` | 细绳| 索引的默认模型的模型ID。选修的。您必须至少指定一个`default_model_id` 或者`neural_field_default_id`。如果两者都提供，`neural_field_default_id` 优先。
`neural_field_default_id` | 目的| 密钥地图-代表文档字段名称及其关联的默认模型ID的值对。选修的。您必须至少指定一个`default_model_id` 或者`neural_field_default_id`。如果两者都提供，`neural_field_default_id` 优先。
`tag` | 细绳| 处理器的标识符。选修的。
`description` | 细绳| 处理器的描述。选修的。

## 例子

以下示例请求使用`neural_query_enricher` 搜索请求处理器。处理器在索引级别设置默认模型ID，并为索引中的两个特定字段提供不同的默认模型ID：

```json
PUT /_search/pipeline/default_model_pipeline 
{
  "request_processors": [
    {
      "neural_query_enricher" : {
        "tag": "tag1",
        "description": "Sets the default model ID at index and field levels",
        "default_model_id": "u5j0qYoBMtvQlfhaxOsa",
        "neural_field_default_id": {
           "my_field_1": "uZj0qYoBMtvQlfhaYeud",
           "my_field_2": "upj0qYoBMtvQlfhaZOuM"
        }
      }
    }
  ]
}
```
{％包含副本-curl.html％}

