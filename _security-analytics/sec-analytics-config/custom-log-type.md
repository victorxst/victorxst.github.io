---
layout: default
title: 创建自定义日志类型
parent: 设置安全分析
nav_order: 18
---


# 创建自定义日志类型

日志类型表示安全分析中用于威胁检测的数据源。除了标准[日志类型]({{site.url}}{{site.baseurl}}/security-analytics/sec-analytics-config/log-types/) 在安全分析的支持下，您可以为威胁检测器创建自定义日志类型。

## 创建自定义日志类型

创建自定义日志类型：
1. 从仪表板中，选择**OpenSearch插件** >**安全分析**，然后选择**探测器** >**日志类型**。
1. 选择**创建日志类型**。
1. 输入名称，并选择为日志类型的描述。
   
   日志类型名称支持字符--Z（小写），0--9，连字符和下划线。
   {: .note}
   
1. 选择一个类别。类别列出[支持的日志类型]({{site.url}}{{site.baseurl}}/security-analytics/sec-analytics-config/log-types/)。
1. 选择**创建日志类型** 在较低-屏幕的右角。屏幕返回到**日志类型** 页面，新日志类型出现在列表中。请注意，新日志类型的源指示**风俗**。

## 日志类型API

要使用REST API执行自定义日志类型的操作，请参见[日志类型API]({{site.url}}{{site.baseurl}}/security-analytics/api-tools/log-type-api/)。


