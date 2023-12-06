---
layout: default
title: 别名
nav_order: 10
has_children: false
parent: 支持的字段类型
redirect_from:
  - /opensearch/supported-field-types/alias/
  - /field-types/alias/
---

# 别名字段类型

别名字段类型为现有字段创建另一个名称。您可以在[搜索](#using-aliases-in-search-api-operations) 和[现场功能](#using-aliases-in-field-capabilities-api-operations) API操作，有一些[例外](#exceptions)。设置一个[别名](#alias-field)，您需要指定[原始领域](#original-field) 名称`path` 范围。

## 例子

```json
PUT movies 
{
  "mappings" : {
    "properties" : {
      "year" : {
        "type" : "date"
      },
      "release_date" : {
        "type" : "alias",
        "path" : "year"
      }
    }
  }
}
```
{% include copy-curl.html %}

## 参数

范围| 描述
:--- | :--- 
`path` | 原始字段的完整路径，包括所有父对象。例如，parent.child.field_name。必需的。

## 别名字段

别名字段必须遵守以下规则：

- 别名字段只能具有一个原始字段。
- 在嵌套对象中，别名必须具有与原始场相同的嵌套级别。

要更改别名引用的字段，请更新映射。请注意，任何先前存储的Percolator查询中的别名仍将引用原始字段。
{:.note}

## 原始领域

别名必须遵守以下规则：
- 必须在创建别名之前创建原始字段。
- 原始字段不能是对象或其他别名。

## 在搜索API操作中使用别名

您可以在搜索API的以下读取操作中使用别名：
- 查询
- 等
- 聚合
- `stored_fields`
- `docvalue_fields`
- 建议
- 强调
- 访问字段值的脚本

## 在现场功能API操作中使用别名

要在字段功能API中使用别名，请在字段参数中指定它。

```json
GET movies/_field_caps?fields=release_date
```
{% include copy-curl.html %}

## 例外

在以下情况下，您不能使用别名：
- 在写请求中，例如更新请求。
- 并发-字段或目标`copy_to`。
- 作为用于过滤结果的_source参数。
- 在具有字段名称的API中，例如术语向量。
- 在`terms`，`more_like_this`， 和`geo_shape` 查询（检索文档时不支持别名）。

## 通配符

在搜索和现场功能通配符查询中，原始场和别名都与通配符模式相匹配。

```json
GET movies/_field_caps?fields=release*
```
{% include copy-curl.html %}

