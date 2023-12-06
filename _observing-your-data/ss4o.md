---
layout: default
title: 可观察性的简单模式
nav_order: 120
redirect_from:
- /observing-your-data/ssfo/ 
---

# 可观察性的简单模式
引入2.6
{: .label .label-purple }

[可观察性]({{site.url}}{{site.baseurl}}/observing-your-data/index/) 是插件和应用程序的集合，可让您可视化数据-通过使用管道处理语言（PPL）来探索和查询OpenSearch中存储的数据。[可观察性的简单模式](https://github.com/opensearch-project/opensearch-catalog/tree/main/docs/schema/observability)，使用模式约定`ss4o`，是符合常见和统一的可观察性模式的标准化。有了架构，可观察性工具可以摄入，自动提取和汇总数据并创建自定义仪表板，从而更容易在更高级别上理解系统。

可观察性的简单架构都受到两者的启发[Opentelemetry](https://opentelemetry.io/docs/) 以及弹性常见模式（ECS）并使用Amazon弹性容器服务（[亚马逊ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_cwe_events.html)）事件日志和opentelemetry（Otel）元数据。

将来的发行版将支持警报。
{: .note}

## 用例

可观察性的简单模式的用例包括：

*从不同数据类型中摄取可观察性数据。
*从非专有配置转变为非-可转移到合并的可共享可观察性解决方案，该解决方案允许用户摄入并显示任何类型的遥测数据的分析。
*将仪表板符合仪表板与数据结构保持一致，以便您可以以有效代表您的数据的方式设计和组织仪表板组件和可视化。

数据预先符合指标的模式，并将逐渐支持迹线和日志。数据预备[跟踪映射]({{site.url}}{{site.baseurl}}/data-prepper/common-use-cases/trace-analytics/) 当前提供`service-map` 以不同的方式数据`ss4o` 迹线。为了使跟踪映射与可观察性兼容，它将与`ss4o` 痕迹模式并将介绍`service-map` 作为一个丰富的领域。
{: .note}

## 痕迹和指标

可观察性插件定义和支持迹线和指标的架构定义。这些模式定义包括：

- 索引结构（映射）。
- 这[索引命名约定](https://github.com/opensearch-project/observability/issues/1405)。
- 用于执行和验证结构的JSON模式。
- 这[一体化](https://github.com/opensearch-project/OpenSearch-Dashboards/issues/3412) 用于添加预配置仪表板和资产的功能。

