---
layout: default
title: 高级配置
parent: Logstash
nav_order: 230
redirect_from:
 - /clients/logstash/advanced-config/
---

# 高级配置

本节介绍了如何为logstash设置高级配置选项，例如引用字段值和条件语句。

## 引用字段值

要访问字段，请使用`- field` 句法。
您也可以通过Square Brackets包围字段名称`- [field]` 这使您更明确地提到了一个字段。


例如，如果您有以下事件：

```bash
{
  "request": "/products/view/123",
  "verb": "GET",
  "response": 200,
  "headers": {
  "request_path" => "/"
  }
}
```

访问`request` 字段，使用`- request` 或者`- [request]`。

如果要引用嵌套字段，请使用方括号语法并指定通往字段的路径。每个级别都被封闭在方括号内：`- [headers][request_path]`。

您可以使用`sprintf` 格式。这也称为字符串扩展。您需要添加一个％符号，然后将字段参考包装在卷曲括号中。

使用条件语句时，您需要参考字段值。

例如，您可以使文件名动态并包含已处理事件的类型- 任何一个`access` 或者`error`。这`type` 选项主要用于根据正在处理的事件的类型来条件应用过滤器插件。

让我们添加一个`type` 选项并指定一个值`access`。


```yml
input {
  file {
    path => ""
  start_position => "beginning"
  type => "access"
  }
  http {
    type => "access"
  }
}

filter {
  mutate {
    remove_field => {"host"}
  }
}

output {
  stdout {
    codec => rubydebug
  }
file {
  path => "%{[type]}.log"
  }
}
```

启动LogStash并发送HTTP请求。处理的事件是在终端中输出的。该活动现在包括一个名为的字段`type`。

你会看到的`access.log` 在LogStash目录中创建的文件。

## 有条件的语句

您可以使用条件语句根据某些条件来控制代码执行的流程。

句法：

```yml
if EXPR {
  ...
} else if EXPR {
  ...
} else {
  ...
}
```

`EXPR` 是评估布尔值的任何有效的LogStash语法。
例如，您可以检查事件类型是否设置为`access` 或者`error` 并基于此行动：

```yml
if [type] == "access" {
...
} else if [type] == "error" {
file { .. }
} else {
...
}
```

您可以将字段值与某些任意值进行比较：

```yml
if [headers][content_length] >= 1000 {
...
}
```

您可以将REGEX：

```yml
if [some_field =~ /[0-9]+/ {
  //some field only contains digits
}
```

您可以使用数组：

```yml
if [some_field] in ["one", "two", "three"] {
  some field is either "one", "two", or "three"
}
```

您可以使用布尔运营商：

```yml
if [type] == "access" or [type] == "error" {
  ...
}
```


## 格式日期

您可以使用`sprintf` 格式或字符串扩展到格式日期。
例如，您可能希望当前日期成为文件名的一部分。

要格式化日期，请在卷曲括号中添加一个加号，然后添加日期格式- `%{+yyyy-MM-dd}`。

```yml
file {
  path => "%{[type]}_%{+yyyy_MM_dd}.log"
}
```

这是存储在@timestamp字段中的日期，这是事件的时间和日期。
将请求发送到管道，并验证输出包含事件日期的文件名。

您也可以将日期嵌入其他输出中，例如OpenSearch中的索引名称。

## 发送时间信息

您可以设置事件的时间。

LogStash已经设置了@timestamp字段中输入插件接收事件的时间。
在某些情况下，您可能需要使用其他时间戳。
例如，如果您有一家电子商务商店，并且每天在午夜处理订单。当Logstash在午夜接收事件时，它将时间戳设置为当前时间。
但是您希望它是下订单的时间，而不是Logstash收到事件的时间。

让我们将事件时间戳更改为Web服务器收到请求的日期。您可以使用名为的过滤器插件来执行此操作`dates`。
这`dates` 过滤器通过`date` 或者`datetime` 从字段中值，并将结果用作事件时间戳。

添加`date` 插入底部`filter` 堵塞：

```yml
date {
  match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
}
```

时间戳是`grok` 图案创建。
`Z` 是时区。即UTC偏移。

启动LogStash并发送HTTP请求。

您可以看到该文件名包含请求的日期，而不是当前日期。

如果日期的通过失败，`filter` 插件添加了一个名称的标签`_datepassfailure` 到文本字段。

将@timestamp字段设置为新值后，您实际上并不需要其他`timestamp` 字段了。您可以用`remove_field` 选项。

```yml
date {
  match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
  remove_field => [ "timestamp" ]
}
```

## 解析用户代理

用户代理是日志条目的最后一部分，它由浏览器的名称，浏览器版本和设备的OS组成。

用户可能正在使用各种浏览器，设备和操作系统。手动执行此操作很难。

你不能使用`grok` 模式是因为`grok` 模式仅与字符串中的用法完全匹配，并且不知道访问者使用的哪个浏览器。

logstash用一个包含正则表达式的文件寄出。这使得提取用户代理信息非常容易，您可以将其发送到OpenSearch并运行聚合。

为此，添加一个`source` 包含字段名称的选项。在这种情况下，这就是`agent` 场地。
默认情况下，用户代理插件在顶部添加了许多字段-事件的水平。
由于那会变得非常困惑，我们可以添加一个名称的选项`target` 有价值`ua`，用户代理的缩写。这样做的是它将字段嵌套在一个名为的对象中`ua`，使事情变得更有条理。

```yml
useragent {
  source => "agent"
  target => "ua"
}
```

启动LogStash并发送HTTP请求。

您可以看到一个名为的字段`ua` 带有许多键，包括浏览器名称和版本，操作系统和设备。

您可以使用OpenSearch仪表板创建一个饼图，该饼图显示有多少访问者正在使用移动设备以及桌面用户有多少个。或者，您可以获取浏览器版本流行的统计信息。

## 丰富地理数据

您可以使用IP地址并执行地理查找，以解决用户的地理位置`geoip` 筛选。

这`geoip` 过滤插件带有一个名为的数据库`geolite 2`，由一家名为Maxmind的公司提供。`geolite 2` 是地理数据的流行来源，可以免费使用。
添加`geoip` 插入底部`else` 堵塞。

价值`source` 选项是包含IP地址的字段的名称，在这种情况下是`clientip`。您可以使用`grok` 图案。

```yml
geoip {
  source => "clientip"
}
```

启动LogStash并发送HTTP请求。

在终端中，您会看到一个名为的新字段`geoip` 其中包含诸如时区，国家，大陆，城市，邮政编码和纬度/经度对等信息。

如果您只需要国家名称，请包括一个名称的选项`fields` 带有您想要的字段名称的数组`geoip` 插件要返回。

某些字段（例如城市名称和地区）并不总是可用的，因为将IP地址转换为地理位置通常不是那么准确。如果是`geoip` 插件无法查找地理位置，它添加了一个名称的标签`geoip_lookup_failure`。

您可以使用`geoip` 带有OpenSearch输出的插件是因为`location` 对象`geoip` 对象是用于表示JSON中地理空间数据的标准格式。这与OpenSearch使用的格式相同`geo_point` 数据类型。

您可以使用强大的opensearch地理空间查询来处理地理数据。

