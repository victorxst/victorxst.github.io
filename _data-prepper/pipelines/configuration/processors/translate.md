---
layout: default
title: 翻译
parent: 处理器
grand_parent: 管道
nav_order: 117
---

# 翻译

这`translate` 处理器将事件中的值转换为预配置的值。

## 基本用法

使用`translate` 处理器，创建以下`pipeline.yaml` 文件：

```yaml
translate-pipeline:
  source:
    file:
      path: "/full/path/to/logs_json.log"
      record_type: "event"
      format: "json"
  processor:
    - translate:
        mappings:
          - source: "status"
            targets:
              - target: "translated_result"
                map:
                  404: "Not Found"
  sink:
    - stdout:
```

然后创建以下文件名称`logs_json.log` 并更换`path` 在您的文件源中`pipeline.yaml` 文件包含包含以下JSON数据的文件路径：

```json
{ "status": "404" }
```

这`translate` 处理器配置在`pipeline.yaml` 检索`source` 事件数据的值，并将其与下指定的键进行比较`targets`。
找到匹配时，处理器将相应的映射值放入`target` 配置中提供的密钥。

当您使用以前的数据进行数据预先`pipeline.yaml` 文件，您应该收到以下输出：

```json
{
  "status": "404",
  "translated_result": "Not Found"
}
```

## 高级选项

下面的示例显示了更多涉及的映射，并具有其他配置`translate` 处理器：

```yaml
processor:
  - translate:
      mappings:
        - source: "status"
          targets:
            - target: "translated_result"
              map:
                404: "Not Found"
              default: "default"
              type: "string"
              translate_when: "/response != null"
            - target: "another_translated_result"
              regex:
                exact: false
                patterns:
                  "2[0-9]{2}" : "Success" # Matches ranges from 200-299
                  "5[0-9]{2}": "Error"    # Matches ranges form 500-599
      file: 
        name: "path/to/file.yaml"
        aws:
          bucket: my_bucket
          region: us-east-1
          sts_role_arn: arn:aws:iam::123456789012:role/MyS3Role
```

在最高级别，指定`mappings` 用于内联映射配置，或`file` 从文件中提取映射配置。两个都`mappings` 和`file` 可以一起指定选项，并且处理器考虑了两个来源的映射以进行翻译。在管道配置和文件映射共享重复的情况下`source` 和`target` 对，管道配置中指定的映射优先。


## 配置

您可以使用以下选项来配置`translate` 处理器。

| 范围| 必需的| 类型| 描述|
| :--- | :---  | :--- | :--- |
| 映射| 不| 列表| 定义内联映射。有关更多信息，请参阅[映射](#mappings)。|
| 文件| 不| 地图| 指向包含映射配置的文件。有关更多信息，请参阅[文件](#file)。|

### 映射

每个项目`mappings` 配置包含以下选项。

| 范围| 必需的| 类型| 描述|
| :--- | :--- | :--- | :--- |
| 来源| 是的| 字符串或列表| 源字段要翻译。可以是字符串或字符串列表。|
| 目标| 是的| 列表|  目标字段配置的列表，例如目标字段键或翻译图。|

每个项目`targets` 配置包含以下选项。

| 范围| 必需的| 类型| 描述|
| :--- | :---  | :--- | :--- |
| 目标| 是的| 细绳| 指定将放置翻译值的输出中字段的键。|
| 地图| 不| 地图| 关键列表-定义翻译的价值对。每个键代表源字段中可能的值，相应的值表示应将其翻译成的值。有关示例，请参见[地图选项](#map-option)。至少之一`map` 和`regex` 应配置。|
| 正则| 不| 地图| 定义翻译图的键图。有关更多选项，请参阅[REGEX选项](#regex-option)。至少之一`map` 和`regex` 应配置。|
| 默认| 不| 细绳| 在翻译过程中找不到匹配时要使用的默认值。|
| 类型| 不| 细绳| 指定目标值的数据类型。|
| translate_when| 不| 细绳| 使用[数据预先表达]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/expression-syntax/) 指定执行翻译的条件。指定时，只有在满足条件时，表达式才会翻译。|

#### 地图选项

使用地图选项时，您可以使用以下密钥类型：

*单个钥匙
  ```yaml
    map:
      ok : "Success"
      120: "Found"
  ```
*数字范围
  ```yaml
    地图：
      "100-200"："Success"
      "400-499"："Error"
  ```
* Comma-delimited keys
  ```yaml
    地图：
      "key1,key2,key3"："value1"
      "100-200,key4"："value2"
  ```

When configuring the keys inside the `地图` option, do not use any overlapping number ranges or duplicate keys.

#### regex option

You can use the following options with the `正则` 选项。

| 范围| 必需的| 类型| 描述|
| :--- | :---  | :--- | :--- |
| 模式| 是的| 地图| 密钥地图-值对定义键的正则键模式和要转化为每个模式的值。|
| 精确的| 不| 布尔| 是在Regex模式上使用完整的字符串匹配还是部分字符串匹配。如果`true`，仅当整个密钥与模式匹配时，该模式才被视为匹配。否则，当一个子时，该模式被认为是匹配的-钥匙的字符串与模式匹配。|

### 文件

这`file` 选项`translate` 处理器采用本地YAML文件或包含翻译映射的Amazon简单存储服务（Amazon S3）对象。文件的内容应采用以下格式：
```yaml
mappings:
  - source: "status"
    targets:
      - target: "result"
        map:
          "foo": "bar"
        # Other configurations
```

您可以在`file` 配置。

| 范围| 必需的| 类型| 描述|
| :--- | :---  | :--- | :--- |
| 姓名| 是的| 细绳| S3对象的本地文件或密钥名称的完整路径。|
| AWS| 不| 地图| 当文件是S3对象时，AWS配置。有关更多信息，请参见下表。|

您可以使用以下选项与`aws` 配置。

| 范围| 必需的| 类型| 描述|
| :--- | :---  | :--- | :--- |
| `bucket` | 是的| 细绳| Amazon S3存储桶名。|
| `region` | 是的| 细绳| 用于凭证的AWS区域。|
| `sts_role_arn` | 是的| 细绳| AWS安全令牌服务（AWS STS）将要担任亚马逊S3的请求。|

