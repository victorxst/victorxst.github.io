---
layout: default
title: delete_entries
parent: 处理器
grand_parent: 管道
nav_order: 51
---

# delete_entries

这`delete_entries` 处理器删除条目，例如密钥-价值对，来自事件。您可以定义要在`with-keys` 字段以下`delete_entries` 在YAML配置文件中。这些键及其值已删除。

## 配置

您可以配置`delete_entries` 带有以下选项的处理器。

| 选项| 必需的| 描述|
：--- | ：--- | ：---
| `with_keys` | 是的| 要删除条目的一系列键。|

## 用法

首先，创建以下内容`pipeline.yaml` 文件：

```yaml
pipeline:
  source:
    ...
  ....  
  processor:
    - delete_entries:
        with_keys: ["message"]
  sink:
```
{％include copy.html％}

接下来，创建一个名为的日志文件`logs_json.log` 并更换`path` 在您的文件源中`pipeline.yaml` 用该文件备件提交。有关更多信息，请参阅[配置数据预备]({{site.url}}{{site.baseurl}}/data-prepper/getting-started/#2-configuring-data-prepper)。

例如，在运行之前`delete_entries` 处理器，如果`logs_json.log` 文件包含以下事件记录：

```json
{"message": "hello", "message2": "goodbye"}
```

当您运行`delete_entries` 处理器，将消息解析为以下输出：

```json
{"message2": "goodbye"}
```

> 如果`message` 在此事件中不存在，因此不会发生任何行动。

