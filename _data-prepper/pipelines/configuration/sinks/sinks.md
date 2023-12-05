---
layout: default
title: 下沉
parent: 管道
has_children: true
nav_order: 30
---

# 下沉

sinks定义数据预先将数据写入的位置。

## 所有水槽类型的一般选项

下表描述了您可以使用的选项来配置`sinks` 下沉。

选项| 必需的| 类型| 描述
：--- | :--- |:------------| ：---
路线| 不| 字符串列表| 该水槽适用的路线列表。如果没有提供，此水槽将接收所有事件。看[条件路由]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/pipelines#conditional-routing) 了解更多信息。
tags_target_key| 不| 细绳| 指定时，在提供的密钥的输出中包括事件标签。
include_keys| 不| 字符串列表| 指定时，在发送到接收器的数据中提供此列表中的键。某些编解码器和水槽不允许使用此字段。
dubl_keys| 不| 字符串列表| 指定时，排除从发送到接收器的数据中给出的密钥。某些编解码器和水槽不允许使用此字段。

