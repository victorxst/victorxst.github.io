---
layout: default
title: 创建相关规则
parent: 设置安全分析
nav_order: 17
---

# 创建相关规则

相关规则允许您通过匹配在不同日志类型中发生的威胁事件的签名来定义涉及基础架构中多个系统的威胁方案。一旦规则包含至少两个不同的日志源以及定义预期威胁场景的首选字段和字段值，相关引擎可以查询相关规则中指定的索引并确定发现之间的任何相关性。

---
## Configuring rules

规则配置中至少具有两个数据源是在基础架构中不同系统之间建立连接并识别相关性的基础。因此，每个相关规则至少需要两个查询。但是，您可以包含两个以上的查询，以更好地定义威胁场景并寻找多个系统之间的相关性。请按照以下步骤创建相关规则：

1. 首先选择**安全分析** 在OpenSearch仪表板主菜单中。然后选择**相关规则** 从屏幕左侧的“安全分析”菜单中。这**相关规则** 显示页面，如下图所示。
   
   <img src ="{{site.url}}{{site.baseurl}}/images/Security/sec-analytics/create-corr-rule.png" alt ="The correlation rules page" width="85%">

1. 选择**创建相关规则**。这**创建相关规则** 窗口打开。
1. 在里面**相关规则细节** 字段，输入该规则的名称，如下图所示。
  
   <img src ="{{site.url}}{{site.baseurl}}/images/Security/sec-analytics/corr-rule-config1.png" alt ="The correlation rule name" width="50%">

1. 这**相关查询** 字段包含两个下拉列表。在里面**选择索引** 下拉列表，为数据源指定索引或索引模式。在里面**日志类型** 下拉列表，指定与索引关联的日志类型，如下图所示。
  
   <img src ="{{site.url}}{{site.baseurl}}/images/Security/sec-analytics/corr-rule-config2.png" alt ="The data source and log type for the query" width="45%">
  
1. 在里面**场地** 下拉列表，指定日志字段。在里面**场值** 文本框，输入字段值，如下图所示。
  
   <img src ="{{site.url}}{{site.baseurl}}/images/Security/sec-analytics/corr-rule-config3.png" alt ="The field and field value for the query" width="45%">

1. 要在查询中添加更多字段，请选择**添加字段**。
1. 配置第一个查询后，重复上一步以配置第二个查询。您可以选择**添加查询** 在窗口的底部，以添加有关该规则的更多查询，如下图所示。
  
   <img src ="{{site.url}}{{site.baseurl}}/images/Security/sec-analytics/corr-rule-config4.png" alt ="A second query for the correlation rule" width="50%">

1. 规则完成后，选择**创建相关规则** 在较低-窗户的右角。OpenSearch创建了一个新规则，屏幕返回到**相关规则** 窗口，新规则出现在相关规则表中。要编辑规则，请在**姓名** 柱子。这**编辑相关规则** 窗口打开。

---
## 设置一个时间窗口

群集设置API允许您在设定的时间窗口中关联发现。例如，如果您的时间窗口为三分钟，则系统仅在彼此之间发生三分钟内就将其关联在威胁方案中定义的发现。默认情况下，时间窗口为五分钟。有关集群设置API的更多信息，请参阅[集群设置]({{site.url}}{{site.baseurl}}/api-reference/cluster-api/cluster-settings/)。

### 示例请求

以下呼叫将时间窗口设置为两分钟：

```json
PUT /_cluster/settings
{
  "transient": {
    "plugins.security_analytics.correlation_time_window": "2m"
  }
}
```
{% include copy-curl.html %}

---
## Next steps

创建检测器和相关规则后，您可以使用相关图来观察来自不同日志源发现之间的相关性。有关使用相关图的信息，请参见[使用相关图]（{{site.url}}} {{site.baseurl}}/security-分析/用法/相关性-图形/）。


