---
layout: default
title: 检索搜索管道
nav_order: 25
has_children: false
parent: 搜索管道
grand_parent: 搜索
---

# 检索搜索管道

要检索现有搜索管道的详细信息，请使用搜索管道API。

要查看所有搜索管道，请使用以下请求：

```json
GET /_search/pipeline
```
{％包含副本-curl.html％}

响应包含您在上一节中设置的管道：
<详细信息打开降价="block">
  <summary>
    回复
  </summary>
  {： 。文本-三角洲}

```json
{
  "my_pipeline" : {
    "request_processors" : [
      {
        "filter_query" : {
          "tag" : "tag1",
          "description" : "This processor is going to restrict to publicly visible documents",
          "query" : {
            "term" : {
              "visibility" : "public"
            }
          }
        }
      }
    ]
  }
}
```
</delect>

要查看特定管道，请将管道名称指定为路径参数：

```json
GET /_search/pipeline/my_pipeline
```
{％包含副本-curl.html％}

您也可以使用通配符模式查看管道的子集，例如：

```json
GET /_search/pipeline/my*
```
{％包含副本-curl.html％}

