---
layout: default
title: service_map 
parent: 处理器
grand_parent: 管道
nav_order: 95
---

# service_map

这`service_map` 处理器使用OpentElemetry数据来创建一个分布式服务映射，以在OpenSearch仪表板中可视化。

## 配置

下表描述了您可以使用的选项来配置`service_map` 处理器。

选项| 必需的| 类型| 描述
:--- | :--- | :--- | :---
window_duration| 不| 整数| 代表固定的时间窗口，以秒为单位，在其中评估了服务地图关系。默认值为180。

<！---## 配置

内容将添加到本节中。--->

## 指标

下表描述了常见[抽象处理器](https://github.com/opensearch-project/data-prepper/blob/main/data-prepper-api/src/main/java/org/opensearch/dataprepper/model/processor/AbstractProcessor.java) 指标。

| 公制名称| 类型| 描述|
| ------------- | ---- | -----------|
| `recordsIn` | 柜台| 表示记录进入管道组件的公制。|
| `recordsOut` | 柜台| 表示管道组件中记录出口的公制。|
| `timeElapsed` | 计时器| 公制表示管道组件执行期间经过的时间。|

这`service-map-stateful` 处理器包括以下自定义指标：

*`traceGroupCacheCount`：跟踪组缓存中的跟踪组数。
*`spanSetCount`：跨度集集合中的跨度集数量

