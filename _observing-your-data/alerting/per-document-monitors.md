---
layout: default
title: 每个文档监视器
nav_order: 20
parent: 监视器
grand_parent: 警报
has_children: false
---

# 每个文档监视器
引入2.0
{: .label .label-purple }

每个文档监视器是一种警报监视器，可用于识别和警报OpenSearch索引中的特定文档。例如，您可以使用监视器来：

- 检测损坏的数据或未经授权的更改。
- 执行数据质量策略，例如确保所有文档包含一定字段或字段中的值在一定范围内。
- 跟踪特定文档随着时间的变化，这可能有助于审计和合规性目的

## 定义查询

每个文档监视器允许您最多定义10个查询，以比较所需值所选字段。您可以使用以下操作员来定义支持的现场数据类型：

- `is` 
- `is not`
- `is greater than`
- `is greater than equal`
- `is less than`
- `is less than equal`

你可以查询每个[扳机]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/triggers/) 最多使用10个标签，将标签添加为单个触发条件，而不是指定单个查询。这[警报插件]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/monitors/) 处理所有查询的触发条件作为逻辑`OR` 操作，因此，如果满足任何查询条件，它会触发警报。然后，警报插件告诉[通知插件]({{site.url}}{{site.baseurl}}/observing-your-data/notifications/index/) 将警报通知发送到频道。

您只能使用_tags_--- 也就是说，可以应用于多个查询以与逻辑相结合的标签`OR`` operation---in a per document monitor.
{: .important}

## Document findings

The Alerting plugin creates a list of _Findings_ that contain metadata about which document matches each query. A _Finding_ is a record of a document identified by the per document monitor query as meeting the alert condition. Key components of a finding include the document ID, timestamp, alert condition details. Findings are stored in the Findings index, `.opensearch-警报-寻找*`。

安全分析可以使用调查结果数据来跟踪和分析与警报过程分开的查询数据。看[与调查结果一起工作]({{site.url}}{{site.baseurl}}/security-analytics/usage/findings/) 了解更多。
{: .note}

警报API还提供了一个_个图表-Level Monitor_可以编程地完成与OpenSearch仪表板中的_per文档Monitor_相同的功能。看[文档-电平监视器]({{site.url}}{{site.baseurl}}/monitoring-plugins/alerting/api/#document-level-monitors) 了解更多。

为了防止大量发现-摄入群集，不建议为每个发现配置警报通知，除非规则定义得当。
{： 。重要的}

为每个文档发现条目提供以下元数据：

***文档**：文档ID和索引名称。例如：`Re5akdirhj3fl | test-logs-index`。
***询问**：与文档匹配的查询名称。
***找到时间**：指示在运行时发现文档的时间戳。

