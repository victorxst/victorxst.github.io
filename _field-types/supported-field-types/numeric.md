---
layout: default
title: Numeric字段类型
grand_parent: 支持的字段类型
nav_order: 15
has_children: true
redirect_from:
  - /opensearch/supported-field-types/numeric/
  - /field-types/numeric/
---

# Numeric字段类型

下表列出了OpenSearch支持的所有数字字段类型。

字段数据类型| 描述
：--- | ：--- 
`byte` | 一个签名8-位整数。最低限度为＆减少128。最大值为127。
`double` | 双重-精度64-位IEEE 754浮动-点值。最小幅度为2 <sup>＆sinus; 1074 </sup>。最大幅度为（2＆minus; 2 <sup>＆minus; 52 </sup>）＆middot;2 <sup> 1023 </sup>。显着位的数量为53。显着数字的数量为15.95。
`float` | 一个-精度32-位IEEE 754浮动-点值。最小幅度为2 <sup>＆sinus; 149 </sup>。最大幅度为（2＆minus; 2 <sup>＆sinus; 23 </sup>）＆middot;2 <sup> 127 </sup>。显着位的数量为24。显着数字的数量为7.22。
`half_float` | 一半-精度16-位IEEE 754浮动-点值。最小幅度为2 <sup>＆sinus; 24 </sup>。最大幅度为65504。显着位的数量为11。显着数字的数量为3.31。
`integer` | 签名的32-位整数。最小值IS＆MINUS; 2 <sup> 31 </sup>。最大值为2 <sup> 31 </sup>&minus;;1。
`long` | 签名64-位整数。最小值IS＆MINUS; 2 <sup> 63 </sup>。最大值为2 <sup> 63 </sup>&minus;;1。
[`unsigned_long`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/unsigned-long/) | 未签名的64-位整数。最小值为0。最大为2 <sup> 64 </sup>&minus;；1。
`short` | 签名16-位整数。最小值IS＆MANUS; 2 <sup> 15 </sup>。最大值为2 <sup> 15 </sup>＆minus;1。
[`scaled_float`](#scaled-float-field-type) | 浮动-点值乘以双重尺度因子并将其存储为长值。

整数，长，浮动和双场类型都有相应的[范围字段类型]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/range/)。
{: .note}

如果您的数字字段包含标识符（例如ID），则可以将此字段映射为[关键词]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/keyword/) 优化更快的期限-级查询。如果您需要在此字段上使用范围查询，则除关键字字段类型外，还可以将此字段映射为数字字段类型。
{: .tip }

## 例子

创建一个映射，其中integer_value是整数字段：

```json
PUT testindex 
{
  "mappings" : {
    "properties" :  {
      "integer_value" : {
        "type" : "integer"
      }
    }
  }
}
```
{% include copy-curl.html %}

索引具有整数值的文档：

```json
PUT testindex/_doc/1 
{
  "integer_value" : 123
}
```
{% include copy-curl.html %}

## 缩放的浮点场类型

缩放的浮点场类型是浮动-点值乘以比例因子并将其存储为长值。它采用数字字段类型所采用的所有可选参数，以及附加的scaling_factor参数。创建缩放的浮子时需要比例因子。

缩放的浮子对于节省磁盘空间很有用。较大的Scaleing_Factor值可提高准确性，但开销更高。
{: .note}

## 缩放的浮动示例

创建一个映射`scaled` 是一个scaled_float字段：

```json
PUT testindex 
{
  "mappings" : {
    "properties" :  {
      "scaled" : {
        "type" : "scaled_float",
        "scaling_factor" : 10
      }
    }
  }
}
```
{% include copy-curl.html %}

索引一个具有scaled_float值的文档：

```json
PUT testindex/_doc/1 
{
  "scaled" : 2.3
}
```
{% include copy-curl.html %}

这`scaled` 值将存储为23。

## 参数

下表列出了数字字段类型接受的参数。所有参数都是可选的。

范围| 描述
：--- | ：--- 
`boost` | 浮动-指定该字段对相关性分数的重量的点值。值高于1.0的值增加了该领域的相关性。0.0至1.0之间的值降低了该场的相关性。默认值为1.0。
`coerce` | 一个布尔值，该值向整数值截断了小数，并将字符串转换为数字值。默认为`true`。
`doc_values` | 布尔值指定是否应将字段存储在磁盘上，以便将其用于聚合，排序或脚本。默认为`true`。
`ignore_malformed` | 布尔值指定忽略畸形值而不引发异常的值。默认为`false`。
`index` | 布尔值指定是否应搜索该字段。默认为`true`。
`meta` | 接受该领域的元数据。
[`null_value`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/index#null-value) | 用于代替的值`null`。必须与字段相同。如果未指定此参数，则该字段在其值为时被视为丢失`null`。默认为`null`。
`store` | 布尔值指定是否应存储字段值，并且可以与_source字段分开检索。默认为`false`。

缩放浮子具有附加的必需参数：`scaling_factor`。

范围| 描述
：--- | ：--- 
`scaling_factor` | 双重值乘以场值并舍入到最近的长度。必需的。

