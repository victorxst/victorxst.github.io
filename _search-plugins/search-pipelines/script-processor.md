---
layout: default
title: 脚本
nav_order: 30
has_children: false
parent: 搜索处理器
grand_parent: 搜索管道
---

# 脚本处理器

这`script` 搜索请求处理器拦截搜索请求，并添加一个在传入请求上运行的无痛脚本。脚本只能在以下请求字段上运行：

- `from` 
- `size` 
- `explain` 
- `version` 
- `seq_no_primary_term` 
- `track_scores`  
- `track_total_hits` 
- `min_score` 
- `terminate_after` 
- `profile` 

有关请求字段定义，请参阅[搜索请求字段]({{site.url}}{{site.baseurl}}/api-reference/search#request-body)。

## 请求字段

下表列出了所有可用的请求字段。

场地| 数据类型| 描述
:--- | :--- | :---
`source` | 内联脚本| 要运行的脚本。必需的。
`lang` | 细绳| 脚本语言。选修的。仅有的`painless` 得到支持。
`tag` | 细绳| 处理器的标识符。选修的。
`description` | 细绳| 处理器的描述。选修的。
`ignore_failure` | 布尔| 如果`true`，OpenSearch[忽略任何故障]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/creating-search-pipeline/#ignoring-processor-failures) 该处理器并继续在搜索管道中运行其余的处理器。选修的。默认为`false`。

## 例子

以下请求会创建一个使用`script` 请求处理器。脚本限制得分解释仅为一个文档，因为`explain` 是一个昂贵的操作：

```json
PUT /_search/pipeline/explain_one_result
{
  "description": "A pipeline to limit the explain operation to one result only",
  "request_processors": [
    {
      "script": {
        "lang": "painless",
        "source": "if (ctx._source['size'] > 1) { ctx._source['explain'] = false } else { ctx._source['explain'] = true }"
      }
    }
  ]
} 
```
{% include copy-curl.html %}

