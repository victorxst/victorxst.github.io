---
layout: default
title: 安全分析设置
nav_order: 100
has_children: false
---

# 安全分析设置

安全分析插件支持以下设置。此列表中的所有设置都是动态的：

`plugins.security_analytics.index_timeout` （时间值）：使用REST API创建检测器，发现，规则和自定义日志类型的超时。默认值为60秒。

`plugins.security_analytics.alert_history_enabled` （布尔）：指定是否创建`.opensearch-sap-<detector_type>-alerts-history-<date>` 索引。默认为`true`。

`plugins.security_analytics.alert_finding_enabled` （布尔）：指定是否创建`.opensearch-sap-<detector_type>-findings-<date>` 索引。默认为`true`。

`plugins.security_analytics.alert_history_rollover_period` （时间值）：指定频率翻转的频率和删除警报历史记录索引。默认值为12小时。

`plugins.security_analytics.alert_finding_rollover_period` （时间值）：指定频率翻转的频率和删除查找历史记录索引。默认值为12小时。

`plugins.security_analytics.correlation_history_rollover_period` （时间值）：指定滚动和删除相关历史记录索引的频率。默认值为12小时。

`plugins.security_analytics.alert_history_max_age` （时间值）：在创建新索引之前，最旧的文档存储在警报记录索引中。如果此时间段的警报数量不超过`alert_history_max_docs`，每个周期都会创建一个新的警报历史记录索引（例如，每30天一个索引）。默认值为30天。

`plugins.security_analytics.finding_history_max_age` （时间值）：在创建新索引之前，最旧的文档存储在查找历史索引中。如果此期间的发现数不超过`finding_history_max_docs`，每个时期创建一个新的发现历史指数（例如，每30天一个索引）。默认值为30天。

`plugins.security_analytics.correlation_history_max_age` （时间值）：在创建新索引之前，最旧的文档存储在相关历史记录索引中。如果此时间段内的相关数量不超过`correlation_history_max_docs`，每个周期都会创建一个新的相关历史记录索引（例如，每30天一个索引）。默认值为30天。

`plugins.security_analytics.alert_history_max_docs` （整数）：在创建新索引之前，在警报记录索引中存储的最大警报数量。默认值为1,000。

`plugins.security_analytics.alert_finding_max_docs` （整数）：在创建新索引之前，要存储在发现历史记录索引中的最大发现数。默认值为1,000。

`plugins.security_analytics.correlation_history_max_docs` （整数）：创建新索引之前，要存储在相关历史记录索引中的最大相关数。默认值为1,000。

`plugins.security_analytics.alert_history_retention_period` （时间值）：在自动删除警报记录索引之前保留警报索引的时间。默认值为60天。

`plugins.security_analytics.finding_history_retention_period` （时间值）：在自动删除历史记录索引之前，请继续查找历史记录索引的时间。默认值为60天。

`plugins.security_analytics.correlation_history_retention_period` （时间值）：自动删除相关历史记录索引的时间量。默认值为60天。

`plugins.security_analytics.request_timeout` （时间值）：安全分析插件的所有请求超时发送到OpenSearch的其他部分。默认值为10秒。

`plugins.security_analytics.action_throttle_max_value` （时间值）：您可以设置为操作节流的最大时间。默认值为24小时。（此值在OpenSearch仪表板中显示为1440分钟。）

`plugins.security_analytics.filter_by_backend_roles` （布尔）：设置为`true`，限制启用时访问探测器，警报，调查结果和自定义日志类型的访问。默认为`false`。

`plugins.security_analytics.enable_workflow_usage` （布尔值）：支持与安全分析的警报插件工作流程集成。在创建安全分析中的新威胁检测器后，确定是否为警报插件生成复合监视器工作流。设置为`true`，启用了基于相关威胁检测器的配置的复合监视器工作流程。设置为`false`，基于相关威胁检测器的配置的复合监视器工作流程被禁用。默认为`true`。有关警报插件工作流与安全分析的更多信息，请参见[集成警报插件工作流程]({{site.url}}{{site.baseurl}}/security-analytics/sec-analytics-config/detectors-config/#integrated-alerting-plugin-workflows)。

`plugins.security_analytics.correlation_time_window` （时间值）：安全分析在时间窗口内生成相关性。此设置指定必须将文档索引到索引中的时间窗口，以便包含在同一相关中。默认值为5分钟。

`plugins.security_analytics.mappings.default_schema` （字符串）：用于配置安全分析检测器的字段映射的默认映射架构。默认为`ecs`。

`plugins.security_analytics.threatintel.tifjob.update_interval` （时间值）：威胁智能功能使用作业跑步者定期获取新提要。此设置是跑步者获取并更新这些新提要的速度。默认值为1440分钟。

`plugins.security_analytics.threatintel.tifjob.batch_size` （整数）：在威胁智能供给数据创建过程中，在批量请求中摄入的最大文档数量。默认值为10,000。

`plugins.security_analytics.threat_intel_timeout` （时间值）：创建和删除威胁智能供稿数据的超时值。默认值为30秒。

要了解有关静态和动态设置的更多信息，请参阅[配置OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)

