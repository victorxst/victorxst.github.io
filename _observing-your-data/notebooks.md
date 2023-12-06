---
layout: default
title: 笔记本
nav_order: 50
redirect_from: 
  - /dashboards/notebooks/
  - /observability-plugin/notebooks/
has_children: false
---

# 笔记本

OpenSearch仪表板笔记本电脑是一个接口，可让您轻松地将代码段，实时可视化和叙事文本组合到单个笔记本接口中。

笔记本电脑可让您通过运行不同的可视化来进行交互性探索数据，您可以与团队成员共享这些可视化，以在项目上进行协作。

笔记本是一个由两个元素组成的文档：代码块（Markdown/SQL/PPL）和可视化。选择多个时间表以比较和对比度可视化。

您也可以生成[报告]({{site.url}}{{site.baseurl}}/dashboards/reporting/) 直接来自笔记本。

常见用例包括创建验尸报告，设计运行书，构建实时基础架构报告和编写文档。

OpenSearch仪表板中的租户是保存笔记本和其他OpenSearch仪表板对象的空间。有关更多信息，请参阅[OpenSearch仪表板并发-租赁]({{site.url}}{{site.baseurl}}/security/multi-tenancy/tenant-index/)。
{:.note}


## 开始笔记本

要开始，请选择**笔记本** 在OpenSearch仪表板中。


### 步骤1：创建笔记本

笔记本是用于创建报告的接口。

1. 选择**创建笔记本** 并输入描述性名称。
1. 选择**创造**。

选择**动作** 重命名，复制或删除笔记本。

![创建笔记本]({{site.url}}{{site.baseurl}}/images/create_notebook.gif)

### 步骤2：添加一个段落

段落结合了用于描述数据的代码块和可视化。

#### 添加代码块

代码块支持Markdown，SQL和PPL语言。

使用第一行中指定输入语言`%[language type]` 句法。
例如，输入`%md` 对于Markdown，`%sql` 对于SQL，`%ppl` 对于ppl。

##### 样本降价块

```
%md
Add in text formatted in markdown.
```

![降价段落]({{site.url}}{{site.baseurl}}/images/markdown_notebooks.gif)

##### 样品SQL块

```sql
%sql
Select * from opensearch_dashboards_sample_data_flights limit 20;
```

![SQL段落]({{site.url}}{{site.baseurl}}/images/sql_notebooks.gif)

##### 样品PPL块

```
%ppl
source=opensearch_dashboards_sample_data_logs | head 20
```

![PPL段落]({{site.url}}{{site.baseurl}}/images/ppl_notebooks.gif)


#### 添加可视化

1. 要添加可视化，请选择**添加段落** 并选择**可视化**。
1. 在**标题**，选择可视化并选择一个日期范围。您可以选择多个时间表来比较和对比鲜明的可视化。
1. 要运行并保存段落，请选择**跑步**。

![可视化段落]({{site.url}}{{site.baseurl}}/images/visualization_notebooks.gif)

## 段落动作

您可以在段落上执行以下操作：

- 在报告的顶部添加新段落。
- 在报告的底部添加一个新段落。
- 同时运行所有段落。
- 清除所有段落的输出。
- 删除所有段落。

![样本笔记本]({{site.url}}{{site.baseurl}}/images/paragraphs_notebooks.gif)

## 样本笔记本

我们准备了以下样品笔记本，以展示各种用例：

- 使用SQL查询OpenSearch仪表板样本飞行数据。
- 使用PPL查询OpenSearch仪表板样本Web日志数据。
- 使用PPL和可视化来执行示例根本原因事件分析在OpenSearch仪表板样本Web日志数据上。

要添加示例笔记本，请选择**动作** 并选择**添加示例笔记本**。

![样本笔记本]({{site.url}}{{site.baseurl}}/images/sample_notebooks.gif)

## 创建报告

您可以使用笔记本来创建PNG和PDF报告：

1. 在顶部菜单栏中，选择**报告行动**。
1. 你可以选择**下载PDF** 或者**下载PNG**。

   报告在后台产生异步，可能需要几分钟，具体取决于报告的大小。当您的报告准备下载时，会出现通知。

1. 创建时间表-基于报告，选择**创建报告定义**。有关创建报告定义的步骤，请参见[使用定义创建报告]({{site.url}}{{site.baseurl}}/dashboards/reporting#creating-reports-using-a-definition)。
1. 要查看所有报告，请选择**查看所有报告**。

![报告笔记本]({{site.url}}{{site.baseurl}}/images/report_notebooks.gif)

