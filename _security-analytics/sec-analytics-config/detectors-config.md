---
layout: default
title: 创建检测器
parent: 设置安全分析
nav_order: 15
---

# 创建检测器

安全分析提供了监视和响应广泛安全威胁的选项和功能。探测器是确定要寻找的内容以及如何应对这些威胁的重要组成部分。本节涵盖了他们的创建和配置。

有关使用现有检测规则的信息，请参见[创建检测规则]({{site.url}}{{site.baseurl}}/security-analytics/usage/detectors/)。

---
## Step 1. Define a detector

您可以通过命名检测器，然后选择数据源和检测器类型来定义新的检测器。定义检测器后，您可以配置字段映射，创建检测器时间表并设置警报。

定义检测器：

1. 在**安全分析** 主页或**探测器** 页面，选择**创建检测器**。
1. 给探测器一个名称，并选择为描述。
1. 在里面**数据源** 部分，为日志数据选择一个或多个来源。使用星号（*）指示通配符模式。选择多个数据源时，它们的日志必须为相同的类型。我们建议为不同的日志类型创建单独的检测器。
   
1. 在里面**检测** 部分，为数据源选择日志类型。有关支持的日志类型列表，请参见[{stite.url}}} {{site.baseurl}}/Security/Security（{site.url}}}-分析/秒-分析-config/log-类型/）。要创建自己的日志类型，请参见[创建自定义日志类型]（{{site.url}}} {{site.baseurl}}/security-分析/秒-分析-config/custom-日志-类型/）。
    
    当您选择时`network`，`cloudtrail`， 或者`s3` 作为日志类型，系统会自动创建检测器仪表板。仪表板为检测器提供可视化，并可以提供安全性-对日志源数据的相关洞察力。有关可视化的更多信息，请参见[构建数据可视化]（{{site.url}}} {{site.baseurl}}}/dashboards/vutionize/viz-指数/）。
    
     
1. 扩张**检测规则** 显示所选日志类型的可用检测规则列表。最初，默认情况下选择所有规则。以下示例显示了与**视窗** 日志类型。

    <img src ="{{site.url}}{{site.baseurl}}/images/Security/detector-rules.png" alt ="Selecting threat detector log type to auto-populate rules" width="100%">

    您可以在查看规则时执行以下操作：
    
    *使用向左的切换**规则名称** 选择或取消选择规则。
    * 使用**规则严重性** 和**来源** 下拉列表以过滤您要选择的规则。
    * 使用**搜索** 栏以搜索特定规则。
    
    要快速选择一个或多个已知的规则并驳回其他规则，请首先通过关闭所有规则**规则名称** 切换，然后搜索您的目标规则名称，然后通过打开其切换来单独选择每个目标。
    {: .tip }

1. 查看现场映射。字段映射允许系统将事件数据从日志准确地传递到检测器，然后使用数据触发警报。有关现场映射的更多信息，请参阅**关于现场映射** 稍后在此主题中。
    
1. 在里面**探测器时间表** 部分，为运行检测器的频率创建时间表。指定时间单位和一个相应的数字来设置间隔。下图显示检测器每3分钟运行一次。
    
    <img src ="{{site.url}}{{site.baseurl}}/images/Security/detector-schedule.png" alt ="Detector schedule settings to determine how often the detector runs" width="40%">
    
1. 选择**下一个**。这**设置警报** 出现页面并显示警报触发器的设置。


---
## 步骤2.设置警报

创建检测器的第二步涉及设置警报。警报被配置为创建触发器，该触发器与一组检测规则标准匹配时，请发送有关可能的安全事件的通知。您可以在任何组合中选择规则名称，规则严重性和标签来定义触发器。定义触发器后，警报设置使您可以选择要通知的频道，并提供了为通知提供消息的选项。

在检测器开始生成发现之前，至少需要一个警报条件。
{: .note}

您还可以从**发现** 窗户。查看如何从**发现** 窗口，请参阅[调查结果列表]({{site.url}}{{site.baseurl}}/security-analytics/usage/findings/#the-findings-list)。添加其他警报的最终选项是编辑检测器并导航到**警报触发器** 选项卡，您可以在其中编辑现有警报并添加新警报。有关详细信息，请参阅[编辑检测器]({{site.url}}{{site.baseurl}}/security-analytics/usage/detectors/#editing-a-detector)。

要设置检测器的警报，请继续执行以下步骤：

1. 在里面**触发名称** 框，可选输入触发器的名称或编辑默认名称。
1. 要定义警报的规则匹配，请选择安全规则，严重性级别和标签。
    
    <img src ="{{site.url}}{{site.baseurl}}/images/Security/alert_rules.png" alt ="Defining an alert" width="70%">

    *选择一个将触发警报的规则或多个规则。将光标放在**规则名称** 框并输入一个名称以搜索它。要删除规则名称，请选择**X** 在名字旁边。要删除所有规则名称，请选择**X** 在下拉列表的下箭头旁边。

    <img src ="{{site.url}}{{site.baseurl}}/images/Security/rule_name_delete.png" alt ="Deletes all selected rules" width="45%">

    *选择一个或多个规则严重性水平作为警报条件。
    *从标签列表中选择以将作为警报的条件包含。

1. 要定义警报的通知，请分配警报严重性，选择通知通道，然后自定义为警报生成的消息。

    <img src ="{{site.url}}{{site.baseurl}}/images/Security/alert_notify.png" alt ="Notification settings for the alert" width="45%">

    *为警报分配一定程度的严重性，以使接收者表明其紧迫性。
    *从**选择通知的频道** 下拉列表。示例包括Slack，Chime或电子邮件。要创建一个新频道，请选择**管理频道** 链接到该领域的右侧。这**频道** 有关通知的页面在新选项卡中打开，您可以在其中编辑和创建新频道。有关通知的更多信息，请参阅[通知]({{site.url}}{{site.baseurl}}/observing-your-data/notifications/index/) 文档。
    * 扩张**显示通知消息** 显示消息首选项。消息主体和消息主体填充了有关当前警报配置的详细信息。您可以编辑这些文本字段以自定义消息。在消息主体文本框下方，您可以选择**生成消息** 在消息中填充更多详细信息，例如规则名称，规则严重性级别和规则标签。
    * 选择**添加另一个警报触发器** 配置额外的警报。

1. 在上前字段中配置条件后，选择**创建检测器** 在较低-屏幕的右角。

## 集成警报插件工作流程

默认情况下，当您创建威胁检测器时，系统会自动创建复合监视器并触发警报插件的工作流程。检测器的规则被转换为警报插件监视器的搜索查询，并根据从检测器的配置中得出的时间表执行其查询。

您可以通过启用或禁用工作流功能来更改自动生成的复合监视器的行为`plugins.security_analytics.enable_workflow_usage` 环境。此设置是使用[集群设置API]({{site.url}}{{site.baseurl}}/api-reference/cluster-api/cluster-settings/)。

有关复合监视器及其工作流程的更多信息，请参阅[复合监视器]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/composite-monitors/)。

---

## About field mappings
第一步中指定的数据源（日志索引），日志类型和检测规则确定哪些字段可用于映射。例如，当"Windows logs" 被选为日志类型，此参数以及特定的检测规则确定可用于映射的检测字段名称列表。同样，所选数据源确定可用于映射的日志源字段名称的列表。

该系统使用预包装的Sigma规则来创建检测器。它可以自动将特定日志类型的重要字段映射到Sigma规则中的相应字段。字段映射步骤介绍了自动映射字段的视图，同时还提供了自定义，更改或添加新的字段映射的选项。当检测器包含自定义规则时，您可以按照此步骤手动映射检测器规则字段名称将源字段名称。

由于系统具有自动映射字段名称的能力，因此此步骤是可选的。但是，可以在检测器字段和日志源字段之间映射的字段越多，生成的发现的准确性就越大。
 
### A note on field names
如果您选择执行手动字段映射，则应熟悉日志索引中的字段名称，并了解这些字段中包含的数据。如果您了解索引中的日志源字段，则映射通常是一个简单的过程。

安全分析利用预包装的Sigma规则进行安全事件检测。因此，字段名称源自Sigma规则字段标准。但是，为了使它们易于识别，我们已经基于开放的Sigma规则字段创建了别名-源弹性通用模式（ECS）规范。这些别名规则字段名称是这些步骤中使用的字段名称。他们出现在**检测器字段名称** 映射表的列。

尽管ECS规则字段名称在很大程度上是自我的-解释性，您可以在GitHub Security Analytics存储库中找到所有受支持的日志类型的Sigma规则字段名称的预定量映射到ECS规则字段名称。导航到[osmappings]（https://github.com/opensearch-项目/安全性-分析/树/Main/src/main/resources/osmapping）文件夹，然后选择特定日志类型的文件。例如，要查看与Windows日志类型的ECS规则字段相对应的Sigma规则字段，请选择[`windows_logtype.json` 文件]（https://github.com/opensearch-项目/安全性-分析/blob/main/src/main/resources/osmapping/windows_logtype.json）。这`raw_field` 文件中的值表示映射中的Sigma规则字段名称。

### Amazon Security Lake logs

[亚马逊安全湖]（https://docs.aws.amazon.com/security-湖泊/最新/USERGUIDE/什么-是-安全-lake.html）将安全日志和事件数据转换为[开放网络安全架构框架]（https://docs.aws.aws.amazon.com/security-湖泊/最新/USERGUIDE/OPEN-网络安全-模式-Framework.html）（OCSF）正常化数据并促进其管理。OpenSearch支持以OCSF格式从安全湖中摄入日志数据，安全分析可以自动将OCSF映射到ECS（默认字段）-映射模式）。

可以用作检测器创建的日志源的安全湖日志类型包括CloudTrail，Route 53和VPC流。鉴于该路线53是捕获DNS活动的日志，应将其日志类型指定为**DNS** [定义检测器]（定义检测器）（#步-1-定义-A-探测器）。此外，由于可以想象可以以原始格式和OCSF捕获诸如CloudTrail日志之类的日志，因此以将这些日志保持分开且易于识别的方式命名索引是很好的做法。在与安全分析相关的任何API中指定索引名称时，这将变得有用。

要以原始格式或OCSF显示日志索引的字段，请使用[get mappings view]（{{site.url}}} {{site.baseurl}}/security-分析/API-工具/映射-api/#得到-映射-查看）API并指定索引`index_name` 请求字段。
{: .tip }

### Automatically mapped fields

选择数据源和日志类型后，系统将尝试在日志和规则字段之间自动映射字段。切换到**映射字段** 选项卡以显示这些映射的列表。当字段名称彼此相似时，系统可以成功匹配两者，如下图所示。

<img src ="{{site.url}}{{site.baseurl}}/images/Security/automatic-mappings.png" alt ="Field mapping example for automatic mappings" width="85%">

尽管这些自动匹配通常是可靠的，但最好查看映射**映射字段** 表并验证它们是否正确且按预期匹配。如果发现似乎不准确的映射，则可以使用下拉列表来搜索并选择正确的字段名称。有关匹配字段名称的更多信息，请参见以下部分。

### Available fields

未自动映射的字段名称出现在**可用字段** 桌子。在此表中，您可以手动将检测规则字段映射到数据源字段，如下图所示。

<img src ="{{site.url}}{{site.baseurl}}/images/Security/pending-mappings.png" alt ="Field mapping example for available mappings" width="85%">

在映射字段时，请考虑以下内容：
* 这**检测规则字段** 列列出了基于与所选日志类型关联的所有预包装规则的字段名称。
* 这**数据源字段** 列包括每个检测器字段的下拉列表。每个下拉列表包含从日志索引提取的字段名称。
*要将检测器字段名称映射到日志源字段名称，请使用下拉箭头打开日志源字段列表，然后从列表中选择日志字段名称。要在日志字段列表中搜索名称，请在**选择数据源字段** 盒子。
  
*选择日志源字段名称并映射到检测器字段名称，该图标是**地位** 右侧的列从警报图标更改为复选标记。
*在字段名称之间进行尽可能多的匹配，以完成检测器和日志源字段的准确映射。

## What's next

如果您准备查看新检测器生成的发现，请参见[使用调查结果]（{{site.url}}} {{site.baseurl}}/security-分析/用法/发现/）部分。如果您想在使用调查结果之前导入规则或设置自定义规则，请参见[使用检测规则]（{{site.url}}} {{site.baseurl}}/Security-分析/用法/规则/）部分。

要配置安全分析以识别整个系统中不同日志中发生的事件之间的相关性，请参见[使用相关规则]（{{site.url}}} {{site.baseurl}}/security-分析/用法/规则/）。

