---
layout: default
title: 创建Perftop仪表板
parent: 性能分析仪
nav_order: 2
redirect_from:
  - /monitoring-plugins/pa/dashboards/
---

# Perftop仪表板

仪表板在JSON中定义，由三个主要元素组成：表，线图和条形图。您可以定义一个行和列的网格，然后将元素放置在该网格中，每个元素均按照指定的数量和列的数量。

开始构建自定义仪表板的最佳方法是复制和修改现有的JSON文件之一`dashboards` 目录。
{: .tip }

---

#### Table of contents
1. TOC
{:toc}

---


## 元素摘要

- 表显示每个维度的指标。例如，如果您的指标是`CPU_Utilization` 和你的尺寸`ShardID`，一个perftop表显示了每个节点上每个碎片的一行。
- 条形图是为集群汇总的，除非您添加`nodeName` 到仪表板。看到[所有元素的选项](#all-elements)。
- 每个节点汇总了线图。每行代表一个节点。


## 位置元素

perftop位置元素在网格中。例如，考虑此12 * 12网格。

![仪表板网格]({{site.url}}{{site.baseurl}}/images/perftop-grid.png)

上层-网格的左表示第0行，第0列，因此三个框的起始位置是：

- 橙色：第0行，第0列
- 紫色：第2行，第2列
- 绿色：第1行，第6列

这些框横跨许多行和列。在这种情况下：

- 橙色：2行，4列
- 紫色：1行，4列
- 绿色：3行，2列

以JSON形式，我们有以下内容：

```json
{
  "gridOptions": {
    "rows": 12,
    "cols": 12
  },
  "graphs": {
    "tables": [{
        "options": {
          "gridPosition": {
            "row": 0,
            "col": 0,
            "rowSpan": 2,
            "colSpan": 4
          }
        }
      },
      {
        "options": {
          "gridPosition": {
            "row": 2,
            "col": 2,
            "rowSpan": 1,
            "colSpan": 4
          }
        }
      },
      {
        "options": {
          "gridPosition": {
            "row": 1,
            "col": 6,
            "rowSpan": 3,
            "colSpan": 2
          }
        }
      }
    ]
  }
}
```

但是，在这一点上，JSON所做的就是定义三个表的大小和位置。要用数据填充元素，请指定查询。


## 添加查询

查询使用与[REST API]({{site.url}}{{site.baseurl}}/monitoring-plugins/pa/api/)，仅以JSON形式：

```json
{
  "queryParams": {
    "metrics": "estimated,limitConfigured",
    "aggregates": "avg,avg",
    "dimensions": "type",
    "sortBy": "estimated"
  }
}
```

有关可用指标的详细信息，请参阅[指标参考]({{site.url}}{{site.baseurl}}/monitoring-plugins/pa/reference/)。


## 添加选项

选项包括标签，颜色和刷新间隔。不同的元素类型具有不同的选项。

仪表板支持16种ANSI颜色：黑色，红色，绿色，黄色，蓝色，洋红色，青色和白色。为了"bright" 这些颜色的变体，使用数字8--15.如果您的终端支持256种颜色，则也可以使用十六进制代码（例如，`#6D40ED`）。
{:.note}


### 所有元素

选项| 类型| 描述
:--- | :--- | :---
`label` | 字符串或整数| 鞋面中的文字-盒子的左角。
`labelColor` | 字符串或整数| 标签的颜色。
`refreshInterval` | 整数| 拨打绩效分析仪API的毫秒数以获取新数据。最低值为5000。
`dimensionFilters` | 字符串数组| 图表显示的尺寸值。例如，如果您查询`metric=Net_Throughput&agg=sum&dim=Direction` 并且可能的维度值为`in` 和`out`，您可以定义`dimensionFilters: ["in"]` 仅显示用于的度量数据`in` 方面
`nodeName` | 细绳| 如果不-无效，让您将元素限制为单个节点。您可以直接在仪表板文件中指定节点名称，但更好的方法是使用`"nodeName": "#nodeName"` 在仪表板中，包括`--nodename <node_name>` 启动perftop时的论点。


### 表

选项| 类型| 描述
:--- | :--- | :---
`bg` | 字符串或整数| 背景颜色。
`fg` | 字符串或整数| 文字颜色。
`selectedFg` | 字符串或整数| 专注文本的文本颜色。
`selectedBg` | 字符串或整数| 专注文本的背景颜色。
`columnSpacing` | 整数| 列之间的空间数量（以字符为单位）。
`keys` | 布尔| 目前没有影响。


### 酒吧

选项| 类型| 描述
:--- | :--- | :---
`barWidth` | 整数| 图中每个条的宽度（以字符测量）。
`xOffset` | 整数| Y之间的空间数量（以字符为单位）-轴和图中的第一个条。
`maxHeight` | 整数| 图中每个条的最大高度（以字符为单位测量）。


### 线

选项| 类型| 描述
:--- | :--- | :---
`showNthLabel` | 整数| 哪一个`xAxis` 显示标签。例如，`"showNthLabel": 2` 显示所有其他标签。
`showLegend` | 布尔| 是否显示线图的传奇。
`legend.width` | 整数| 图表中的传说宽度（以字符为单位）。
`xAxis` | 字符串数组| X的标签数组-轴。例如，`["0:00", "0:10", "0:20", "0:30", "0:40", "0:50"]`。
`colors` | 字符串数组| 一系列线色可供选择。例如，`["magenta", "cyan"]`。如果您不提供此值，则Perftop为每行选择随机颜色。

