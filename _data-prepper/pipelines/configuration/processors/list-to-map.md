---
layout: default
title: list_to_map 
parent: 处理器
grand_parent: 管道
nav_order: 58
---

# list_to_map

这`list_to_map` 处理器转换一个从事件的对象列表，每个对象都包含一个`key` 字段，进入目标键的地图。

## 配置

下表描述了用于生成映射的目标键的配置选项。

选项| 必需的| 类型| 描述
:--- | :--- | :--- | :---
`key` | 是的| 细绳| 在生成的映射中将要提取为键的字段的钥匙。
`source` | 是的| 细绳| 具有的对象列表`key` 要转换为生成地图的键的字段。
`target` | 不| 细绳| 生成地图的目标。如果未指定，生成的地图将放在根节点中。
`value_key` | 不| 细绳| 指定时，值给定一个`value_key` 在源列表中包含的对象中，将根据生成的映射提取并将其转换为该选项指定的值。如果未指定，源列表中包含的对象将在映射时保留其原始值。
`flatten` | 不| 布尔| 什么时候`true`，生成的地图输出中的值将基于`flattened_element`。否则，从生成地图中映射到值的对象显示为列表。
`flattened_element` | 有条件的| 细绳| 要保留的元素`first` 或者`last`， 什么时候`flatten` 被设定为`true`。

## 用法

以下示例显示了如何测试使用`list_to_map` 在您自己的来源使用处理器之前，处理器。

创建一个名为的源文件`logs_json.log`。因为`file` 来源在`.log` 文件作为事件，即使包含多个对象，对象列表也以一行的形式出现：

```json
{"mylist":[{"name":"a","value":"val-a"},{"name":"b","value":"val-b1"},{"name":"b",  "value":"val-b2"},{"name":"c","value":"val-c"}]}
```
{% include copy.html %}

接下来，创建一个`pipeline.yaml` 使用的文件`logs_json.log` 文件作为`source` 通过指向`.log` 文件的正确路径：

```yaml
pipeline:
  source:
    file:
      path: "/full/path/to/logs_json.log"
      record_type: "event"
      format: "json"
  processor:
    - list_to_map:
        key: "name"
        source: "mylist"
        value_key: "value"
        flatten: true
  sink:
    - stdout:
```
{% include copy.html %}

运行管道。如果成功，处理器将返回生成的映射，并根据其对象根据其映射的对象`value_key`。类似于原始源，其中包含一行，因此，处理器将以下JSON作为一行返回。为了可读性，已调整了以下示例和所有随后的JSON示例以跨越多行：

```json
{
  "mylist": [
    {
      "name": "a",
      "value": "val-a"
    },
    {
      "name": "b",
      "value": "val-b1"
    },
    {
      "name": "b",
      "value": "val-b2"
    },
    {
      "name": "c",
      "value": "val-c"
    }
  ],
  "a": "val-a",
  "b": "val-b1",
  "c": "val-c"
}
```

### 示例：设置为`target`

以下示例`pipeline.yaml` 文件显示`list_to_map` 处理器设置为指定目标时，`mymap`：

```yaml
pipeline:
  source:
    file:
      path: "/full/path/to/logs_json.log"
      record_type: "event"
      format: "json"
  processor:
    - list_to_map:
        key: "name"
        source: "mylist"
        target: "mymap"
        value_key: "value"
        flatten: true
  sink:
    - stdout:
```
{% include copy.html %}

生成的地图出现在目标密钥下：

```json
{
  "mylist": [
    {
      "name": "a",
      "value": "val-a"
    },
    {
      "name": "b",
      "value": "val-b1"
    },
    {
      "name": "b",
      "value": "val-b2"
    },
    {
      "name": "c",
      "value": "val-c"
    }
  ],
  "mymap": {
    "a": "val-a",
    "b": "val-b1",
    "c": "val-c"
  }
}
```

### 示例：否`value_key` 指定的

以下示例`pipeline.yaml` 文件显示`list_to_map` 没有否`value_key` 指定的。因为`key` 被设定为`name`，处理器提取对象名称在地图中用作密钥。

```yaml
pipeline:
  source:
    file:
      path: "/full/path/to/logs_json.log"
      record_type: "event"
      format: "json"
  processor:
    - list_to_map:
        key: "name"
        source: "mylist"
        flatten: true
  sink:
    - stdout:
```
{% include copy.html %}

来自生成地图的值出现为原始对象`.log` 来源，如以下示例响应所示：

```json
{
  "mylist": [
    {
      "name": "a",
      "value": "val-a"
    },
    {
      "name": "b",
      "value": "val-b1"
    },
    {
      "name": "b",
      "value": "val-b2"
    },
    {
      "name": "c",
      "value": "val-c"
    }
  ],
  "a": {
    "name": "a",
    "value": "val-a"
  },
  "b": {
    "name": "b",
    "value": "val-b1"
  },
  "c": {
    "name": "c",
    "value": "val-c"
  }
}
```

### 例子：`flattened_element` 设置`last`

以下示例`pipeline.yaml` 文件设置`flattened_element` 最后，因此，基于每个值的最后一个元素对处理器输出进行弄平：

```yaml
pipeline:
  source:
    file:
      path: "/full/path/to/logs_json.log"
      record_type: "event"
      format: "json"
  processor:
    - list_to_map:
        key: "name"
        source: "mylist"
        target: "mymap"
        value_key: "value"
        flatten: true
        flattened_element: "last"
  sink:
    - stdout:
```
{% include copy.html %}

处理器地图对象`b` 值`val-b2` 因为`val-b2` 是对象中的最后一个元素`b`，如以下输出所示：

```json
{
  "mylist": [
    {
      "name": "a",
      "value": "val-a"
    },
    {
      "name": "b",
      "value": "val-b1"
    },
    {
      "name": "b",
      "value": "val-b2"
    },
    {
      "name": "c",
      "value": "val-c"
    }
  ],
  "a": "val-a",
  "b": "val-b2",
  "c": "val-c"
}
```


### 例子：`flatten` 设置为false

以下示例`pipeline.yaml` 文件集`flatten` 到`false`，导致处理器从生成的地图作为列表中输出值：

```yaml
pipeline:
  source:
    file:
      path: "/full/path/to/logs_json.log"
      record_type: "event"
      format: "json"
  processor:
    - list_to_map:
        key: "name"
        source: "mylist"
        target: "mymap"
        value_key: "value"
        flatten: false
  sink:
    - stdout:
```
{% include copy.html %}

响应中的某些对象的值可能具有多个元素，如以下响应所示：

```json
{
  "mylist": [
    {
      "name": "a",
      "value": "val-a"
    },
    {
      "name": "b",
      "value": "val-b1"
    },
    {
      "name": "b",
      "value": "val-b2"
    },
    {
      "name": "c",
      "value": "val-c"
    }
  ],
  "a": [
    "val-a"
  ],
  "b": [
    "val-b1",
    "val-b2"
  ],
  "c": [
    "val-c"
  ]
}
```
