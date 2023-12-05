---
layout: default
title: 查询工作台
nav_order: 125
redirect_from:
  - /search-plugins/sql/workbench/
---

# 查询工作台

查询工作台是OpenSearch仪表板中的工具。您可以使用查询工作台进行运行-要求[SQL]({{site.url}}{{site.baseurl}}/search-plugins/sql/sql/index/) 和[ppl]({{site.url}}{{site.baseurl}}/search-plugins/sql/ppl/index/) 查询，将查询转换为等效的REST API调用，并查看和保存结果不同[响应格式]({{site.url}}{{site.baseurl}}/search-plugins/sql/response-formats/)。

下图显示了OpenSearch仪表板中查询工作台接口的视图。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/query-workbench-ui.png" alt="Query Workbench interface within OpenSearch Dashboards"  width="700">

## 先决条件

在开始之前，请确保您有[索引您的数据]({{site.url}}{{site.baseurl}}/im-plugin/index/)。

对于本教程，您可以索引以下示例文档。或者，您可以使用[OpenSearch Playground](https://playground.opensearch.org/app/opensearch-query-workbench#/)，它已经预加载了索引，您可以使用这些索引来尝试查询工作台。

要索引示例文档，请发送以下内容[散装API]({{site.url}}{{site.baseurl}}/api-reference/document-apis/bulk/) 要求：

```json
PUT accounts/_bulk?refresh
{"index":{"_id":"1"}}
{"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
{"index":{"_id":"6"}}
{"account_number":6,"balance":5686,"firstname":"Hattie","lastname":"Bond","age":36,"gender":"M","address":"671 Bristol Street","employer":"Netagy","email":"hattiebond@netagy.com","city":"Dante","state":"TN"}
{"index":{"_id":"13"}}
{"account_number":13,"balance":32838,"firstname":"Nanette","lastname":"Bates","age":28,"gender":"F","address":"789 Madison Street","employer":"Quility","email":"nanettebates@quility.com","city":"Nogal","state":"VA"}
{"index":{"_id":"18"}}
{"account_number":18,"balance":4180,"firstname":"Dale","lastname":"Adams","age":33,"gender":"M","address":"467 Hutchinson Court","email":"daleadams@boink.com","city":"Orick","state":"MD"}
```
{% include copy-curl.html %}}

## 在查询工作台中运行SQL查询

请按照以下步骤了解如何使用查询工作台对您的OpenSearch数据运行SQL查询：

1. 访问查询工作台。
    - 要访问查询工作台，请转到OpenSearch仪表板并选择**OpenSearch插件** >**查询工作台** 从主菜单。

2. 运行查询。
    - 选择**SQL** 按钮。在查询编辑器中，键入A SQL表达式，然后选择**跑步** 按钮运行查询。
    
    以下示例查询检索名字，姓氏和平衡`accounts` 余额大于10,000的帐户的索引，按降序按余额进行分类：

    ```SQL
    选择
      名，
      姓，
      平衡
    从
      帐户
    在哪里
      余额> 10000
    订购
      平衡desc;
    ```
    {％include copy.html％}
    
3. 查看结果。
    - 查看结果**结果** 窗格，以表格格式显示查询输出。您可以根据需要过滤和下载结果。

   下图显示了先前的SQL查询的查询编辑器窗格和结果窗格：

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/query-workbench-query-step2.png" alt ="Query Workbench SQL query input and results output panes" 宽度="800">

4. 清除查询编辑器。
    - 选择**清除** 按钮清除查询编辑器并运行新的查询。

5. 检查查询的处理方式。
    - 选择**解释** 按钮以检查OpenSearch如何处理查询，包括所涉及的步骤和操作顺序。

    下图显示了在步骤2中运行的SQL查询的说明。

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/query-explain.png" alt ="Query Workbench SQL query explanation pane" 宽度="500">

## 在查询工作台中运行PPL查询

请按照以下步骤了解如何使用查询工作台对您的OpenSearch数据运行PPL查询：

1. 访问查询工作台。
    - 要访问查询工作台，请转到OpenSearch仪表板并选择**OpenSearch插件** >**查询工作台** 从主菜单。

2. 运行查询。
    - 选择**ppl** 按钮。在查询编辑器中，键入PPL查询，然后选择**跑步** 按钮运行查询。
    
    以下是检索的示例查询`firstname` 和`lastname` 文档的字段`accounts` 年龄大于年龄的索引`18`：
    
    ```sql
    search source=accounts
    | where age > 18
    | fields firstname, lastname
    ```
    {％include copy.html％}
    
3. 查看结果。
    - 查看结果**结果** 窗格，以表格格式显示查询输出。

   下图显示了查询编辑器窗格和结果窗格的PPL查询，该查询是在步骤2中运行的：

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/query-workbench-ppl.png" alt ="Query Workbench PPL query input and results output panes">

4. 清除查询编辑器。
    - 选择**清除** 按钮清除查询编辑器并运行新的查询。

5. 检查查询的处理方式。
    - 选择**解释** 按钮以检查OpenSearch如何处理查询，包括所涉及的步骤和操作顺序。

    下图显示了在步骤2中运行的PPL查询的说明。

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/query-PPL-explain.png" alt ="Query Workbench PPL query explanation pane" 宽度="500">

查询工作台不支持通过SQL或PPL更新操作。读取数据访问-仅有的。
{： 。重要的

