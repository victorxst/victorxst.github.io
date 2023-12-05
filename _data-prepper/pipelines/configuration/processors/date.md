---
layout: default
title: 日期
parent: 处理器
grand_parent: 管道
nav_order: 50
---

# 日期


这`date` 处理器将默认时间戳添加到事件，解析时间戳字段，然后将时间戳信息转换为国际标准化组织（ISO）8601格式。此时间戳信息可用作事件时间戳。

## 配置

下表描述了您可以使用的选项来配置`date` 处理器。

选项| 必需的| 类型| 描述
:--- | :--- | :--- | :---
匹配| 有条件的| 列表| 列表`key` 和`patterns` 其中模式为列表。比赛列表可以完全有一个`key` 和`patterns`。没有默认值。此选项不能与`from_time_received`。如果使用两个选项，则将多个日期处理器包括在管道中。
From_Time_received| 有条件的| 布尔| 一个布尔值，用于将默认时间戳添加到事件元数据的事件数据中，这是源接收事件的时间。默认值是`false`。此选项不能与`match`。如果使用两个选项，则将多个日期处理器包括在管道中。
目的地| 不| 细绳| 字段存储按日期处理器解析的时间戳。它可以与两者一起使用`match` 和`from_time_received`。默认值是`@timestamp`。
source_timezone| 不| 细绳| 时区用于解析日期。如果无法从值中提取区域或偏移，则使用它。如果区域或偏移是该值的一部分，则忽略时区。查找所有可用时区[数据库时区列表](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List) 在里面**TZ数据库名称** 柱子。
destination_timezone| 不| 细绳| 时区用于存储时间戳`destination` 场地。可用时区值与`source_timestamp`。
语言环境| 不| 细绳| 语言环境用于解析日期。它通常用于解析月份（`MMM`）。它可以使用IETF BCP 47具有语言，国家和变体字段或字符串表示形式[语言环境](https://docs.oracle.com/javase/8/docs/api/java/util/Locale.html) 目的。例如`en-US` 对于IETF BCP 47和`en_US` 用于字符串表示环境。可以找到包括语言，国家和变体在内的语言环境字段的完整列表[语言子标签注册表](https://www.iana.org/assignments/language-subtag-registry/language-subtag-registry)。默认值是`Locale.ROOT`。

<！---## 配置

内容将添加到本节中。--->

## 指标

下表描述了常见[抽象处理器](https://github.com/opensearch-project/data-prepper/blob/main/data-prepper-api/src/main/java/org/opensearch/dataprepper/model/processor/AbstractProcessor.java) 指标。

| 公制名称| 类型| 描述|
| ------------- | ---- | -----------|
| `recordsIn` | 柜台| 表示记录进入管道组件的公制。|
| `recordsOut` | 柜台| 表示管道组件中记录出口的公制。|
| `timeElapsed` | 计时器| 公制表示管道组件执行期间经过的时间。|

这`date` 处理器包括以下自定义指标。

*`dateProcessingMatchSuccessCounter`：返回与至少一种模式匹配的记录数量`match configuration` 选项。
*`dateProcessingMatchFailureCounter`：返回与不匹配任何模式的记录数量`patterns match` 配置选项

