---
layout: default
title: 个性化搜索排名
nav_order: 40
has_children: false
parent: 搜索处理器
grand_parent: 搜索管道
---

# 个性化搜索排名处理器

这`personalize_search_ranking` 搜索响应处理器拦截搜索响应并使用[亚马逊个性化](https://aws.amazon.com/personalize/) 根据他们的亚马逊个性化排名，请重新搜索结果。该排名基于用户过去的行为和搜索项和用户的元数据。

使用`personalize_search_ranking` 处理器，您必须首先安装Amazon个性化搜索排名（`opensearch-search-processor`） 插入。有关详细说明，请参阅[安装和配置Amazon个性化搜索排名插件](https://docs.aws.amazon.com/personalize/latest/dg/opensearch-install.html)。
{： 。重要的}

## 请求字段

下表列出了所有可用的请求字段。

场地| 数据类型| 描述
:--- | :--- | :--- 
`campaign_arn` | 细绳|  亚马逊个性化广告系列的亚马逊资源名称（ARN）用于个性化结果。必需的。
`recipe` | 细绳| 亚马逊个性化食谱的名称要使用。当前，该字段的唯一支持值是`aws-personalized-ranking`。必需的。
`weight` | 漂浮| 与OpenSearch和Amazon提供的排名一起使用的权重。有效值在[0.0，1.0]范围内。重量越接近1.0，与计算排名时相比，亚马逊个性化的权重越多。如果指定0.0，则使用OpenSearch排名。如果指定1.0，则使用Amazon个性化排名。必需的。
`item_id_field` | 细绳| 如果是`_id` OpenSearch中的索引文档的字段与您的Amazon个性化不符`itemId`，指定执行字段的名称。默认情况下，插件假设`_id` 数据与`itemId` 在您的亚马逊个性化数据中。
`iam_role_arn` | 细绳| 如果您使用多个角色来限制组织中不同群体用户的权限，请指定有权访问亚马逊个性化的角色的ARN。如果您仅在OpenSearch密钥库中使用AWS凭据，则可以省略此字段。选修的。
`tag` | 细绳| 处理器的标识符。选修的。
`description` | 细绳| 处理器的描述。选修的。
`ignore_failure` | 布尔| 如果`true`，OpenSearch[忽略任何故障]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/creating-search-pipeline/#ignoring-processor-failures) 该处理器并继续在搜索管道中运行其余的处理器。选修的。默认为`false`。

## 例子

以下示例证明了使用搜索管道与`personalize_search_ranking` 处理器。

### 创建搜索管道

以下请求会创建一个使用`personalize_search_ranking` 响应处理器：

```json
PUT /_search/pipeline/my-pipeline
{
  "description": "A pipeline to apply custom reranking from Amazon Personalize",
  "response_processors" : [
    {
      "personalized_search_ranking" : {
        "campaign_arn" : "Amazon Personalize Campaign ARN",
        "item_id_field" : "productId",
        "recipe" : "aws-personalized-ranking",
        "weight" : "0.3",
        "tag" : "personalize-processor",
        "iam_role_arn": "Role ARN",
        "aws_region": "AWS region"
      }
    }
  ]
}
```
{% include copy-curl.html %}

### 使用搜索管道

要使用管道搜索，请在`search_pipeline` 查询参数。例如，以下请求使用上一节中设置的管道来搜索喜剧：

```json
GET /movies/_search?search_pipeline=my-pipeline
{
  "query": {
    "multi_match": {
      "query": "Comedy",
      "fields": ["GENRES"]
    }
  },
  "ext": {
    "personalize_request_parameters": {
      "user_id": "user ID",
      "context": { "DEVICE" : "mobile phone" }
    }
  }
}
```
{% include copy-curl.html %}

有关其他详细信息，请参阅[个性化搜索结果来自OpenSearch（self-管理）](https://docs.aws.amazon.com/personalize/latest/dg/personalize-opensearch.html)

