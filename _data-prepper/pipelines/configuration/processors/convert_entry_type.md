---
layout: default
title: convert_entry_type
parent: 处理器
grand_parent: 管道
nav_order: 47
---

# convert_entry_type

这`convert_entry_type` 处理器将与事件中指定键关联的值类型转换为指定类型。这是一个铸造处理器，它会更改事件中某些字段的类型。某些数据必须转换为其他类型，例如整数转换为double，或将字符串转换为整数，以便它可以通过条件传递事件-基于处理器或执行条件路由。

## 配置

您可以配置`convert_entry_type` 带有以下选项的处理器。

| 选项| 必需的| 描述|
| :--- | :--- | :--- |
| `key`| 是的| 值的键需要转换为其他类型。|
| `type` | 不| 键的目标类型-价值对。可能的值是`integer`，`double`，`string`， 和`Boolean`。默认值是`integer`。|

## 用法

首先，创建以下内容`pipeline.yaml` 文件：

```yaml
type-conv-pipeline:
  source:
    ...
  ....  
  processor:
    - convert_entry_type:
        key: "response_status"
        type: "integer"
```
{% include copy.html %}

接下来，创建一个名为的日志文件`logs_json.log` 并更换`path` 在您的文件源中`pipeline.yaml` 用该文件备件提交。有关更多信息，请参阅[配置数据预备]({{site.url}}{{site.baseurl}}/data-prepper/getting-started/#2-configuring-data-prepper)。

例如，在运行之前`convert_entry_type` 处理器，如果`logs_json.log` 文件包含以下事件记录：


```json
{"message": "value", "response_status":"200"}
```

这`convert_entry_type` 处理器将接收到的输出转换为以下输出，其中类型`response_status` 价值从字符串变为整数：

```json
{"message":"value","response_status":200}
```

