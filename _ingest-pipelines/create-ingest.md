---
layout: default
title: 创建管道
nav_order: 10
redirect_from:
  - /opensearch/rest-api/ingest-apis/create-update-ingest/
  - /api-reference/ingest-apis/create-ingest/
---

# 创建流水线
**引入 1.0**
{: .label .label-purple }

使用创建管道 API 操作在 OpenSearch 中创建或更新管道。请注意，管道要求你至少定义一个指定如何更改文档的处理器。

## 路径和 HTTP 方法

替换为 `<pipeline-id>` 管道 ID：

```json
PUT _ingest/pipeline/<pipeline-id>
```
#### 示例请求

下面是一个 JSON 格式的示例，用于创建包含两个 `set` 处理器和一个 `uppercase` 处理器的引入管道。第一个 `set` 处理器将 `grad_year` 设置为 `2023`，第二个 `set` 处理器设置为 `graduated` `true`。 `uppercase` 处理器将 `name` 字段转换为大写。

```json
PUT _ingest/pipeline/my-pipeline
{
  "description": "This pipeline processes student data",
  "processors": [
    {
      "set": {
        "description": "Sets the graduation year to 2023",
        "field": "grad_year",
        "value": 2023
      }
    },
    {
      "set": {
        "description": "Sets graduated to true",
        "field": "graduated",
        "value": true
      }
    },
    {
      "uppercase": {
        "field": "name"
      }
    }
  ]
}
```
{% include copy-curl.html %}

要了解有关错误处理的详细信息，请参阅[处理管道故障]({{site.url}}{{site.baseurl}}/api-reference/ingest-apis/pipeline-failures/)。

## 请求正文字段

下表列出了用于创建或更新管道的请求正文字段。

参数 | 必需 | 类型 | 描述
:--- | :--- | :--- | :---
 `processors` | 必需 | 处理器对象数组 | 一组处理器，每个处理器转换文档。处理器按指定的顺序按顺序运行。
 `description` | 可选 | 字符串 | 引入管道的说明。

## 路径参数

参数 | 必需 | 类型 | 描述
:--- | :--- | :--- | :---
 `pipeline-id` | 必需 | 字符串 | 分配给引入管道的唯一标识符或管道 ID。

## 查询参数

参数 | 必需 | 类型 | 描述
:--- | :--- | :--- | :---
 `cluster_manager_timeout` | 可选 | 时间 | 等待与集群管理器节点建立连接的时间段。默认值为 30 秒。
 `timeout` | 可选 | 时间 | 等待响应的时间段。默认值为 30 秒。

## 模板片段

某些处理器参数支持[Mustache](https://mustache.github.io/)模板代码段。若要获取字段的值，请将字段名称括在三个大括号中，例如 `{% raw %}{{{field-name}}}{% endraw %}`.

#### 示例： `set` 使用 Mustache 模板代码段引入处理器

以下示例使用以下值 `{% raw %}{{{tenure}}}{% endraw %}` 设置字段 `{% raw %}{{{role}}}{% endraw %}`：

```json
PUT _ingest/pipeline/my-pipeline
{
 "processors": [
    {
      "set": {
        "field": "{% raw %}{{{role}}}{% endraw %}",
        "value": "{% raw %}{{{tenure}}}{% endraw %}"
         }
    }
  ]
}
```
{% include copy-curl.html %}
