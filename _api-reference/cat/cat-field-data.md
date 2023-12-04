---
layout: default
title: CAT 字段数据
parent: CAT API
nav_order: 15
has_children: false
redirect_from:
- /opensearch/rest-api/cat/cat-field-data/
---

# Cat FieldData
**引入1.0**
{: .label .label-purple }

CAT FieldData操作列出了每个节点每个字段使用的内存大小。

## 例子

```json
GET _cat/fielddata?v
```
{% include copy-curl.html %}

要将信息限制到特定字段，请在查询之后添加字段名称：

```json
GET _cat/fielddata/<field_name>?v
```
{% include copy-curl.html %}

如果您想获取多个字段的信息，请将字段名称与逗号分开：

```json
GET _cat/fielddata/field_name_1,field_name_2,field_name_3
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET _cat/fielddata?v
GET _cat/fielddata/<field_name>?v
```

## URL参数

所有CAT FieldData URL参数都是可选的。

除了[常见的URL参数]({{site.url}}{{site.baseurl}}/api-reference/cat/index)，您可以指定以下参数：

范围| 类型| 描述
:--- | :--- | :---
字节| 字节大小| 指定字节大小的单元。例如，`7kb` 或者`6gb`。有关更多信息，请参阅[支持单位]({{site.url}}{{site.baseurl}}/opensearch/units/)。

## 回复

以下响应显示所有字段的内存大小为284个字节：

```json
id                     host       ip         node       field size
1vo54NuxSxOrbPEYdkSF0w 172.18.0.4 172.18.0.4 odfe-node1 _id   284b
ZaIkkUd4TEiAihqJGkp5CA 172.18.0.3 172.18.0.3 odfe-node2 _id   284b
```

