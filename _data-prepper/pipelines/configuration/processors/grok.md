---
layout: default
title: grok
parent: 处理器
grand_parent: 管道
nav_order: 54
---

# 格罗克


这`Grok` 处理器采用非结构化数据，并利用模式匹配来结构并提取重要的密钥。

## 配置

下表描述了您可以与`Grok` 处理器构建数据并使您的数据更容易查询。

选项| 必需的| 类型| 描述
：--- | ：--- | ：--- | ：---
匹配| 不| 地图| 指定哪些键与特定模式相匹配。默认值是一个空体。
keep_empty_captures| 不| 布尔| 启用保存`null` 捕获。默认值是`false`。
名称_captures_only| 不| 布尔| 指定是否仅保留命名捕获。默认值是`true`。
break_on_match| 不| 布尔| 指定是否匹配所有模式或在找到第一个成功的比赛后停止。默认值是`true`。
keys_to_overwrite| 不| 列表| 指定如果存在具有相同键值的捕获，则现有密钥将被覆盖。默认值是`[]`。
tatter_definitions| 不| 地图| 允许内联自定义模式使用。默认值是一个空体。
tatters_directories| 不| 列表| 指定包含客户模式文件的目录路径。默认值是一个空列表。
tatter_files_glob| 不| 细绳| 指定从指定的目录中使用的模式文件`pattern_directories`。默认值是`*`。
target_key| 不| 细绳| 指定父母-级键用于存储所有捕获。默认值是`null`。
Timeout_millis| 不| 整数| 匹配发生的最长时间。设置为`0` 禁用超时。默认值是`30,000`。

<！---## 配置

内容将添加到本节中。--->

## 指标

下表描述了常见[抽象处理器](https://github.com/opensearch-project/data-prepper/blob/main/data-prepper-api/src/main/java/org/opensearch/dataprepper/model/processor/AbstractProcessor.java) 指标。

| 公制名称| 类型| 描述|
| ------------- | ---- | -----------|
| `recordsIn` | 柜台| 表示记录进入管道组件的公制。|
| `recordsOut` | 柜台| 表示管道组件中记录出口的公制。|
| `timeElapsed` | 计时器| 公制表示管道组件执行期间经过的时间。|

这`Grok` 处理器包括以下自定义指标。

### 柜台

*`grokProcessingMismatch`：记录与比赛字段中指定的任何模式不匹配的记录数。
*`grokProcessingMatch`：记录至少一种模式的记录数量`match` 场地。
*`grokProcessingErrors`：记录记录处理错误的总数。
*`grokProcessingTimeouts`：记录匹配时计时的记录总数。

### 计时器

*`grokProcessingTime`：各个记录所花费的时间与模式相匹配的时间`match`。这`avg` 度量标准是该计时器最有用的指标，因为它为您提供了所需的记录匹配时间的平均值。

