---
layout: default
title: 滚动
parent: 索引管理
nav_order: 40
redirect_from:
  - /dashboards/admin-ui-index/rollover/
---

# 滚动
引入2.6
{: .label .label-purple }

OpenSearch仪表板允许您执行[索引滚动]({{site.url}}{{site.baseurl}}/im-plugin/ism/error-prevention/index/#rollover) 使用**索引管理**。

## 数据流

要在数据流上执行翻转操作，请执行以下步骤：

1. 在下面**索引管理**， 选择**数据流**。

1. 选择**动作**，然后选择**滚下**，如下图所示。

    ![滚下]({{site.url}}{{site.baseurl}}/images/admin-ui-index/rollover1.png)

1. 在下面**配置源**，选择要执行翻转操作的源数据流。

1. 选择**滚下**，如下图所示。

    ![滚下]({{site.url}}{{site.baseurl}}/images/admin-ui-index/rollover3.png)

## 别名

要在别名上执行转盘操作，请执行以下步骤：

1. 在下面**索引管理**， 选择**别名**。

1. 选择**动作**，然后选择**滚下**，如下图所示。

    ![滚下]({{site.url}}{{site.baseurl}}/images/admin-ui-index/rollover2.png)

1. 在下面**配置源**，选择要执行翻转操作的源别名。

1. 如果别名不包含写入索引，请提示您分配写索引，如下图所示。

    ![滚下]({{site.url}}{{site.baseurl}}/images/admin-ui-index/rollover4.png)

1. 在下面**配置新的翻车索引** 在**定义索引** 窗格，指定索引名称和可选索引别名。

1. 在下面**索引设置**，指定主碎片的数量，副本的数量和刷新间隔，如下图所示。

    ![滚下]({{site.url}}{{site.baseurl}}/images/admin-ui-index/rollover5.png)

1. 选择**滚下**。

