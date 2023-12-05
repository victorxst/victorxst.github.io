---
layout: default
title: 关于安全分析
nav_order: 1
has_children: false
has_toc: false
nav_exclude: true
redirect_from:
  - /security-analytics/
---


# 关于安全分析


安全分析是一种安全信息和事件管理（SIEM）解决方案，旨在调查，检测，分析和应对安全威胁，这可能会损害企业和组织及其在线运营的成功。这些威胁包括潜在的机密数据暴露，网络攻击和其他不利安全事件。安全分析提供了一个-的-这-使用任何OpenSearch发行版自动安装的框解决方案。它包括定义检测参数，生成警报并有效响应潜在威胁所需的工具和功能。


### 资源和信息

作为OpenSearch项目的一部分，安全分析存在于开源社区中，并从该社区的反馈和贡献中受益。要了解有关其发展的建议，做出贡献的选择以及平台上的一般信息，请参见[安全分析存储库](https://github.com/opensearch-project/security-analytics) 在Github。

如果您想留下可以帮助改善安全分析的反馈，请加入有关[OpenSearch论坛](https://forum.opensearch.org/c/plugins/security-analytics/73)。


---
## Components and concepts

安全分析包括其操作的许多工具和功能。组成插件的主要组件在以下各节中总结。


### Detectors

探测器是核心组件，其配置为识别一系列网络安全威胁，对应于-[miter＆ck]（https://attack.mitre.org/）组织维护的对手策略和技术的知识库不断增长。检测器使用日志数据来评估系统中发生的事件。然后，他们应用一组为检测器指定的安全规则，并确定这些事件的发现。

有关配置检测器的信息，请参见[创建检测器]({{site.url}}{{site.baseurl}}/security-analytics/sec-analytics-config/detectors-config/)。


### Log types

日志类型提供用于评估系统中发生事件的数据。OpenSearch支持几种类型的日志，并提供-的-这-最常见的日志源的框映射。请参阅({{site.url}}{{site.baseurl}}/security-analytics/sec-analytics-config/log-types/)对于安全分析当前支持的日志类型列表。

日志类型是在创建检测器期间指定的，包括将日志字段映射到检测器的步骤。安全分析还可以自动根据特定日志类型选择一组适当的规则，并为检测器填充它们。


### Detection rules

安全规则或威胁检测规则定义了应用于摄入的日志数据的条件逻辑，该逻辑允许系统识别感兴趣的事件。安全分析使用预包装的开源[Sigma规则]（https://github.com/sigmahq/sigma）作为描述相关日志事件的起点。但是，凭借其固有的灵活格式和简单的便携性，Sigma规则为安全分析用户提供了用于导入和自定义规则的选项。您可以使用OpenSearch仪表板或API来利用这些选项。

有关配置规则的信息，请参阅[使用规则]({{site.url}}{{site.baseurl}}/security-analytics/usage/rules/)。


### Findings

每当检测器与日志事件匹配规则时，都会生成发现。调查结果不一定指出系统内的迫在眉睫的威胁，但它们总是隔离感兴趣的事件。由于它们代表了检测器的特定定义的结果，因此发现包括选定规则，日志类型和规则严重性的独特组合。因此，您可以在“发现”窗口中搜索特定的发现，并且可以根据严重性和日志类型过滤列表中的发现。

要了解有关发现的更多信息，请参见[使用发现]({{site.url}}{{site.baseurl}}/security-analytics/usage/findings/)。


### Alerts

定义检测器时，您可以指定某些条件会触发警报。当事件触发警报时，系统将向首选频道（例如Amazon Chime，Slack或Email）发送通知。当检测器匹配一个或多个规则时，可以触发警报。可以通过规则严重性和标签来设置进一步的条件。您还可以使用自定义的主题行和消息主体创建通知消息。

有关设置警报的信息，请参见[创建检测器]({{site.url}}{{site.baseurl}}/security-analytics/sec-analytics-config/detectors-config/)。有关在“警报”窗口中管理警报的信息，请参见[使用警报]({{site.url}}{{site.baseurl}}/security-analytics/usage/alerts/)。


### Correlation engine

相关引擎使安全分析能够比较来自不同日志类型的发现并绘制它们之间的相关性。这有助于理解基础架构中不同系统的发现之间的关系，并增加了对事件有意义的信心，需要关注。

相关引擎使用相关规则来定义涉及不同日志类型的威胁场景。然后，它可以在日志上执行查询，以匹配来自这些不同日志源的相关发现。要描述发生在不同日志中的事件之间的关系，相关图提供了发现，它们的连接以及这些连接的接近性的视觉表示。尽管相关规则定义了要寻找的威胁方案，但该图提供了可视化，可帮助您确定安全事件链中不同发现之间的关系。

要了解有关定义相关规则的威胁方案的更多信息，请参见[创建相关规则]({{site.url}}{{site.baseurl}}/security-analytics/sec-analytics-config/correlation-config/)。要了解有关使用相关图的更多信息，请参见[使用相关图]({{site.url}}{{site.baseurl}}/security-analytics/usage/correlation-graph/)。


---
## 第一步

要开始使用安全分析，您需要定义检测器，摄入日志数据，生成发现，定义相关规则和配置警报。看[设置安全分析]({{site.url}}{{site.baseurl}}/security-analytics/sec-analytics-config/index/) 开始配置平台以实现您的目标。


