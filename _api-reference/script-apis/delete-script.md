---
layout: default
title: 删除脚本
parent: 脚本API
nav_order: 4
---

# 删除脚本
**引入1.0**
{: .label .label-purple }

删除存储的脚本

## 路径参数

路径参数是可选的。

| 范围| 数据类型| 描述| 
:--- | :--- | :---
| 脚本-ID| 细绳| 要删除脚本的ID。|

## 查询参数

| 范围| 数据类型| 描述| 
:--- | :--- | :---
| cluster_manager_timeout| 时间| 等待与群集管理器连接的时间。可选，默认为`30s`。|
| 暂停| 时间| 等待回应的时间。如果在超时值之前未收到响应，则将删除请求。

#### 示例请求

以下请求删除了`my-first-script` 脚本：

````json
DELETE _scripts/my-script
````
{% include copy-curl.html %}

#### 示例响应

这`DELETE _scripts/my-first-script` 请求返回以下字段：

````json
{
  "acknowledged" : true
}
````

要确定是否成功删除了存储的脚本，请使用[获取存储的脚本]({{site.url}}{{site.baseurl}}/api-reference/script-apis/get-stored-script/) API，将脚本名称传递给`script` 路径参数。

## 响应字段

<http方法> <Endpoint>请求返回以下响应字段：

| 场地| 数据类型| 描述| 
:--- | :--- | :---
| 承认| 布尔| 是否收到删除脚本请求。

