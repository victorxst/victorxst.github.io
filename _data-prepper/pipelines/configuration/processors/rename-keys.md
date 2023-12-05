---
layout: default
title: rename_keys
parent: 处理器
grand_parent: 管道
nav_order: 85
---

# Rename_Keys

这`rename_keys` 处理器在事件中重命名键。

## 配置

您可以配置`rename_keys` 带有以下选项的处理器。

| 选项| 必需的| 描述|
| ：--- | ：--- | ：--- |
| `entries` | 是的| 重命名的活动条目列表。|
| `from_key` | 是的| 要重命名的条目的关键。|
| `to_key` | 是的| 条目的新钥匙。|
| `overwrite_if_to_key_exists` | 不| 设置为`true`，如果现有值被覆盖`key` 在这种情况下已经存在。默认值是`false`。|

## 用法

首先，创建以下内容`pipeline.yaml` 文件：

```yaml
pipeline:
  source:
    file:
      path: "/full/path/to/logs_json.log"
      record_type: "event"
      format: "json"
  processor:
    - rename_keys:
        entries:
        - from_key: "message"
          to_key: "newMessage"
          overwrite_if_to_key_exists: true
  sink:
    - stdout:
```
{％include copy.html％}


接下来，创建一个名为的日志文件`logs_json.log` 并更换`path` 在您的文件源中`pipeline.yaml` 用该文件备件提交。有关更多信息，请参阅[配置数据预备]({{site.url}}{{site.baseurl}}/data-prepper/getting-started/#2-configuring-data-prepper)。

例如，在运行之前`rename_keys` 处理器，如果`logs_json.log` 文件包含以下事件记录：

```json
{"message": "hello"}
```

当您运行`rename_keys` 处理器，将消息解析为以下"newMessage" 输出：

```json
{"newMessage": "hello"}
```

> 如果`newMessage` 已经存在，其现有价值被覆盖`value`。



## 特殊考虑

重命名操作按钥匙的顺序进行-价值对条目在`pipeline.yaml` 文件。这意味着链接（关键在哪里-价值对被重命名为顺序）隐含在`rename_keys` 处理器。请参阅以下示例`pipeline.yaml` 文件：

```yaml
pipeline:
  source:
    file:
      path: "/full/path/to/logs_json.log"
      record_type: "event"
      format: "json"
  processor:
    - rename_keys:
        entries:
        - from_key: "message"
          to_key: "message2"
        - from_key: "message2"
          to_key: "message3"
  sink:
    - stdout:
```

将以下内容添加到`logs_json.log` 文件：

```json
{"message": "hello"}
```
{％include copy.html％}

之后`rename_keys` 处理器运行，出现以下输出：

```json
{"message3": "hello"}
```
