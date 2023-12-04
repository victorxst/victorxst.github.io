---
layout: default
title: 获取存储的脚本
parent: 脚本API
nav_order: 3
---

# 获取存储的脚本
**引入1.0**
{: .label .label-purple }

检索存储的脚本。

## 路径参数

| 范围| 数据类型| 描述| 
:--- | :--- | :---
| 脚本| 细绳| 存储的脚本或搜索模板名称。必需的。|

## 查询参数

| 范围| 数据类型| 描述| 
:--- | :--- | :---
| cluster_manager_timeout| 时间| 等待与群集管理器连接的时间。可选，默认为`30s`。|

#### 示例请求

以下取回`my-first-script` 存储的脚本。

````json
GET _scripts/my-first-script
````
{% include copy-curl.html %}

#### 示例响应

这`GET _scripts/my-first-script` 请求返回以下字段：

````json
{
  "_id" : "my-first-script",
  "found" : true,
  "script" : {
    "lang" : "painless",
    "source" : """
          int total = 0;
          for (int i = 0; i < doc['ratings'].length; ++i) {
            total += doc['ratings'][i];
          }
          return total;
        """
  }
}
````

## 响应字段

这`GET _scripts/my-first-script` 请求返回以下响应字段：

| 场地| 数据类型| 描述| 
:--- | :--- | :---
| _ID| 细绳| 脚本的名称。|
| 成立| 布尔| 请求的脚本存在并被检索。|
| 脚本| 目的| 脚本定义。看[脚本对象](#script-object)。|

#### 脚本对象

| 场地| 数据类型| 描述| 
:--- | :--- | :---
| 朗| 细绳| 脚本的语言。|
|  来源| 细绳| 剧本的身体。

