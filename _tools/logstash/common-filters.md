---
layout: default
title: 通用过滤器插件
parent: Logstash
nav_order: 220
redirect_from:
 - /clients/logstash/common-filters/
---

# 通用过滤器插件

此页面包含一个通用过滤器插件的列表。

## 突变

您可以使用`mutate` 过滤以更改字段的数据类型。例如，您可以使用`mutate` 过滤如果您将事件发送到OpenSearch，则需要更改字段的数据类型以匹配任何现有的映射。

转换`quantity` 来自A的字段`string` 输入`integer` 类型：

```yml
input {
  http {
    host => "127.0.0.1"
    port => 8080
  }
}

filter {
  mutate {
   convert => {"quantity" => "integer"}
  }
}

output {
  file {
    path => "output.txt"
  }
}
```

#### 样本输出

您可以看到`quantity` 字段从一个`string` 到一个`integer`。

```yml
{
  "quantity" => 3,
  "host" => "127.0.0.1",
  "@timestamp" => 2021-05-23T19:02:08.026Z,
  "amount" => 10,
  "@version" => "1",
  "headers" => {
    "request_path" => "/",
    "connection" => "keep-alive",
    "content_length" => "41",
    "http_user_agent" => "PostmanRuntime/7.26.8",
    "request_method" => "PUT",
    "cache_control" => "no-cache",
    "http_accept" => "*/*",
    "content_type" => "application/json",
    "http_version" => "HTTP/1.1",
    "http_host" => "127.0.0.1:8080",
    "accept_encoding" => "gzip, deflate, br",
    "postman_token" => "ffd1cdcb-7a1d-4d63-90f8-0f2773069205"
   }
}
```

您可以转换的其他数据类型是`float`，，，，`string`， 和`boolean` 值。如果您通过数组，`mutate` 过滤器转换数组中的所有元素。如果你通过`string` 喜欢"world" 铸造`integer` 类型，结果为0，LogStash继续处理事件。

LogStash支持所有过滤器插件的一些常见选项：

选项| 描述
：--- | ：---
`add_field` | 在活动中添加一个或多个字段。
`remove_field` | 从现场删除一个或多个事件。
`add_tag` | 在活动中添加一个或多个标签。您可以使用标签根据其包含的标签执行有条件处理。
`remove_tag` | 从活动中删除一个或多个标签。

例如，您可以删除`host` 活动的字段：

```yml
input {
  http {
    host => "127.0.0.1"
    port => 8080
  }
}

filter {
  mutate {
    remove_field => {"host"}
  }
}

output {
  file {
    path => "output.txt"
  }
}
```

## 格罗克

与`grok` 过滤器，您可以解析非结构化数据并将其构造到字段中。这`grok` 过滤器使用文本模式与日志中的文本匹配。您可以将文本模式视为包含正则表达式的变量。

文本模式的格式如下：

```bash
%{SYNTAX:SEMANTIC}
```

`SYNTAX` 是格式的文本应该符合图案。您可以输入任何`grok`的预定义图案。例如，您可以使用电子邮件标识符匹配给定文本的电子邮件地址。

`SEMANTIC` 是匹配文本的任意名称。例如，如果您使用的是电子邮件标识符语法，则可以将其命名为“电子邮件”。

以下请求包括访问者的IP地址，访问者的名称，请求的时间戳，HTTP动词和URL，HTTP状态代码以及字节数：

```bash
184.252.108.229 - joe [20/Sep/2017:13:22:22 +0200] GET /products/view/123 200 12798
```

将此请求分为不同的字段：

```yml
filter {
  grok {
   match => { "message" => " %{IP: ip_address} %{USER:identity}
                             %{USER:auth} \[%{HTTPDATE:reg_ts}\]
                             \"%{WORD:http_verb}
                             %{URIPATHPARAM: req_path}
                             \" %{INT:http_status:int}
                             %{INT:num_bytes:int}"}
  }
}
```

在哪里：

- `IP`：匹配IP地址字段。
- `USER`：匹配用户名。
- `WORD`：匹配HTTP动词。
- `URIPATHPARAM`：匹配URI路径。
- `INT`：匹配HTTP状态字段。
- `INT`：匹配字节数。

这就是事件的样子`grok` 过滤器将其分解为各个字段：

```yml
ip_address: 184.252.108.229
identity: joe
reg_ts: 20/Sep/2017:13:22:22 +0200
http_verb:GET
req_path: /products/view/123
http_status: 200
num_bytes: 12798
```

对于常见日志格式，您使用此处定义的预定义模式⁠---[logstash模式](https://github.com/logstash-plugins/logstash-patterns-core/blob/main/patterns/ecs-v1)。您可以对结果进行任何调整`mutate` 筛选。

