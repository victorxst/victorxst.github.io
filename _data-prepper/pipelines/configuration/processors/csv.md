---
layout: default
title: csv 
parent: 处理器
grand_parent: 管道
nav_order: 49
---

# CSV

这`csv` 处理器解析逗号-将事件分开的值（CSV）分为列。

## 配置

下表描述了您可以使用的选项来配置`csv` 处理器。

选项| 必需的| 类型| 描述
:--- | :--- | :--- | :---
来源| 不| 细绳| 将被解析的领域。默认值是`message`。
quote_character| 不| 细绳| 该字符用作单个数据列的文本预选赛。默认值是`"`。
定界符| 不| 细绳| 分隔每一列的角色。默认值是`,`。
delete_header| 不| 布尔| 如果指定了事件标题（`column_names_source_key`解析事件后，将删除）。如果没有事件标题，则不会采取任何措施。默认值是正确的。
column_names_source_key| 不| 细绳| 如果指定CSV列名称的字段，将自动检测到。如果需要额外的列名，则列名是根据其索引自动生成的。如果`column_names` 也定义了标题`column_names_source_key` 也可以用于生成事件字段。如果在此字段中指定了太少的列，则将自动生成剩余的列名。如果在此字段中指定了太多的列名，则CSV处理器省略了额外的列名。
column_names| 不| 列表| 用户-CSV列的指定名称。默认值是`[column1, column2, ..., columnN]` 如果CSV记录中没有数据列，并且`column_names_source_key` 没有定义。如果`column_names_source_key` 已定义，标题为`column_names_source_key` 生成事件字段。如果在此字段中指定了太少的列，则将自动生成剩余的列名。如果在此字段中指定了太多的列名，则CSV处理器省略了额外的列名。

<！---## 配置

内容将添加到本节中。--->

## 指标

下表描述了常见[抽象处理器](https://github.com/opensearch-project/data-prepper/blob/main/data-prepper-api/src/main/java/org/opensearch/dataprepper/model/processor/AbstractProcessor.java) 指标。

| 公制名称| 类型| 描述|
| ------------- | ---- | -----------|
| `recordsIn` | 柜台| 表示记录进入管道组件的公制。|
| `recordsOut` | 柜台| 表示管道组件中记录出口的公制。|
| `timeElapsed` | 计时器| 公制表示管道组件执行期间经过的时间。|

这`csv` 处理器包括以下自定义指标。

**柜台**

*`csvInvalidEvents`：无效事件的数量。解析无效的事件时，会抛出例外。未封闭的报价通常会导致此例外。

