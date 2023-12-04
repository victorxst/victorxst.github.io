---
layout: default
title: 创建或更新映射
parent: 索引API
nav_order: 27
redirect_from:
  - /opensearch/rest-api/index-apis/update-mapping/
  - /opensearch/rest-api/update-mapping/
---

# 创建或更新映射
**引入1.0**
{: .label .label-purple }

如果要在索引中创建或添加映射和字段，则可以使用put映射API操作。对于现有的映射，此操作更新映射。

您无法使用此操作来更新已经映射到索引中现有数据的映射。您必须首先使用所需映射创建一个新索引，然后使用[Reindex API操作]({{site.url}}{{site.baseurl}}/opensearch/reindex-data) 将所有文档从旧索引映射到新索引。如果您不想停机时间-索引您的索引，您可以使用[别名]({{site.url}}{{site.baseurl}}/opensearch/index-alias)。


## 必需的路径参数

唯一必需的路径参数是与映射关联的索引。如果您没有指定索引，则会遇到错误。您可以指定单个索引，或以下逗号分隔的多个索引：

```
PUT /<target-index>/_mapping
PUT /<target-index1>,<target-index2>/_mapping
```

## 要求的请求身体领域

请求主体必须包含`properties`，它具有要创建或更新的所有映射。

```json
{
  "properties":{
    "color":{
      "type": "text"
    },
    "year":{
      "type": "integer"
    }
  }
}
```

## 可选请求身体场

### 动态的

您可以通过设置文档结构与索引映射的结构匹配`dynamic` 请求身体领域`strict`，如以下示例所示：

```json
{
  "dynamic": "strict",
  "properties":{
    "color":{
      "type": "text"
    }
  }
}
```

## 可选查询参数

可选地，您可以添加查询参数以提出更具体的请求。例如，要跳过响应中的所有缺失或封闭索引，您可以添加`ignore_unavailable` 查询您的请求参数如下：

```json
PUT /sample-index/_mapping?ignore_unavailable
```

下表定义了put映射查询参数：

范围| 数据类型| 描述
:--- | :--- | :---
允许_no_indices| 布尔| 是否忽略不符合任何索引的通配符。默认为`true`。
Expand_WildCard| 细绳| 将通配符表达式扩展到不同的索引。将多个值与逗号相结合。可用值是`all` （匹配所有索引），`open` （匹配打开索引），`closed` （匹配封闭索引），`hidden` （匹配隐藏索引），`none` （请勿接受通配符表达式），必须与`open`，`closed`， 或两者。默认为`open`。
ignore_unavailable| 布尔| 如果为true，则OpenSearch不会在响应中包括缺失或封闭索引。
ignore_malformed| 布尔| 将此参数与`ip_range` 数据类型要指定OpenSearch应该忽略错误的字段。如果`true`，OpenSearch不包括与响应中索引中指定的IP范围不匹配的条目。默认值为`false`。
cluster_manager_timeout| 时间| 等待连接到群集管理器节点多长时间。默认为`30s`。
暂停| 时间| 等待响应返回多长时间。默认为`30s`。
write_index_only| 布尔| OpenSearch是否应仅将映射更新应用于写索引。

#### 样本请求

以下请求为`sample-index` 指数：

```json
PUT /sample-index/_mapping

{
  "properties": {
    "age": {
      "type": "integer"
    },
    "occupation":{
      "type": "text"
    }
  }
}
```
{% include copy-curl.html %}

#### 样本响应

成功后，响应返回`"acknowledged": true`。

```json
{
    "acknowledged": true
}
```



