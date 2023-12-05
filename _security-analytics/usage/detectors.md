---
layout: default
title: 使用探测器
parent: 使用安全分析
nav_order: 30
---

# 使用探测器

创建检测器后，它将出现在威胁探测器页面上，并保存到系统中。然后，您可以为每个检测器执行许多操作，从编辑其详细信息到更改其状态。有关可用操作的描述，请参见以下各节。

<img src="{{site.url}}{{site.baseurl}}/images/Security/threat-detector.png" alt="Threat detector page" width="60%">

---
## Threat detector list

威胁探测器的列表包括搜索栏，**地位** 下拉列表，**日志类型** 下拉列表。
*使用搜索栏通过检测器名称过滤。
*选择**地位** 下拉列表以通过活动和非活动状态在列表中过滤探测器。
*选择**日志类型** 下拉列表以通过列表中出现的任何日志类型进行过滤探测器（选项取决于列表中存在的检测器及其日志类型）。

### Editing a detector

要编辑检测器，请先在列表的“检测器名称”列中选择指向检测器的链接。检测器的详细信息窗口打开并显示有关检测器配置的详细信息。

<img src ="{{site.url}}{{site.baseurl}}/images/Security/detector-details.png" alt ="Detector details window for editig the detector" 宽度="50%">

*在上部-窗口的左侧部分，详细信息窗口显示了检测器的名称及其状态，即活动或无活动。
*在上部-窗口的右角，您可以选择**查看警报** 转到警报窗口或**查看发现** 转到“发现”窗口。您也可以选择**动作** 为检测器执行动作。请参阅[检测器操作]（{{stite.url}}} {{site.baseurl}}/Security-分析/用法/检测器/#探测器-动作）。
*在窗口的下部，选择**编辑** 探测器详细信息或检测规则的按钮以进行相应更改。
*最后，您可以选择**现场映射** 选项卡以编辑检测器的字段映射，或选择**警报触发器** 选项卡以对与检测器关联的警报进行编辑。

<img src ="{{site.url}}{{site.baseurl}}/images/Security/detector-details2.png" alt ="Field mappings and Alert triggers tabs" 宽度="40%">

选择后**警报触发器** 选项卡，您还可以选择通过选择检测器添加其他警报**添加另一个警报条件** 在页面底部。
{: .tip }

---
## 探测器动作

威胁检测器的操作使您可以停止并启动检测器或删除检测器。要启用操作，请先在列表中的一个或多个检测器旁边选择复选框。

<img src="{{site.url}}{{site.baseurl}}/images/Security/detector-action.png" alt="Threat detector actions" width="50%">

### 更改检测器状态

1.  在列表中选择您想更改其状态的检测器或检测器。这**动作** 下拉列表启用了。
1.  根据检测器当前处于活动状态还是不活动，请选择**停止检测器** 或者**启动检测器**。片刻之后，检测器的状态变化出现在检测器列表中，以无效或有效。

### 删除检测器

1. 选择要删除的列表中的检测器或检测器。这**动作** 下拉列表启用了。
1. 选择**删除** 在下拉列表中。删除检测器弹出窗口打开，并要求您验证要删除检测器或检测器。
1. 选择**取消** 拒绝行动。选择**删除检测器** 从列表中删除检测器或探测器。

## 相关文章
[创建检测器]（{{stite.url}}} {{site.baseurl}}/security-分析/秒-分析-配置/检测器-config/[创建检测器]({{site.url}}{{site.baseurl}}/security-analytics/sec-analytics-config/detectors-config/)[创建检测器]（{{stite.url}}} {{site.baseurl}}/security-分析/秒-分析-配置/检测器-config/

