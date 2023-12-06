---
layout: default
title: Grok
parent: 摄入的处理器
grand_parent: 摄取管道
nav_order: 140
---

# Grok

这`grok` 处理器用于使用模式匹配来解析和构建非结构化数据。您可以使用`grok` 从日志消息，Web服务器访问日志，应用程序日志和其他遵循一致格式的日志数据中提取字段的处理器。

## Grok Basics

这`grok` 处理器使用一组预定义的模式与输入文本的一部分匹配。每个模式都由名称和正则表达式组成。例如，模式`%{IP:ip_address}` 匹配IP地址并将其分配给字段`ip_address`。您可以组合多种模式以创建更复杂的表达式。例如，模式`%{IP:client} %{WORD:method} %{URIPATHPARM:request} %{NUMBER:bytes %NUMBER:duration}` 匹配Web服务器访问日志的行，并提取客户端IP地址，HTTP方法，请求URI，发送的字节数以及请求的持续时间。

这`grok` 处理器建立在[Oniguruma正则表达式库](https://github.com/kkos/oniguruma/blob/master/doc/RE) 并支持该库中的所有模式。您可以使用[Grok Debugger](https://grokdebugger.com/) 测试和调试您的Grok表达式的工具。

## Grok处理器语法

以下是`grok` 处理器：

```json
{
  "grok": {
    "field": "your_message",
    "patterns": ["your_patterns"]
  }
}
```
{% include copy-curl.html %}

## 配置参数

配置`grok` 处理器，您有各种选项，使您可以定义模式，匹配特定的密钥并控制处理器的行为。下表列出了所需的和可选参数`grok` 处理器。

范围| 必需的| 描述|
|-----------|-----------|-----------|
`field`  | 必需的| 包含应解析的文本的字段的名称。|
`patterns`  | 必需的| 用于匹配和提取的命名捕获的Grok表达式列表。返回列表中的第一个匹配表达式。| 
`pattern_definitions`  | 选修的| 用于定义当前处理器的自定义模式的图案名称和图案元素的字典。如果图案与现有名称匹配，则覆盖了PRE-现有定义。|
`trace_match` | 选修的| 当参数设置为`true`，处理器添加了一个名称的字段`_grok_match_index` 到处理的文档。该字段包含模式的索引`patterns` 成功匹配文档的数组。此信息对于调试和了解将哪种模式应用于文档可能很有用。默认为`false`。|
`description` | 选修的| 处理器的简要说明。|
`if` | 选修的| 运行此处理器的条件。|
`ignore_failure` | 选修的| 如果设置为`true`，失败被忽略。默认为`false`。|
`ignore_missing` | 选修的| 如果设置为`true`，如果字段不存在或为`null`。默认为`false`。|
`on_failure` | 选修的| 如果处理器失败，则可以运行的处理器列表。|
`tag` | 选修的| 处理器的标识符标签。可用于调试以区分同一类型的处理器。|

## 创建管道

以下步骤指导您创建一个[摄入管道]({{site.url}}{{site.baseurl}}/ingest-pipelines/index/) 与`grok` 处理器。

**步骤1：创建管道。** 

以下查询创建了一个命名的管道`log_line`。它从`message` 使用指定模式的文档字段。在这种情况下，它提取`clientip`，，，，`timestamp`， 和`response_status` 字段：

```json
PUT _ingest/pipeline/log_line
{
  "description": "Extract fields from a log line",
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": ["%{IPORHOST:clientip} %{HTTPDATE:timestamp} %{NUMBER:response_status:int}"]
      }
    }
  ]
}
```
{% include copy-curl.html %}

**步骤2（可选）：测试管道。**

{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/icons/alert-icon.png" class ="inline-icon" alt ="alert icon"/> {:/}**笔记**<br>建议您在摄入文档之前测试管道。
{: .note}

要测试管道，请运行以下查询：

```json
POST _ingest/pipeline/log_line/_simulate
{
  "docs": [
    {
      "_source": {
        "message": "127.0.0.1 198.126.12 10/Oct/2000:13:55:36 -0700 200"
      }
    }
  ]
}
```
{% include copy-curl.html %}

#### 回复

以下响应证实了管道正常工作：

```json
{
  "docs": [
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "message": "127.0.0.1 198.126.12 10/Oct/2000:13:55:36 -0700 200",
          "response_status": 200,
          "clientip": "198.126.12",
          "timestamp": "10/Oct/2000:13:55:36 -0700"
        },
        "_ingest": {
          "timestamp": "2023-09-13T21:41:52.064540505Z"
        }
      }
    }
  ]
}
```

**步骤3：摄取文档。**

以下查询将文档摄入到名为的索引中`testindex1`：

```json
PUT testindex1/_doc/1?pipeline=log_line
{
  "message": "127.0.0.1 198.126.12 10/Oct/2000:13:55:36 -0700 200"
}
```
{% include copy-curl.html %}

**步骤4（可选）：检索文档。**

要检索文档，请运行以下查询：

```json
GET testindex1/_doc/1
```
{% include copy-curl.html %}

## 自定义模式

您可以使用默认模式，也可以使用该模式添加自定义模式`patterns_definitions` 范围。自定义的Grok模式可以在管道中使用，以从不匹配构建的日志消息中提取结构化数据-在Grok模式中。这对于从自定义应用程序中解析日志消息或解析已修改的日志消息可能很有用。自定义模式遵循直接的结构：每个模式都有一个唯一的名称和相应的正则表达式来定义其匹配行为。

以下是如何在配置中包含自定义模式的示例。在此示例中，发行编号在3至4位数字之间，并分解为`issue_number` 字段和状态被解析为`status` 场地：

```json
PUT _ingest/pipeline/log_line
{
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": ["The issue number %{NUMBER:issue_number} is %{STATUS:status}"],
        "pattern_definitions" : {
          "NUMBER" : "\\d{3,4}",
          "STATUS" : "open|closed"
        }
      }
    }
  ]
}
```
{% include copy-curl.html %}

## 跟踪哪些模式匹配

要追踪哪些模式匹配并填充了字段，您可以使用`trace_match` 范围。以下是如何在配置中包含此参数的示例：

```json
PUT _ingest/pipeline/log_line  
{  
  "description": "Extract fields from a log line",  
  "processors": [  
    {  
      "grok": {  
        "field": "message",  
        "patterns": ["%{HTTPDATE:timestamp} %{IPORHOST:clientip}", "%{IPORHOST:clientip} %{HTTPDATE:timestamp} %{NUMBER:response_status:int}"],  
        "trace_match": true  
      }  
    }  
  ]  
}  
```
{% include copy-curl.html %}

当您模拟管道时，OpenSearch返回`_ingest` 元数据包括`grok_match_index`，如以下输出所示：

```json
{
  "docs": [
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "message": "127.0.0.1 198.126.12 10/Oct/2000:13:55:36 -0700 200",
          "response_status": 200,
          "clientip": "198.126.12",
          "timestamp": "10/Oct/2000:13:55:36 -0700"
        },
        "_ingest": {
          "_grok_match_index": "1",
          "timestamp": "2023-11-02T18:48:40.455619084Z"
        }
      }
    }
  ]
}
```


