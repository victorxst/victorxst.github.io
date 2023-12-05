---
layout: default
title: 数据流
parent: 索引管理
nav_order: 20
redirect_from:
  - /dashboards/admin-ui-index/datastream/
  - /opensearch/data-streams/
---

# 数据流
引入2.6
{: .label .label-purple }

在OpenSearch仪表板中**索引管理** 应用程序允许您查看和管理[数据流]({{site.url}}{{site.baseurl}}/im-plugin/data-streams/) 如下图所示。

![数据流]({{site.url}}{{site.baseurl}}/images/admin-ui-index/datastreams1.png)

## 查看数据流

要查看数据流及其健康状况，请选择**数据流** 在下面**索引管理** 如下图所示。

![数据流]({{site.url}}{{site.baseurl}}/images/admin-ui-index/datastreams5.png)

以下是三个数据流健康状况：

- 绿色：分配了所有主要和复制碎片。
- 黄色：至少没有分配一个复制碎片。
- 红色：至少没有分配一个主碎片。

## 创建数据流

要创建数据流，请执行以下步骤：

1. 在下面**索引管理**， 选择**数据流**。

1. 选择**创建数据流**。

1. 输入数据流的名称**数据流名称**。

1. 确保您具有匹配索引模板。这将在**匹配索引模板**，如下图所示。

    ![数据流]({{site.url}}{{site.baseurl}}/images/admin-ui-index/datastreams3.png)

1. 这**来自模板的继承设置** 和**索引别名** 读取部分-仅，并显示数据流中包含的背索索引。

1. 如下图所示，主碎片的数量，副本数量和刷新间隔是从模板继承的。

    ![数据流]({{site.url}}{{site.baseurl}}/images/admin-ui-index/datastreams4.png)

1. 选择**创建数据流**。

## 删除数据流

要删除数据流，请执行以下步骤：

1. 在下面**索引管理**， 选择**数据流**。

1. 选择要删除的数据流。

1. 选择**动作**，然后选择**删除**。

## 滚动数据流

要在数据流上执行翻转操作，请执行以下步骤：

1. 在下面**索引管理**， 选择**数据流**。

1. 选择**动作**，然后选择**滚下**，如下图所示。

    ![滚下]({{site.url}}{{site.baseurl}}/images/admin-ui-index/rollover1.png)

1. 在下面**配置源**，选择要执行翻转操作的源数据流。

1. 选择**滚下**，如下图所示。

    ![滚下]({{site.url}}{{site.baseurl}}/images/admin-ui-index/rollover3.png)

## 强制合并数据流

要在两个或多个索引上执行武力合并操作，请执行以下步骤：

1. 在下面**索引管理**， 选择**数据流**。

1. 选择要执行力合并操作的数据流。

1. 选择**动作**，然后选择**力合并**。

1. 在下面**配置源索引**，指定要强制合并的数据流。

1. 可选，下**高级设置** 您可以选择**冲洗指数** 或者**仅删除删除** 然后指定**最大段数** 如下图所示，合并到。

    ![力合并]({{site.url}}{{site.baseurl}}/images/admin-ui-index/forcemerge2.png)

## 刷新数据流

刷新数据流可为搜索操作可见的索引提供新的更新。

刷新操作只能应用于打开与指定数据流关联的索引。

要刷新数据流，请从**数据流** 列表下**索引管理**。然后选择**刷新** 来自**动作** 下拉列表。

## 冲洗数据流

冲洗操作执行了Lucene提交，为磁盘编写细分市场并启动了新的翻译。

冲洗操作只能应用于打开与指定数据流相关的索引。

要冲洗数据流，请从**数据流** 列表下**索引管理**。然后选择**冲洗** 来自**动作** 下拉列表。

## 清除数据流缓存

这[清除缓存操作]({{site.url}}{{site.baseurl}}/api-reference/index-apis/clear-index-cache/) 只能应用于打开与指定数据流关联的索引。

要清除数据流缓存，请从**指数** 列表下**索引管理**。然后选择**清除缓存** 来自**动作** 下拉列表。

