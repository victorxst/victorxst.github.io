---
layout: default
title: 二进制
parent: 支持的字段类型
nav_order: 12
has_children: false
redirect_from:
  - /opensearch/supported-field-types/binary/
  - /field-types/binary/
---

# 二进制字段类型

二进制字段类型包含一个二进制值[基础64](https://en.wikipedia.org/wiki/Base64) 编码无法搜索的编码。

## 例子

用二进制字段创建映射：

```json
PUT testindex 
{
  "mappings" : {
    "properties" :  {
      "binary_value" : {
        "type" : "binary"
      }
    }
  }
}
```
{% include copy-curl.html %}

索引具有二进制值的文档：

```json
PUT testindex/_doc/1 
{
  "binary_value" : "bGlkaHQtd29rfx4="
}
```
{% include copy-curl.html %}

使用`=` 作为填充字符。不允许嵌入式新线字符。
{: .note}

## 参数

下表列出了二进制字段类型接受的参数。所有参数都是可选的。

范围| 描述
:--- | :--- 
`doc_values` | 布尔值指定是否应将字段存储在磁盘上，以便将其用于聚合，排序或脚本。选修的。默认为`true`。
`store` | 布尔值指定是否应存储字段值，并且可以与_source字段分开检索。选修的。默认为`false`

