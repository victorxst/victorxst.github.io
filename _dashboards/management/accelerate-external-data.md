---
layout: default
title: 使用 OpenSearch 索引优化查询性能
parent: 将 Amazon S3 连接到 OpenSearch
grand_parent: 数据源
nav_order: 15
has_children: false
---

# 使用 OpenSearch 索引优化查询性能
引入2.11
{: .label .label-purple }


出于网络延迟，数据转换和数据量等原因，查询性能在使用外部数据源时可能会很慢。您可以使用OpenSearch索引（例如跳过索引或覆盖索引）来优化查询性能。_skipping索引_使用跳过加速方法，例如分区，最小值和最大值和值集，以摄取并创建紧凑的聚合数据结构。这使他们成为直接查询场景的经济选择。_ covering Index_将所有或某些数据从源中摄取到OpenSearch中，并可以使用所有OpenSearch仪表板和插件功能。看到[火石索引参考手册](https://github.com/opensearch-project/opensearch-spark/blob/main/docs/index.md) 有关此功能的索引过程的全面指导。

## 数据源用例：加速性能

开始**加速性能** 可用的用例**数据源**， 按着这些次序：

1. 去**OpenSearch仪表板** >**查询工作台** 并从**数据源** 上下拉菜单-左角。
2. 从左边-侧导航菜单，选择一个数据库。使用`http_logs` 数据库如下图所示。

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/query-workbench-accelerate-data.png" alt ="Query Workbench accelerate data UI" 宽度="700"/>

3. 查看表中的结果并确认您具有所需的数据。
4. 通过遵循以下步骤来创建OpenSearch索引：
    1.选择**加速数据** 按钮。流行音乐-出现向上窗口。下图显示了一个示例。

   <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/accelerate-data-popup.png" alt ="Accelerate data pop-up window" 宽度="700"/>

    2.输入您的详细信息**选择数据字段**。在里面**数据库** 字段，选择所需的加速索引：**跳过索引** 或者**覆盖索引**。_skipping Index_使用跳过加速方法，例如分区，最小/最大值和值集，以使用紧凑的聚合数据结构来摄取数据。这使他们成为直接查询场景的经济选择。_ covering Index_将所有或某些数据从源中摄取到OpenSearch中，并可以使用所有OpenSearch仪表板和插件功能。
    
5. 在下面**索引设置**，输入加速索引的信息。有关命名的信息，请选择**帮助**。请注意，亚马逊S3表一次只能有一个跳过索引。下图显示了一个示例。

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/skipping-index-settings.png" alt ="Skipping index settings" 宽度="700"/>

### 定义跳过索引设置

1. 在下面**跳过索引定义**，选择**添加字段** 按钮以定义跳过索引加速度方法，然后选择要添加的字段。下图显示了一个示例。

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/add-fields-skipping-index.png" alt ="Skipping index add fields" 宽度="700"/>

2. 选择**将查询复制到编辑器** 按钮以应用您的跳过索引设置。
3. 查看表窗格中的跳过索引查询详细信息，然后选择**跑步** 按钮。您的索引在左边添加-侧导航菜单包含数据库列表。下图显示了一个示例。

    <img src ="{{site.url}}{{site.baseurl}}/query-workbench-S3.png" alt ="Run a skippping or covering index UI" 宽度="700"/>

### 定义覆盖索引设置

1. 在下面**索引设置**，输入有效的索引名称。请注意，每个Amazon S3表都可以具有多个覆盖索引。下图显示了一个示例。

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/covering-index-naming.png" alt ="Covering index settings" 宽度="700"/>

2. 添加索引名称后，通过选择来定义覆盖索引字段`(add fields here)` 在下面**涵盖索引定义**。下图显示了一个示例。

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/covering-index-fields.png" alt ="Covering index field naming" 宽度="700"/>

3. 选择**将查询复制到编辑器** 按钮应用您的覆盖索引设置。
4. 查看表窗格中的覆盖索引查询详细信息，然后选择**跑步** 按钮。您的索引在左边添加-侧导航菜单包含数据库列表。下图显示了一个示例UI。
 
     <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/run-index-query-workbench.png" alt ="Run index in Query Workbench" 宽度="700"/>

## 限制

此功能仍在开发中，因此存在一些局限性。真正的-时间更新，请参阅[GitHub上的开发人员文档](https://github.com/opensearch-project/opensearch-spark/blob/main/docs/index.md#limitations)。

