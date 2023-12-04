---
layout: default
title: 执行简单脚本
parent: 脚本API
nav_order: 7
---

# 执行简单脚本
**引入1.0**
{: .label .label-purple }

执行无痛脚本API允许您运行未存储的脚本。

## 路径和HTTP方法

```json
GET /_scripts/painless/_execute
POST /_scripts/painless/_execute
```

## 请求字段

| 场地| 描述| 
:--- | :---
| 脚本| 要运行的脚本。必需的|
| 语境| 脚本的上下文。选修的。默认为`painless_test`。|
| context_setup| 为上下文指定其他参数。选修的。| 

#### 示例请求

以下请求使用默认`painless_context` 对于脚本：

```json
GET /_scripts/painless/_execute
{
  "script": {
    "source": "(params.x + params.y)/ 2",
    "params": {
      "x": 80,
      "y": 100
    }
  }
}
```
{% include copy-curl.html %}

#### 示例响应

响应包含两个脚本参数的平均值：

```json
{
  "result" : "90"
}
```

## 响应字段

| 场地| 描述| 
:--- | :--- 
| 结果| 脚本结果。|


## 脚本上下文

选择不同的上下文来控制脚本可用的变量以及结果的返回类型。默认上下文是`painless_test`。

## 无痛测试环境

这`painless_test` 上下文是默认脚本上下文，仅提供`params` 脚本可变。返回的结果始终转换为字符串。请参阅前面的示例请求以获取用法示例。

## 滤波器上下文

这`filter` 上下文运行脚本，就好像脚本在脚本查询中一样。您必须在上下文中提供测试文档。这`_source`，存储的字段，以及`_doc` 变量将用于脚本。

您可以在过滤器上下文中指定以下参数`context_setup`。

范围| 描述
:--- | :---
文档| 暂时将内存索引的文档可用于脚本。
指数| 包含文档映射的索引名称。

例如，首先创建一个带有测试文档映射的索引：

```json
PUT /testindex1
{
  "mappings": {
    "properties": {
      "grad": {
        "type": "boolean"
      },
      "gpa": {
        "type": "float"
      }
    }
  }
}
```
{% include copy-curl.html %}

运行脚本以确定学生是否有资格获得荣誉：

```json
POST /_scripts/painless/_execute
{
  "script": {
    "source": "doc['grad'].value == true && doc['gpa'].value >= params.min_honors_gpa",
    "params": {
      "min_honors_gpa": 3.5
    }
  },
  "context": "filter",
  "context_setup": {
    "index": "testindex1",
    "document": {
      "grad": true,
      "gpa": 3.79
    }
  }
}
```
{% include copy-curl.html %}

响应包含结果：

```json
{
  "result" : true
}
```

## 得分上下文

这`score` 上下文运行脚本，好像脚本在`script_score` 功能`function_score` 询问。

您可以在分数上下文中指定以下参数`context_setup`。

范围| 描述
:--- | :---
文档| 暂时将内存索引的文档可用于脚本。
指数| 包含文档映射的索引名称。
询问| 如果脚本使用`_score` 参数，查询可以指定用于使用`_score` 字段计算分数。

例如，首先创建一个带有测试文档映射的索引：

```json
PUT /testindex1
{
  "mappings": {
    "properties": {
      "gpa_4_0": {
        "type": "float"
      }
    }
  }
}
```
{% include copy-curl.html %}

运行一个将4.0比例尺转换为GPA的脚本，以作为参数提供的不同比例：

```json
POST /_scripts/painless/_execute
{
  "script": {
    "source": "doc['gpa_4_0'].value * params.max_gpa / 4.0",
    "params": {
      "max_gpa": 5.0
    }
  },
  "context": "score",
  "context_setup": {
    "index": "testindex1",
    "document": {
      "gpa_4_0": 3.5
    }
  }
}
```
{% include copy-curl.html %}

响应包含结果：

```json
{
  "result" : 4.375
}
```
