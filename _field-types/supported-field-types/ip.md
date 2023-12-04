---
layout: default
title: IP地址
nav_order: 30
has_children: false
parent: 支持的字段类型
redirect_from:
  - /opensearch/supported-field-types/ip/
  - /field-types/ip/
---

# IP地址字段类型

IP字段类型包含IPv4或IPv6格式中的IP地址。

为了表示IP地址范围，有一个IP[范围字段类型]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/range/)。
{: .note}

## 例子

使用IP地址创建映射：

```json
PUT testindex 
{
  "mappings" : {
    "properties" :  {
      "ip_address" : {
        "type" : "ip"
      }
    }
  }
}
```
{% include copy-curl.html %}

带有IP地址的文档索引：

```json
PUT testindex/_doc/1 
{
  "ip_address" : "10.24.34.0"
}
```
{% include copy-curl.html %}

查询特定IP地址的索引：

```json
GET testindex/_doc/1 
{
  "query": {
    "term": {
      "ip_address": "10.24.34.0"
    }
  }
}
```
{% include copy-curl.html %}

## 搜索IP地址及其关联的网络掩码

您可以在[无阶级的中间-域路由（CIDR）符号](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing#CIDR_notation)。使用CIDR表示法，指定IP地址和前缀长度（0-32），由`/`。例如，前缀长度为24，将与所有IP地址与相同的初始24位匹配。

#### IPv4格式的示例查询

```json
GET testindex/_search 
{
  "query": {
    "term": {
      "ip_address": "10.24.34.0/24"
    }
  }
}
```
{% include copy-curl.html %}

#### IPv6格式的示例查询

```json
GET testindex/_search 
{
  "query": {
    "term": {
      "ip_address": "2001:DB8::/24"
    }
  }
}
```
{% include copy-curl.html %}

如果您在ipv6格式中使用IP地址`query_string` 查询，您需要逃脱`:` 字符是因为它们被解析为特殊字符。您可以通过将IP地址包装在引号标记中并逃脱这些引号，从而实现这一目标`\`。

```json
GET testindex/_search 
{
  "query" : {
    "query_string": {
      "query": "ip_address:\"2001:DB8::/24\""
    }
  }
}
```
{% include copy-curl.html %}

## 参数

下表列出了IP字段类型接受的参数。所有参数都是可选的。

范围| 描述
:--- | :--- 
`boost` | 浮动-指定该字段对相关性分数的重量的点值。值高于1.0的值增加了该领域的相关性。0.0至1.0之间的值降低了该场的相关性。默认值为1.0。
`doc_values` | 布尔值指定是否应将字段存储在磁盘上，以便将其用于聚合，排序或脚本。默认为`true`。
`ignore_malformed` | 布尔值指定忽略畸形值而不引发异常的值。默认为`false`。
`index` | 布尔值指定是否应搜索该字段。默认为`true`。
[`null_value`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/index#null-value) | 用于代替的值`null`。必须与字段相同。如果未指定此参数，则该字段在其值为时被视为丢失`null`。默认为`null`。
`store` | 布尔值指定是否应存储字段值，并且可以与_source字段分开检索。默认为`false`。



