---
layout: default
title: 触发器
nav_order: 40
grand_parent: 警报
parent: 监视器
---

# 触发器

您如何创建触发器会取决于创建监视器时选择的监视方法。监视方法是**视觉编辑器**，**提取查询编辑器**， 和**异常检测器**。在以下各节中了解有关每种类型的更多信息。

## 创建触发器

创建触发器：

1. 在里面**创建监视器** 窗口，选择**添加触发器**。
2. 输入触发名称，严重性级别和触发条件。严重性水平从1（最高）到5（最低）不等，有助于管理警报。例如，严重程度较高的触发器（例如，1或2）可能会通知特定个人，而严重程度较低的触发器（4或5）可能会通知聊天室。触发条件包括"IS ABOVE," "IS BELOW," 和"IS EXACTLY."

询问-级别监视器一次与查询结果一起运行触发脚本，然后存放-级别监视器在每个存储桶上运行触发脚本。创建最适合监视方法的触发器。要运行多个脚本，您必须创建多个触发器。
{: .note}

## 视觉编辑器

查询-Level Monitor的触发条件，指定创建监视器时选择的聚合和时间范围的阈值（例如，"IS BELOW 1,000" 或者"IS EXACTLY 10"）。随着您增加或降低阈值，线路上下移动。一旦越过这条线，触发器就会评估`true`。

对于水桶-级别监视器，您必须为聚合和时间范围指定阈值和值。您最多可以使用五个条件来完善扳机。可选地，您还可以使用关键字过滤器来过滤索引中的特定字段。

用于文档-级别监视器，使用代表由逻辑连接的多个查询的标签`OR` 操作员。创建一个倍数-查询触发器：

1. 选择**每个文档监视器**。
2. 选择一个数据源。
3. 输入查询名称和字段信息。例如，设置查询以搜索`region` 带有任一操作员的字段"is" 或者"is not" 和价值"us-west-2"。
4. 选择**添加标签** 并输入标签名称。
5. 通过选择第二个查询**添加另一个查询** 并在其中添加相同的标签。

现在，您可以创建触发条件并指定标签名称。这会创建一个组合触发器，该组合触发器检查两个都包含相同标签的查询。监视器用逻辑检查两个查询`OR` 操作，如果满足任何查询条件，则会生成警报通知。

## 提取查询编辑器

查询-等级监视器，指定一个返回的无痛脚本`true` 或者`false`。默认的OpenSearch脚本语言是无痛的，具有类似于Groovy的语法。

触发条件脚本围绕`ctx.results[0]` 变量，对应于提取查询响应。例如，脚本可能参考`ctx.results[0].hits.total.value` 或者`ctx.results[0].hits.hits[i]._source.error_code`。

返回值的`true` 意味着已经满足触发条件，并且触发器应运行其动作。使用**跑步** 按钮。

这**信息** 链接旁边**触发条件** 包含有关查询可用的变量和结果的有用摘要。
{: .tip }

桶-级别监视器要求您在触发条件下指定更多信息。至少您必须具有以下字段：

- `buckets_path`：将变量名称映射到脚本中使用的指标。
- `parent_bucket_path`：通往多的路径-铲斗聚集。路径可以包括单个-铲斗聚集，但最后一个聚合必须是多数-桶。例如，如果您有一个管道，例如`agg1>agg2>agg3`，`agg1` 和`agg2` 是单身-铲斗聚集，但是`agg3` 必须是一个多人-铲斗聚集。
- `script`：OpenSearch的脚本运行以评估是否触发任何警报。

以下是一个示例脚本：

```json
{
  "buckets_path": {
    "count_var": "_count"
  },
  "parent_bucket_path": "composite_agg",
  "script": {
    "source": "params.count_var > 5"
  }
}
```

映射后`count_var` 变量为`_count` 指标，您可以使用`count_var` 在您的脚本和参考中`_count` 数据。这`composite_agg` 是通往多人的途径-铲斗聚集。

## 异常检测器

使用异常检测方法：

1. 为了**触发类型**， 选择**探测器等级和信心**。
2. 指定**异常等级条件** 对于创建监视器时选择的聚合和时间范围，例如"IS ABOVE 0.7" 或者"IS EXACTLY 0.5." *异常等级 *是0到1之间的数字，指示数据点异常。
3. 指定**异常置信条件** 对于您之前选择的聚合和时间范围，"IS ABOVE 0.7" 或者"IS EXACTLY 0.5." *异常置信度 *是对报告异常等级匹配预期异常等级的概率的估计。随着您的增加并降低阈值，该线会上下移动。一旦越过这条线，触发器就会评估`true`。

### 示例脚本


```groovy
// Evaluates to true if the query returned any documents
ctx.results[0].hits.total.value > 0
```

```groovy
// Returns true if the avg_cpu aggregation exceeds 90
if (ctx.results[0].aggregations.avg_cpu.value > 90) {
  return true;
}
```

```groovy
// Performs some crude custom scoring and returns true if that score exceeds a certain value
int score = 0;
for (int i = 0; i < ctx.results[0].hits.hits.length; i++) {
  // Weighs 500 errors 10 times as heavily as 503 errors
  if (ctx.results[0].hits.hits[i]._source.http_status_code == "500") {
    score += 10;
  } else if (ctx.results[0].hits.hits[i]._source.http_status_code == "503") {
    score += 1;
  }
}
if (score > 99) {
  return true;
} else {
  return false;
}
```

#### 触发变量

多变的| 数据类型| 描述
:--- | :--- | :---
`ctx.trigger.id` | 细绳| 触发ID。
`ctx.trigger.name` | 细绳| 触发名称。
`ctx.trigger.severity` | 细绳| 触发严重性。
`ctx.trigger.condition`| 目的| 包含创建显示器时使用的无痛脚本。
`ctx.trigger.condition.script.source` | 细绳| 用于定义脚本的语言。必须无痛。
`ctx.trigger.condition.script.lang` | 细绳| 用于定义触发器的脚本。
`ctx.trigger.actions`| 大批| 一个带有一个元素的数组，其中包含有关监视器需要触发的操作的信息。

#### 其他变量

多变的| 数据类型| 描述
:--- | :--- ：：---
`ctx.results` | 大批| 一个具有一个元素的数组（`ctx.results[0]`）。包含查询结果。如果触发器无法检索结果，则此变量为空。看`ctx.error`。
`ctx.last_update_time` | 毫秒| unix epoch最后更新的显示器的时间。
`ctx.periodStart` | 细绳| Unix时间戳在触发警报的期间开始。例如，如果监视器每10分钟运行一次，则可能从10:40开始，并在10:50结束。
`ctx.periodEnd` | 细绳| 警报触发的时期结束。
`ctx.error` | 细绳| 如果触发器无法检索结果或无法评估，则显示错误消息，通常是由于编译错误或无效指针异常。否则为无效。
`ctx.alert` | 目的| 当前的主动警报（如果存在）。包括`ctx.alert.id`，`ctx.alert.version`， 和`ctx.alert.isAcknowledged`。null如果没有警报处于活动状态。仅与查询一起使用-电平监视器。
`ctx.dedupedAlerts` | 目的| 已触发的警报。OpenSearch保留现有警报，以防止插件创建无数数字的同一警报。仅带有水桶-电平监视器。
`ctx.newAlerts` | 目的| 新创建的警报。仅带有水桶-电平监视器。
`ctx.completedAlerts` | 目的| 不再持续的警报。仅带有水桶-电平监视器。
`bucket_keys` | 细绳| 逗号-监视器的存储键钥匙值的分开列表。仅适用于`ctx.dedupedAlerts`，`ctx.newAlerts`， 和`ctx.completedAlerts`。访问`ctx.dedupedAlerts[0].bucket_keys`。
`parent_bucket_path` | 细绳| 触发警报的桶的父级路径。访问`ctx.dedupedAlerts[0].parent_bucket_path`。

