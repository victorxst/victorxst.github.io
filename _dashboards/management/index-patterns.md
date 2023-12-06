---
layout: default
title: 索引模式
parent: 仪表板管理
nav_order: 10
---

# 索引模式

索引模式对于访问OpenSearch数据至关重要。_index模式_引用一个或多个索引，数据流或索引别名。例如，索引模式可以将您指向昨天的日志数据或包含该数据的所有索引。

如果将数据存储在多个索引中，则创建一个索引模式使您的可视化能够从匹配索引模式的所有索引中检索数据。您需要创建索引模式来定义如何检索数据并格式化字段，以便您可以查询，搜索和显示数据。

## 开始

在本教程中，您将学习创建索引模式。

{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/icons/alert-icon.png" class ="inline-icon" alt ="alert icon" 大小="m"/> {:/}**笔记**<br>
要创建或修改索引模式，您必须具有创建，管理和删除权限。联系您的管理员以寻求支持。有关更多信息，请参阅[并发-租赁配置]({{site.url}}{{site.baseurl}}/security/multi-tenancy/multi-tenancy-config/#give-roles-access-to-tenants)。
{:.note}

## 先决条件

在创建索引模式之前，必须对数据进行索引。要了解在OpenSearch中索引您的数据，请参见[管理索引]({{site.url}}{{site.baseurl}}/im-plugin/index/)。

## 最佳实践

创建索引模式时考虑以下最佳实践：

- **使您的索引模式具体。** 与其创建与所有索引匹配的索引模式`my-index-`。您的索引模式越具体，查询和分析数据的越好。
- **很少使用通配符。** 通配符可用于匹配多个索引，但它们也可能使管理您的索引模式更加困难。尝试特别使用通配符。
- **测试您的索引模式。** 确保测试您的索引模式，以确保它们匹配正确的索引。

## 创建索引模式

如果添加了示例数据，则可以使用索引模式来分析该数据。要为您自己的数据创建索引模式，请按照以下步骤操作。

### 步骤1：定义索引模式

1. 转到OpenSearch仪表板，然后选择**管理** >**仪表板管理** >**索引模式**。
2. 选择**创建索引模式**。
3. 来自**创建索引模式** 窗口，通过输入您的索引模式的名称来定义索引模式**索引模式名称** 场地。仪表板自动添加通配符，`*`，一旦您开始键入。使用通配符有助于将索引模式与多个源或索引匹配。在您开始键入时显示匹配您的索引模式的所有索引的下拉列表。
4. 选择**下一步**。

步骤1的示例如下图所示。请注意索引模式`security*` 匹配三个索引。通过用通配符定义图案`*`，您可以查询和可视化索引中的所有数据。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/index-patterns-step1.png" alt="Index pattern step 1 UI " width="700"/>

### 步骤2：配置设置

1. 选择`@timestamp` 从下拉菜单中指定按时间过滤文档时要使用的时间字段。选择此时间过滤器确定应用到哪个字段。它可以是请求的时间戳或任何相关时间戳字段。如果您不想使用时间过滤器，请从下拉菜单中选择该选项。如果选择此选项，OpenSearch将返回与模式匹配的索引中的所有数据。

2. 选择**创建索引模式。** 下图显示了一个示例。

    <img src ="{{site.url}}{{site.baseurl}}/images/dashboards/index-pattern-step2.png" alt ="Index pattern step 2 UI " width="700"/>

创建索引模式后，您可以查看匹配索引的映射。在表中，您可以看到字段列表以及它们的数据类型和属性。下图显示了一个示例。

<img src="{{site.url}}{{site.baseurl}}//images/dashboards/index-pattern-table.png" alt="Index pattern table UI " width="700"/>

## 下一步

- [通过视觉效果了解您的数据]({{site.url}}{{site.baseurl}}/dashboards/visualize/viz-index/)。
- [挖掘您的数据]({{site.url}}{{site.baseurl}}/dashboards/discover/index-discover/)。

