---
layout: default
title: 搜索相关性
nav_order: 55
has_children: true
has_toc: false
redirect_from:
  - /search-plugins/search-relevance/
---

# 搜索相关性

搜索相关性评估查询返回的搜索结果的准确性。相关性越高，搜索引擎越好。比较搜索结果是OpenSearch中的第一个搜索相关功能。

## 比较搜索结果

比较OpenSearch仪表板中的搜索结果使您可以并排比较两个查询的结果，以确定一个查询是否会产生比另一个查询更好的结果。使用此工具，您可以通过尝试查询来评估搜索质量。

例如，当应用以下查询之一更改时，您可以看到结果如何变化：

- 以不同的方式加权不同的领域
- 不同的茎或柠檬水策略
- 摇摆

## 先决条件

在开始之前，必须在OpenSearch中索引数据。要了解如何创建新索引，请参阅[索引数据]({{site.url}}{{site.baseurl}}/opensearch/index-data)。

另外，您可以使用以下步骤在OpenSearch仪表板中添加示例数据：

1. 在顶部菜单栏上，转到**OpenSearch仪表板>概述**。
1. 选择**查看应用程序目录**。
1. 选择**添加示例数据**。
1. 选择其中之一-在数据集中，选择**添加数据**。

## 使用比较搜索仪表板中的搜索结果

要比较OpenSearch仪表板中的搜索结果，请执行以下步骤。

**步骤1：** 在顶部菜单栏上，转到**OpenSearch插件>搜索相关性**。

**第2步：** 在搜索栏中输入搜索文本。

**步骤3：** 选择一个索引**查询1** 并输入查询（仅请求主体）[OpenSearch查询DSL]({{site.url}}{{site.baseurl}}/opensearch/query-dsl)。这`GET` HTTP方法和`_search` 端点是隐式的。使用`%SearchText%` 可变以参考搜索栏中的文本。

以下是一个示例查询：

```json
{
  "query": {
    "multi_match": {
      "query": "%SearchText%",
      "fields": [ "description", "item_name" ]
    }
  }
}
```

**步骤4：** 选择一个索引**查询2** 并输入查询（仅请求主体）。

以下示例查询提升`title` 搜索结果中的字段：

```json
{
  "query": {
    "multi_match": {
      "query": "%SearchText%",
      "fields": [ "description", "item_name^3" ]
    }
  }
}
```

**步骤5：** 选择**搜索** 并比较结果**结果1** 和**结果2**。

以下示例屏幕显示了搜索单词"cup" 在里面`description` 和`item_name` 有和没有提升的领域`item_name`：

<img src="{{site.url}}{{site.baseurl}}/images/search_relevance.png" alt="Compare search results"/>{: .img-fluid }

如果结果1中的结果在结果2中出现，则`Up` 和`Down` 与结果2相比，结果编号低于结果编号，结果与结果2相比。在此示例中，ID 2的文档为`Up 1` 与结果1相比，在结果2中放置`Down 1` 与结果2相比，放置在结果1中。

## 更改结果数

默认情况下，OpenSearch返回前10个结果。要将返回结果的数量更改为其他值，请指定`size` 查询中的参数：

```json
{
  "size": 15,
  "query": {
    "multi_match": {
      "query": "%SearchText%",
      "fields": [ "title^3", "text" ]
    }
  }
}
```

环境`size` 高价值（例如，大于250个文档）可能会降低性能。
{:.note}

您无法保存给定的比较以供将来使用，因此比较搜索结果不适合系统测试。
{:.note}

## 将OpenSearch搜索结果与重新访问结果进行比较

一个用于比较搜索结果的用例是将RAW OpenSearch结果与通过Reranking应用程序处理相同的结果进行比较。OpenSearch当前与以下两个Rerankers集成：

- [Kendra智能排名opensearch](#reranking-results-with-kendra-intelligent-ranking-for-opensearch)
- [亚马逊个性化搜索排名](#personalizing-search-results-with-amazon-personalize-search-ranking)

### 肯德拉（Kendra）智能排名的重读结果

重读者的一个例子是**Kendra智能排名opensearch**由亚马逊肯德拉团队贡献。该插件从OpenSearch获取搜索结果，并应用了使用向量嵌入和其他语义搜索技术计算的Amazon Kendra的语义相关排名。对于许多应用程序，这提供了更好的结果排名。

要尝试Kendra Intelligent排名，您必须首先设置Amazon Kendra服务。要开始，请参阅[亚马逊肯德拉](https://aws.amazon.com/kendra/)。有关详细信息，包括插件设置说明，请参阅[使用Amazon Kendra智能排名OpenSearch（自我管理）结果](https://docs.aws.amazon.com/kendra/latest/dg/opensearch-rerank.html)。

### 将搜索结果与重新访问的结果进行比较opensearch仪表板中的结果

要将搜索结果与Reranked结果进行比较OpenSearch仪表板的结果，请输入查询**查询1** 并使用reranker输入相同的查询**查询2**。然后将Opensearch的搜索结果与重新依据的结果进行比较。

以下示例搜索文本"snacking nuts" 在里面`abo` 指数。索引中的文档包含零食描述`bullet_point` 大批。

<img src="{{site.url}}{{site.baseurl}}/images/kendra_query.png" alt="OpenSearch Intelligent Ranking query"/>{: .img-fluid }

1. 进入`snacking nuts` 在搜索栏中。
1. 输入以下查询，搜索`bullet_point` 搜索文本的字段"snacking nuts"， 在**查询1**：

    ```json
    {
      "query": {
        "match": {
          "bullet_point": "%SearchText%"
        }
      },
      "size": 25
    }
    ```
1. 使用reranker输入相同的查询**查询2**。此示例使用Kendra智能排名：

    ```json
    {
      "query" : {
        "match" : {
          "bullet_point": "%SearchText%"
        }
      },
      "size": 25,
      "ext": {
        "search_configuration":{
          "result_transformer" : {
            "kendra_intelligent_ranking": {
              "order": 1,
              "properties": {
                "title_field": "item_name",
                "body_field": "bullet_point"
              }
            }
          }
        }
      }
    }
    ```

    在上一个查询中，`body_field` 指索引中文档的身体场，肯德拉智能排名用来对结果进行排名。这`body_field` 是必需的，而`title_field` 是可选的。
1. 选择**搜索** 并比较结果**结果1** 和**结果2**。

### 亚马逊个性化搜索结果个性化搜索排名

重读者的另一个例子是**亚马逊个性化搜索排名**，由亚马逊个性化团队贡献。亚马逊个性化使用机器学习（ML）技术为您的用户生成自定义建议。该插件从OpenSearch获取搜索结果，并应用[搜索管道]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/index/) 根据他们的亚马逊个性化排名，将它们重读。Amazon个性化排名基于用户的过去行为和有关搜索项和用户的元数据。此工作流程通过个性化搜索结果来改善用户的搜索体验。

要尝试Amazon个性化搜索排名，您必须首先设置Amazon个性化。要开始，请参阅[亚马逊个性化](https://docs.aws.amazon.com/personalize/latest/dg/setup.html)。有关详细信息，包括插件设置说明，请参阅[个性化搜索结果来自OpenSearch（self-管理）](https://docs.aws.amazon.com/personalize/latest/dg/personalize-opensearch.html)。

