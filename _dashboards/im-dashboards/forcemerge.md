---
layout: default
title: 强制合并
parent: 索引管理
nav_order: 30
redirect_from:
  - /dashboards/admin-ui-index/forcemerge/
---

# 强制合并
引入2.6
{: .label .label-purple }

OpenSearch仪表板允许您执行[强制合并]({{site.url}}{{site.baseurl}}/im-plugin/ism/error-prevention/index#force_merge) 在两个或多个索引上操作**索引管理**。

## 强制合并索引

要在两个或多个索引上执行强制合并操作，请执行以下步骤：

1. 在下面**索引管理**， 选择**指数**。

1. 选择要强制合并的索引。

1. 选择**动作**，然后选择**强制合并**，如下图所示。

    ![强制合并]({{site.url}}{{site.baseurl}}/images/admin-ui-index/forcemerge1.png)

1. 在下面**配置源索引**，指定要强制合并的索引。

1. 可选，下**高级设置** 您可以选择**冲洗指数** 或者**仅删除删除** 然后指定**最大段数** 如下图所示，合并到。

    ![强制合并]({{site.url}}{{site.baseurl}}/images/admin-ui-index/forcemerge2.png)

## 强制合并数据流

要在两个或多个索引上执行强制合并操作，请执行以下步骤：

1. 在下面**索引管理**， 选择**数据流**。

1. 选择要强制合并的数据流。

1. 选择**动作**，然后选择**强制合并**。

1. 在下面**配置源索引**，指定要强制合并的数据流。

1. 可选，下**高级设置** 您可以选择**冲洗指数** 或者**仅删除删除** 然后指定**最大段数** 如下图所示，合并到。

    ![强制合并]({{site.url}}{{site.baseurl}}/images/admin-ui-index/forcemerge2.png)

