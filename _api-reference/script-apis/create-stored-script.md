---
layout: default
title: 创建或更新存储的脚本
parent: 脚本API
nav_order: 1
---

# 创建或更新存储的脚本
**引入1.0**
{: .label .label-purple }

创建或更新存储的脚本或搜索模板。

有关无痛脚本的其他信息，请参见：

*[k-nn无痛的脚本扩展]({{site.url}}{{site.baseurl}}/search-plugins/knn/painless-functions/)。

*[k-nn]({{site.url}}{{site.baseurl}}/search-plugins/knn/index/)。


## 路径参数

| 范围| 数据类型| 描述| 
:--- | :--- | :---
| 脚本-ID| 细绳| 存储的脚本或搜索模板ID。必须在整个集群中是唯一的。必需的。|

## 查询参数

所有参数都是可选的。

| 范围| 数据类型| 描述| 
:--- | :--- | :---
| 语境| 细绳| 脚本或搜索模板要运行的上下文。为了防止错误，API立即在此上下文中编译脚本或模板。|
| cluster_manager_timeout| 时间| 等待与群集管理器连接的时间。默认为30秒。|
| 暂停| 时间| 等待回应的时间。如果在超时值之前未收到响应，则请求失败并返回错误。默认为30秒。|

## 请求字段

| 场地| 数据类型| 描述| 
:--- | :--- | :---
| 脚本| 目的| 定义脚本或搜索模板，其参数及其语言。请参阅下面的 *脚本对象 *。|

*脚本对象*

| 场地| 数据类型| 描述| 
:--- | :--- | :---
| 朗| 细绳| 脚本语言。必需的。|
| 来源| 字符串或对象| 必需的。<br /> <br />对于脚本，一个带有脚本内容的字符串。<br /> <br />对于搜索模板，定义搜索模板的对象。支持与[搜索]({{site.url}}{{site.baseurl}}/api-reference/search) API请求主体。搜索模板还支持胡须变量。|

#### 示例请求

该示例使用索引称为`books` 使用以下文件：

````json
{"index":{"_id":1}}
{"name":"book1","author":"Faustine","ratings":[4,3,5]}
{"index":{"_id":2}}
{"name":"book2","author":"Amit","ratings":[5,5,5]}
{"index":{"_id":3}}
{"name":"book3","author":"Gilroy","ratings":[2,1,5]}
````

以下请求创建了无痛的脚本`my-first-script`。它总和每本书的评分并在输出中显示总和。

````json
PUT _scripts/my-first-script
{
  "script": {
      "lang": "painless",
      "source": """
          int total = 0;
          for (int i = 0; i < doc['ratings'].length; ++i) {
            total += doc['ratings'][i];
          }
          return total;
        """
  }
}
````
{% include copy.html %}

上面的示例使用OpenSearch仪表板中的Dev工具控制台的语法。您也可以使用卷曲请求。
{: .note }

以下curl请求等效于以前的仪表板控制台示例：

````json
curl -XPUT "http://opensearch:9200/_scripts/my-first-script" -H 'Content-Type: application/json' -d'
{
  "script": {
      "lang": "painless",
      "source": "\n          int total = 0;\n          for (int i = 0; i < doc['\''ratings'\''].length; ++i) {\n            total += doc['\''ratings'\''][i];\n          }\n          return total;\n        "
  }
}'
````
{% include copy.html %}


以下请求创建了无痛的脚本`my-first-script`，总和每本书的评分并在输出中显示总和：

````json
PUT _scripts/my-first-script
{
  "script": {
      "lang": "painless",
      "source": """
          int total = 0;
          for (int i = 0; i < doc['ratings'].length; ++i) {
            total += doc['ratings'][i];
          }
          return total;
        """
  }
}
````
{% include copy-curl.html %}

看[执行无痛的存储脚本]({{site.url}}{{site.baseurl}}/api-reference/script-apis/exec-stored-script/) 有关运行脚本的信息。

#### 示例响应

这`PUT _scripts/my-first-script` 请求返回以下字段：

````json
{
  "acknowledged" : true
}
````

要确定是否成功创建了脚本，请使用[获取存储的脚本]({{site.url}}{{site.baseurl}}/api-reference/script-apis/get-stored-script/) API，将脚本名称传递给`script` 路径参数。
{: .note}

### 响应字段

| 场地| 数据类型| 描述| 
:--- | :--- | :---
| 承认| 布尔| 是否收到请求。|

## 使用参数创建或更新存储的脚本

无痛的脚本支持`params` 将变量传递到脚本。

#### 例子

以下请求创建了无痛的脚本`multiplier-script`。该请求总计每本书的评分，将总价值乘以`multiplier` 参数，并在输出中显示结果：

````json
PUT _scripts/multiplier-script
{
  "script": {
      "lang": "painless",
      "source": """
          int total = 0;
          for (int i = 0; i < doc['ratings'].length; ++i) {
            total += doc['ratings'][i];
          }
          return total * params['multiplier'];
        """
  }
}
````
{% include copy-curl.html %}

### 示例响应

这`PUT _scripts/multiplier-script` 请求返回以下字段：

````json
{
  "acknowledged" : true
}
````
