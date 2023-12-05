---
layout: default
title: 处理结果
parent: 使用安全分析
nav_order: 35
---

# 与调查结果一起工作

这**发现** 窗口包括用于查看和使用发现的功能。这两个主要特征是：
*带有计数，日期和日志类型或规则严重性的发现信息的条形图。
* 这**发现** 按时间安排的列表，查找ID，规则名称和其他详细信息。

你可以选择**刷新** 随时刷新有关**发现** 页。

---
## The Findings graph

发现图可以通过日志类型或规则严重性显示发现。使用**通过...分组** 下拉列表以指定日志类型或规则严重性。

要指定您希望显示的日期范围，请先选择日历下拉列表。日期选择器窗口打开。

<img src ="{{site.url}}{{site.baseurl}}/images/Security/find-date-pick.png" alt ="Date selector for findings graph" 宽度="55%">

您可以使用**快速选择** 设置以指定确切的时间窗口。
*选择**最后的** 或者**下一个** 在第一个下拉列表中，以设置当前设置后面或当前设置之前的时间窗口。
*在第二个下拉列表中选择一个数字以定义范围的值。
*在第三个下拉列表中选择一个时间单元。可用的选项是秒，分钟，小时，天，几周，几个月和数年。
选择**申请** 将日期的范围应用于图。如下图所示，有关图表的信息相应地变化。

<img src ="{{site.url}}{{site.baseurl}}/images/Security/quickset.png" alt ="Quick select settings example" 宽度="40%">

您可以使用左和右箭头将时间窗口移动到当前日期范围之后或在当前日期范围之前。当您使用这些箭头时，启动日期和末端日期出现在日期范围字段中。然后，您可以选择每个设置一个绝对，相对或当前日期和时间。对于绝对和相对变化，请选择**更新** 应用更改。

<img src ="{{site.url}}{{site.baseurl}}/images/Security/date-pick.png" alt ="Altering date range" 宽度="55%">

作为替代方案，您可以在**常用** 部分（请参阅日历下拉列表的前面图像），以方便地设置时间窗口。选项包括日期范围，例如**今天**，，，，**昨天**，，，，**本星期**， 和**一周**。

当选择了一个常用的时间窗口之一时，您可以选择**显示日期** 在日期范围字段中以填充日期范围。在此之后，您可以选择“开始日期”或“结束日期”以指定绝对，相对或当前日期和时间设置。对于绝对和相对变化，请选择**更新** 应用更改。

作为另一种选择，您可以从**最近使用的日期范围** 部分返回到上一个设置。

---
## 调查结果列表

这**发现** 列表根据发现的时间，查找ID，生成发现的规则名称，捕获发现的检测器以及其他细节（如下图中所示）显示所有发现。

<img src="{{site.url}}{{site.baseurl}}/images/Security/finding-list.png" alt="A list of all findings" width="85%">

使用**规则严重性** 下拉列表以按严重性过滤发现列表。使用**日志类型** 下拉列表以按日志类型过滤列表。

这**动作** 列包括每个发现的两个选项：
*对角线箭提供了一种打开的方法[**查找细节**](#finding-details) 窗格，根据创建检测器时定义的参数描述发现，并包括生成发现的文档。
*钟声图标允许您打开**创建检测器警报触发器** 窗格，您可以根据需要快速设置特定发现和修改规则及其条件的警报。
有关设置警报的信息，请参阅[步骤2.设置警报]({{site.url}}{{site.baseurl}}/security-analytics/sec-analytics-config/detectors-config/#step-2-set-up-alerts) 在检测器创建文档中。

### 查找细节

列表中的每个发现还包括一个**查找ID**。除了使用对角线箭头**动作**，您可以选择ID以打开**查找细节** 窗格。一个例子**查找细节** 如下图所示。

<img src="{{site.url}}{{site.baseurl}}/images/Security/findings1.png" alt="Finding details pane" width="60%">

#### 查看周围的文件

这**查找细节** 窗格包含有关发现的特定信息，包括生成发现的文档。为了调查导致发现或遵循发现的一系列事件，您可以选择**查看周围的文件** 在**发现** 面板并查看之前或关注它的其他文档。

1. 打开**查找细节** 通过选择**查找ID** 在里面**发现** 列表。
1. 在里面**文件** 部分，选择**查看周围的文件**。如果文档已经存在索引模式，则**发现** 面板打开并显示文档。如果不存在索引模式，**创建索引模式以查看文档** 窗口打开并提示您创建一个索引模式，如下图所示。

    <img src ="{{site.url}}{{site.baseurl}}/images/Security/findings2.png" alt ="popup window prompting users to create an index pattern" 宽度="60%">

1. 在里面**创建索引模式以查看文档** 窗口，索引模式名称自动填充。从用于确定日志事件的时机的日志索引中输入适当的时间字段。选择**创建索引模式**。这**创建索引模式以查看文档** 确认窗口打开。
1. 选择**查看周围的文件** 在确认窗口中。这**发现** 面板打开，如下图所示。

    <img src ="{{site.url}}{{site.baseurl}}/images/Security/findings4.png" alt ="Discover panel with surrounding documents" 宽度="85%">
    
这**发现** 面板显示以突出显示的背景生成发现的文档。还显示了事件之前或之后出现的其他文档。

有关与之合作的详细信息**发现** 在OpenSearch仪表板中，请参阅[探索数据]({{site.url}}{{site.baseurl}}/dashboards/discover/index-discover/)。

#### 查看相关的发现

要查看发现与其他发现的相关性，请选择**相关性** 标签。相关性是表达涉及多种日志类型的特定威胁场景的发现之间的关系。信息中的信息**相关的发现** 表显示了生成相关发现的时间，发现的ID，用于生成发现的日志类型，其威胁严重性和相关评分---衡量其靠近参考发现的---如下图所示。

<img src="{{site.url}}{{site.baseurl}}/images/Security/corr-details-findings.png" alt="A table of correlated findings with respect to the reference finding" width="60%">

您可以选择**查看相关图** 可视化发现之间的相关性。有关使用相关图的更多信息，请参见[使用相关图]({{site.url}}{{site.baseurl}}/security-analytics/usage/correlation-graph/)。

