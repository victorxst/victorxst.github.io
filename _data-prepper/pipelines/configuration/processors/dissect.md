---
layout: default
title: 解剖
parent: 处理器
grand_parent: 管道
nav_order: 52
---

# 解剖

这`dissect` 处理器从事件中提取值，并根据用户将其映射到单个字段-定义`dissect` 模式。该处理器非常适合从具有已知结构的日志消息中提取现场提取。

## 基本用法

使用`dissect` 处理器，创建以下`pipeline.yaml` 文件：

```yaml
dissect-pipeline:
  source:
    file:
      path: "/full/path/to/logs_json.log"
      record_type: "event"
      format: "json"
  processor:
    - dissect:
        map:
          log: "%{Date} %{Time} %{Log_Type}: %{Message}"
  sink:
    - stdout:
```

然后创建以下文件名称`logs_json.log` 并更换`path` 在您的文件源中`pipeline.yaml` 文件包含包含以下JSON数据的文件路径：

```
{"log": "07-25-2023 10:00:00 ERROR: error message"}
```

这`dissect` 处理器将检索字段（`Date`，，，，`Time`，，，，`Log_Type`， 和`Message`） 来自`log` 消息，基于模式`%{Date} %{Time} %{Type}: %{Message}` 在管道中配置。

运行管道后，您应该收到以下标准输出：

```
{
    "log" : "07-25-2023 10:00:00 ERROR: Some error",
    "Date" : "07-25-2023"
    "Time" : "10:00:00"
    "Log_Type" : "ERROR"
    "Message" : "error message"
}
```

## 配置

您可以配置`dissect` 带有以下选项的处理器。

| 选项| 必需的| 类型| 描述|
| ：--- | ：--- | ：--- | ：--- |
| `map` | 是的| 地图| 定义`dissect` 特定键的模式。有关如何在`dissect` 模式，请参阅[字段符号](#field-notations)。|
| `target_types` | 不| 地图| 指定提取字段的数据类型。有效的选项是`integer`，，，，`double`，，，，`string`， 和`boolean`。默认情况下，所有字段均为`string` 类型。|
| `dissect_when` | 不| 细绳| 指定执行的条件`dissect` 使用a的操作[数据预先表达]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/expression-syntax/)。如果指定，`dissect` 仅当表达式评估为true时，操作才会运行。|

### 字段符号

您可以定义`dissect` 具有以下字段类型的模式。

#### 普通场

没有后缀或前缀的字段。该字段将直接添加到输出事件中。格式为`%{field_name}`。

#### 跳过场

活动将不包含的字段。格式为`%{}` 或者`%{?field_name}`。

#### 附加字段

将与其他字段结合的字段。要附加多个值并在字段中包含最终值，请使用`+` 在字段名称之前`dissect` 图案。格式为`%{+field_name}`。

例如，使用模式`%{+field_name}, %{+field_name}`，日志消息`"foo, bar"` 会解析`{"field_name": "foobar"}`。

您也可以在后缀的帮助下定义串联的顺序`/<integer>`。

例如，具有模式`"%{+field_name/2}, %{+field_name/1}"`，日志消息`"foo, bar"` 会解析`{"field_name": "barfoo"}`。

如果未提及订单，则附加操作将按照在`dissect` 图案。

#### 间接字段

从另一个字段使用该值作为字段名称的字段。定义模式时，将字段前缀为`&` 为在字段中找到的值作为密钥中的键-价值对。

例如，具有模式`"%{?field_name}, %{&field_name}"`，日志消息`"foo, bar"` 会解析`{“foo”: “bar”}`。在日志消息中，`foo` 是从跳过场捕获的`%{?field_name}`。`foo` 然后用作从字段捕获的值的钥匙`%{&field_name}`。

#### 填充字段

右侧的带有桨板的字段已移除。这`->` 操作员可以用作后缀，以表明该字段之后的白色空间可以忽略。

例如，具有模式`%{field1->} %{field2}`，日志消息`“firstname    lastname”` 会解析`{“field1”: “firstname”, “field2”: “lastname”}`。

