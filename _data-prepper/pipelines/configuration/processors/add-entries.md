---
layout: default
title: add_entries
parent: 处理器
grand_parent: 管道
nav_order: 40
---

# add_entries

这`add_entries` 处理器将条目添加到事件中。

### 配置

您可以配置`add_entries` 带有以下选项的处理器。

| 选项| 必需的| 描述|
| ：--- | ：--- | ：--- |
| `entries` | 是的| 添加到事件的条目列表。|
| `key` | 是的| 要添加的新条目的关键。钥匙的一些示例包括`my_key`，，，，`myKey`， 和`object/sub_Key`。|
| `value` | 是的| 要添加的新条目的价值。您可以使用以下数据类型：字符串，布尔值，数字，空，嵌套对象和数组。|
| `overwrite_if_key_exists` | 不| 设置为`true`，如果现有值被覆盖`key` 在这种情况下已经存在。默认值是`false`。|

### 用法

首先，创建以下内容`pipeline.yaml` 文件：

```yaml
pipeline:
  source:
    ...
  ....  
  processor:
    - add_entries:
        entries:
        - key: "newMessage"
          value: 3
          overwrite_if_key_exists: true
  sink:
```
{％include copy.html％}


例如，当您的源包含以下事件记录时：

```json
{"message": "hello"}
```

然后您运行`add_entries` 使用示例管道的处理器，它添加了一个新条目，`{"newMessage": 3}`，到现有事件，`{"message": "hello"}`，使得新事件在最终输出中包含两个条目：

```json
{"message": "hello", "newMessage": 3}
```

> 如果`newMessage` 已经存在，其现有价值被覆盖`3`。


