---
layout: default
title: 设置安全分析
nav_order: 10
has_children: true
has_toc: false
redirect_from:
  - /security-analytics/sec-analytics-config/
---

# 设置安全分析

在安全分析可以开始生成调查结果和发送警报之前，管理员必须创建检测器并将日志数据提供给系统。一旦探测器能够生成发现，您就可以罚款-调整您的警报以关注特定感兴趣领域。以下步骤概述了在安全分析中设置组件的基本工作流程。

1. 创建威胁探测器和警报，并摄入日志数据。看[创建检测器]({{site.url}}{{site.baseurl}}/security-analytics/sec-analytics-config/detectors-config/) 了解更多信息。
1. 考虑[创建相关规则]({{site.url}}{{site.baseurl}}/security-analytics/sec-analytics-config/correlation-config/) 确定整个系统中不同日志中发生的事件之间的连接和可能发生的威胁。
1. 检查从检测器输出生成的发现，并创建任何其他警报。
1. 如果需要，请创建自定义规则，以更好地焦点检测器-系统中的优先关注。看[创建检测规则]({{site.url}}{{site.baseurl}}/security-analytics/usage/rules/#creating-detection-rules) 了解更多信息。

## 导航到安全分析

1. 要开始，请在仪表板主页上选择顶部菜单，然后选择**安全分析**。显示安全分析的概述页面。
1. 从页面左侧的选项中，选择**探测器** 开始创建检测器。

<img src="{{site.url}}{{site.baseurl}}/images/Security/secanalytics-det-nav.png" alt="Navigating to create a detector page" width="70%">

