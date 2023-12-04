---
layout: default
title: 聚合
has_children: true
nav_order: 5
nav_exclude: true
redirect_from:
  - /opensearch/aggregations/
  - /query-dsl/aggregations/
---

# 聚合

OpenSearch不仅是搜索。聚合使您可以利用Opensearch强大的分析引擎来分析您的数据并从中提取统计信息。

聚合的用例从实时分析数据来采取一些措施来使用OpenSearch仪表板来创建可视化仪表板。

OpenSearch可以在毫秒内的大量数据集上执行聚合。与查询相比，聚集消耗更多的CPU周期和内存。

## 文本字段上的聚合

默认情况下，OpenSearch不支持文本字段上的聚合。由于文本字段已被象征化，因此文本字段上的聚合必须将令牌化过程倒回其原始字符串，然后基于该字符串制定聚合。这种操作会消耗大量内存并降低集群性能。

虽然您可以通过设置文本字段启用汇总`fielddata` 参数为`true` 在映射中，聚集仍然基于令牌化的单词，而不是基于原始文本。

我们建议将文本字段的原始版本保留为`keyword` 您可以汇总的字段。

在这种情况下，您可以在`title.raw` 字段，而不是在`title` 场地：

```json
PUT movies
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fielddata": true,
        "fields": {
          "raw": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

## 一般聚集结构

聚合查询的结构如下：

```json
GET _search
{
  "size": 0,
  "aggs": {
    "NAME": {
      "AGG_TYPE": {}
    }
  }
}
```

如果您只对汇总结果感兴趣，而不是对查询结果感兴趣，请设置`size` 到0。

在里面`aggs` 属性（您可以使用`aggregations` 如果需要），可以定义任何数量的聚合。每个聚合都由其名称和OpenSearch支持的聚合类型之一定义。

聚合的名称可帮助您区分响应中的不同聚合。这`AGG_TYPE` 属性是您指定聚合类型的地方。

## 样品聚集

本节使用OpenSearch仪表板示例电子商务数据和Web日志数据。要添加示例数据，请登录到OpenSearch仪表板，请选择**家**，然后选择**尝试我们的示例数据**。为了**样本电子商务订单** 和**示例Web日志**， 选择**添加数据**。

### avg

找到平均值`taxful_total_price` 场地：

```json
GET opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "avg_taxful_total_price": {
      "avg": {
        "field": "taxful_total_price"
      }
    }
  }
}
```

#### 示例响应

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4675,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "avg_taxful_total_price" : {
      "value" : 75.05542864304813
    }
  }
}
```

响应中的聚合块显示了`taxful_total_price` 场地。

## 聚合的类型

聚集的三种主要类型：

- 公制聚合- 计算指标，例如`sum`，`min`，`max`， 和`avg` 在数字字段上。
- 水桶聚集- 根据某些标准将查询结果分组为组。
- 管道聚合- 将一个聚合的输出作为输入作为输入。

## 嵌套聚集

聚合中的聚合称为嵌套或亚凝集。

公制聚合会产生简单的结果，并且不能包含嵌套聚合。

铲斗聚合会产生可以嵌套在其他聚合中的文档桶。您可以通过在存储桶聚合中嵌套度量和存储桶聚合来对数据进行复杂的分析。

### 一般嵌套聚集语法

```json
{
  "aggs": {
    "name": {
      "type": {
        "data"
      },
      "aggs": {
        "nested": {
          "type": {
            "data"
          }
        }
      }
    }
  }
}
```

内心`aggs` 关键字开始一个新的嵌套聚合。母体聚集和嵌套聚集的语法是相同的。嵌套聚合在前面的父聚集的上下文中运行。

您还可以将汇总与搜索查询配对，以缩小您要在汇总之前要分析的内容。如果您不添加查询，则openSearch隐式使用`match_all` 询问。

## 限制

因为使用`double` 所有值的数据类型，`long` 2 <sup> 53 </sup>以上的值近似。

