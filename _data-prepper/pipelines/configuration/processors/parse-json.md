---
layout: default
title: parse_json 
parent: 处理器
grand_parent: 管道
nav_order: 80
---

# parse_json

这`parse_json` 处理器对事件（包括任何嵌套字段）的JSON数据解析。处理器提取JSON指针数据并将输入事件添加到提取的字段中。


## 配置

您可以配置`parse_json` 带有以下选项的处理器。

| 选项| 必需的| 类型| 描述|
| ：--- | ：--- | ：--- | ：--- | 
| `source` | 不| 细绳| 该领域`event` 这将被解析。默认值是`message`。|
| `destination` | 不| 细绳| 解析的JSON的目的地字段。默认为`event`。不可能是`""`，，，，`/`，或任何空格-仅有的`string` 因为这些无效`event` 字段。|
| `pointer` | 不| 细绳| JSON指向该领域的指针进行解析。没有`pointer` 默认情况下，意味着整个`source` 解析。这`pointer` 也可以访问JSON阵列索引。如果JSON指针无效，那么整个`source` 数据被解析为外向`event`。如果指向的密钥已经存在于`event` 和`destination` 是根，然后指针使用钥匙的整个路径。|

## 用法

首先，创建以下内容`pipeline.yaml` 文件：

```yaml
parse-json-pipeline:
  source:
    ...
  ....  
  processor:
    - parse_json:
```

### 基本示例

测试`parse_json` 带有以前配置的处理器，运行管道并将以下行粘贴到您的控制台中，然后输入`exit` 在新线上：

```
{"outer_key": {"inner_key": "inner_value"}}
```
{％include copy.html％}

这`parse_json` 处理器将消息解析为以下格式：

```
{"message": {"outer_key": {"inner_key": "inner_value"}}", "outer_key":{"inner_key":"inner_value"}}}
```

### 用JSON指针的示例

您可以使用JSON指针来解析JSON数据的选择`pointer` 配置中的选项。首先，创建以下内容`pipeline.yaml` 文件：

```yaml
parse-json-pipeline:
  source:
    ...
  ....  
  processor:
    - parse_json:
        pointer: "outer_key/inner_key"
```

测试`parse_json` 带有指针选项的处理器，运行管道，将以下行粘贴到您的控制台中，然后输入`exit` 在新线上：

```
{"outer_key": {"inner_key": "inner_value"}}
```
{％include copy.html％}

处理器将消息解析为以下格式：

```
{"message": {"outer_key": {"inner_key": "inner_value"}}", "inner_key": "inner_value"}
```
