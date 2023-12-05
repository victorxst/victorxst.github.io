---
layout: default
title: copy_values 
parent: 处理器
grand_parent: 管道
nav_order: 48
---

# copy_values

这`copy_values` 处理器在事件中复制值，是[突变事件]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/processors/mutate-event/) 处理器。

## 配置

您可以配置`copy_values` 带有以下选项的处理器。

| 选项| 必需的| 描述|
:--- | :--- | :---
| `entries` | 是的| 活动中要复制的条目列表。|
| `from_key` | 是的| 要复制的条目的关键。|
| `to_key` | 是的| 要添加的新条目的关键。|
| `overwrite_if_key_exists` | 不| 设置为`true`，如果现有值被覆盖`key` 在这种情况下已经存在。默认值是`false`。|

## 用法

首先，创建以下内容`pipeline.yaml` 文件：

```yaml
pipeline:
  source:
    ...
  ....  
  processor:
    - copy_values:
        entries:
        - from_key: "message"
          to_key: "newMessage"
          overwrite_if_to_key_exists: true
  sink:
```
{% include copy.html %}

接下来，创建一个名为的日志文件`logs_json.log` 并更换`path` 在您的文件源中`pipeline.yaml` 用该文件备件提交。有关更多信息，请参阅[配置数据预备]({{site.url}}{{site.baseurl}}/data-prepper/getting-started/#2-configuring-data-prepper)。

例如，在运行之前`copy_values` 处理器，如果`logs_json.log` 文件包含以下事件记录：

```json
{"message": "hello"}
```

运行此处理器时，它将消息解析为以下输出：

```json
{"message": "hello", "newMessage": "hello"}
```

如果`newMessage` 已经存在，其现有价值被覆盖`value`。

