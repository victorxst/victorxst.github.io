---
layout: default
title: drop_events
parent: 处理器
grand_parent: 管道
nav_order: 53
---

# drop_events


这`drop_events` 处理器将传递到其中的所有事件丢弃。下表描述了何时删除事件以及如何处理掉落事件的例外。

选项| 必需的| 类型| 描述
：--- | ：--- | ：--- | ：---
drop_当| 是的| 细绳| 接受数据prepper表达式字符串之后[数据预先表达语法]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/expression-syntax/)。配置`drop_events` 和`drop_when: true` 丢弃收到的所有事件。
hander_failed_events| 不| 枚举| 指定在评估事件时发生异常时如何处理异常。默认值是`drop`，将事件丢弃，以免将其发送到OpenSearch。可用选项是`drop`，，，，`drop_silently`，，，，`skip`， 和`skip_silently`。有关更多信息，请参阅[hander_failed_events](https://github.com/opensearch-project/data-prepper/tree/main/data-prepper-plugins/drop-events-processor#handle_failed_events)。

<！---## 配置

内容将添加到本节中。--->

