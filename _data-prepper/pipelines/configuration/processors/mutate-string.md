---
layout: default
title: 变异字符串
parent: 处理器
grand_parent: 管道
nav_order: 70
---

# 变异字符串处理器

您可以通过使用突变字符串进程器来更改字符串出现的方式。例如，您可以使用`uppercase_string` 处理器将字符串转换为大写，您可以使用`lowercase_string` 处理器将字符串转换为小写。以下是允许您突变字符串的处理器列表：

*[替代_STRING](#substitute_string)
*[split_string](#split_string)
*[uppercase_string](#uppercase_string)
*[lowercase_string](#lowercase_string)
*[trim_string](#trim_string)

## 替代_STRING

这`substitute_string` 处理器将密钥的值与正则表达式（REGEX）匹配，并用替换字符串替换所有返回的匹配项。

### 配置

您可以配置`substitute_string` 带有以下选项的处理器。

选项| 必需的| 描述
:--- | :--- | :---
`entries` | 是的| 添加到事件的条目列表。|
`source` | 是的| 要修改的钥匙。|
`from` | 是的| 将要替换的正则弦字符串。特殊的正则角色，例如`[` and `]( must be escaped using `\\` when using double quotes and `\` when using single quotes. For more information, see [Class Pattern](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/regex/Pattern.html) 在Java文档中。|
`to` | 是的| 替换每场比赛的字符串`from`。|

### 用法

首先，创建以下内容`pipeline.yaml` 文件：

```yaml
pipeline:
  source:
    file:
      path: "/full/path/to/logs_json.log"
      record_type: "event"
      format: "json"
  processor:
    - substitute_string:
        entries:
          - source: "message"
            from: ":"
            to: "-"
  sink:
    - stdout:
```
{% include copy.html %}

接下来，创建一个名为的日志文件`logs_json.log`。之后，更换`path` 您的文件源中的`pipeline.yaml` 用您的文件路径进行文件。有关更多详细信息，请参阅[配置数据预备]({{site.url}}{{site.baseurl}}/data-prepper/getting-started/#2-configuring-data-prepper)。

在运行PEDPPPEPPEPPEPPEPPEPPEPPER之前，该源以以下格式出现：

```json
{"message": "ab:cd:ab:cd"}
```

运行Data Prepper后，源将转换为以下格式：

```json
{"message": "ab-cd-ab-cd"}
```

`from` 定义更换哪个字符串，并且`to` 定义替换的字符串`from` 细绳。在上一个示例中，字符串`ab:cd:ab:cd` 变成`ab-cd-ab-cd`。如果是`from` Regex String不会返回匹配项，键将返回而没有任何更改。
    
## split_string

这`split_string` 处理器使用定界符字符将字段分为数组。

### 配置

您可以配置`split_string` 带有以下选项的处理器。

选项| 必需的| 描述
:--- | :--- | :---
 `entries` | 是的| 添加到事件的条目列表。|
 `source` | 是的| 要分开的钥匙。|
 `delimiter` | 不| 分离器角色负责拆分。不能与`delimiter_regex`。至少`delimiter` 或者`delimiter_regex` 必须定义。|
`delimiter_regex` | 不| 负责拆分的正则弦字符串。不能与`delimiter`。任何一个`delimiter` 或者`delimiter_regex` 必须定义。|

### 用法

首先，创建以下内容`pipeline.yaml` 文件：

```yaml
pipeline:
  source:
    file:
      path: "/full/path/to/logs_json.log"
      record_type: "event"
      format: "json"
  processor:
    - split_string:
        entries:
          - source: "message"
            delimiter: ","
  sink:
    - stdout:
```
{% include copy.html %}

接下来，创建一个名为的日志文件`logs_json.log`。之后，更换`path` 在您的文件源中`pipeline.yaml` 用您的文件路径进行文件。有关更多详细信息，请参阅[配置数据预备]({{site.url}}{{site.baseurl}}/data-prepper/getting-started/#2-configuring-data-prepper)。

在运行PEDPPPEPPEPPEPPEPPEPPEPPER之前，该源以以下格式出现：

```json
{"message": "hello,world"}
```
运行Data Prepper后，源将转换为以下格式：

```json
{"message":["hello","world"]}
```

## uppercase_string

这`uppercase_string` 处理器将键的值（字符串）从其当前情况转换为大写。

### 配置

您可以配置`uppercase_string` 带有以下选项的处理器。

选项| 必需的| 描述
:--- | :--- | :---
 `with_keys` | 是的| 转换为大写的密钥列表。|

### 用法

首先，创建以下内容`pipeline.yaml` 文件：

```yaml
pipeline:
  source:
    file:
      path: "/full/path/to/logs_json.log"
      record_type: "event"
      format: "json"
  processor:
    - uppercase_string:
        with_keys:
          - "uppercaseField"
  sink:
    - stdout:
```
{% include copy.html %}

接下来，创建一个名为的日志文件`logs_json.log`。之后，更换`path` 在您的文件源中`pipeline.yaml` 使用正确的文件路径文件。有关更多详细信息，请参阅[配置数据预备]({{site.url}}{{site.baseurl}}/data-prepper/getting-started/#2-configuring-data-prepper)。

在运行PEDPPPEPPEPPEPPEPPEPPEPPER之前，该源以以下格式出现：

```json
{"uppercaseField": "hello"}
```
运行Data Prepper后，源将转换为以下格式：

```json
{"uppercaseField": "HELLO"}
```

## lowercase_string

这`lowercase string` 处理器将字符串转换为小写。

### 配置

您可以配置`lowercase string` 带有以下选项的处理器。

选项| 必需的| 描述
:--- | :--- | :---
 `with_keys` | 是的| 转换为小写的键列表。|

### 用法

首先，创建以下内容`pipeline.yaml` 文件：

```yaml
pipeline:
  source:
    file:
      path: "/full/path/to/logs_json.log"
      record_type: "event"
      format: "json"
  processor:
    - lowercase_string:
        with_keys:
          - "lowercaseField"
  sink:
    - stdout:
```
{% include copy.html %}

接下来，创建一个名为的日志文件`logs_json.log`。之后，更换`path` 在您的文件源中`pipeline.yaml` 使用正确的文件路径文件。有关更多详细信息，请参阅[配置数据预备]({{site.url}}{{site.baseurl}}/data-prepper/getting-started/#2-configuring-data-prepper)。

在运行PEDPPPEPPEPPEPPEPPEPPEPPER之前，该源以以下格式出现：

```json
{"lowercaseField": "TESTmeSSage"}
```

运行Data Prepper后，源将转换为以下格式：

```json
{"lowercaseField": "testmessage"}
```

## trim_string

这`trim_string` 处理器从钥匙的开头和结尾处删除空格。

### 配置

您可以配置`trim_string` 带有以下选项的处理器。

选项| 必需的| 描述
:--- | :--- | :---
 `with_keys` | 是的| 钥匙列出的钥匙列表，从中修剪了空格。|

### 用法

首先，创建以下内容`pipeline.yaml` 文件：

```yaml
pipeline:
  source:
    file:
      path: "/full/path/to/logs_json.log"
      record_type: "event"
      format: "json"
  processor:
    - trim_string:
        with_keys:
          - "trimField"
  sink:
    - stdout:
```
{% include copy.html %}

接下来，创建一个名为的日志文件`logs_json.log`。之后，更换`path` 在您的文件源中`pipeline.yaml` 使用正确的文件路径文件。有关更多详细信息，请参阅[配置数据预备]({{site.url}}{{site.baseurl}}/data-prepper/getting-started/#2-configuring-data-prepper)。

在运行PEDPPPEPPEPPEPPEPPEPPEPPER之前，该源以以下格式出现：

```json
{"trimField": " Space Ship "}
```

运行Data Prepper后，源将转换为以下格式：

```json
{"trimField": "Space Ship"}
```

