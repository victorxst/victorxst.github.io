---
layout: default
title: 创建索引
parent: 索引API
nav_order: 25
redirect_from:
  - /opensearch/rest-api/index-apis/create-index/
  - /opensearch/rest-api/create-index/
---

# 创建索引
**引入1.0**
{: .label .label-purple }

虽然您可以通过使用文档作为基础来创建索引，但也可以创建一个空索引以供以后使用。

创建索引时，您可以指定其映射，设置和别名。

## 路径和HTTP方法

```
PUT <index-name>
```

## 索引命名限制

OpenSearch索引具有以下命名限制：

- 所有字母必须是小写。
- 索引名称不能以下划线开头（`_`）或连字符（`-`）。
- 索引名称不能包含空格，逗号或以下字符：

  `:`，`"`，`*`，`+`，`/`，`\`，`|`，`?`，`#`，`>`， 或者`<`

## 路径参数

| 范围| 描述|
:--- | :--- 
| 指数| 细绳| 索引名称。必须符合[索引命名限制](#index-naming-restrictions)。必需的。|

## 查询参数

您可以在请求中包含以下查询参数。所有参数都是可选的。

范围| 类型| 描述
:--- | :--- | :---
wait_for_active_shards| 细绳| 指定在OpenSearch处理请求之前必须可用的活动碎片数。默认值为1（仅是主要碎片）。设置`all` 或一个积极的整数。大于1的值需要复制品。例如，如果指定一个值为3的值，则该索引必须在两个其他节点上分布两个副本才能成功。
cluster_manager_timeout| 时间| 等待连接到群集管理器节点多长时间。默认为`30s`。
暂停| 时间| 等待请求返回多长时间。默认为`30s`。

## 请求身体

作为您请求的一部分，您可以选择指定[索引设置]({{site.url}}{{site.baseurl}}/im-plugin/index-settings/)，[映射]({{site.url}}{{site.baseurl}}/field-types/index/)， 和[别名]({{site.url}}{{site.baseurl}}/opensearch/index-alias/) 对于您新创建的索引。

#### 示例请求

```json
PUT /sample-index1
{
  "settings": {
    "index": {
      "number_of_shards": 2,
      "number_of_replicas": 1
    }
  },
  "mappings": {
    "properties": {
      "age": {
        "type": "integer"
      }
    }
  },
  "aliases": {
    "sample-alias1": {}
  }
}
```

