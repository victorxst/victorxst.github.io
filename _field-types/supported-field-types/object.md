---
layout: default
title: Object
nav_order: 41
has_children: false
parent: Object字段类型
grand_grand_parent: 支持的字段类型
redirect_from: 
  - /opensearch/supported-field-types/object/
  - /field-types/object/
---

# 对象字段类型

对象字段类型包含JSON对象（一组名称/值对）。JSON对象中的值可能是另一个JSON对象。没有必要指定`object` 作为映射对象字段时的类型，因为`object` 是默认类型。

## 例子

使用对象字段创建映射：

```json
PUT testindex1/_mappings
{
    "properties": {
      "patient": { 
        "properties" :
          {
            "name" : {
              "type" : "text"
            },
            "id" : {
              "type" : "keyword"
            }
          }   
      }
    }
}
```
{% include copy-curl.html %}

带有对象字段的文档索引：

```json
PUT testindex1/_doc/1
{ 
  "patient": { 
    "name" : "John Doe",
    "id" : "123456"
  } 
}
```
{% include copy-curl.html %}

嵌套对象在内部存储为平键/值对。要引用嵌套对象中的字段，请使用`parent field`。`child field` （例如，`patient.id`）。

搜索ID 123456的患者：

```json
GET testindex1/_search
{
  "query": {
    "term" : {
      "patient.id" : "123456"
    }
  }
}
```
{% include copy-curl.html %}

## 参数

下表列出了对象字段类型接受的参数。所有参数都是可选的。

范围| 描述
:--- | :--- 
[`dynamic`](#the-dynamic-parameter) | 指定是否可以将新字段动态添加到此对象中。有效值是`true`，`false`， 和`strict`。默认为`true`。
`enabled` | 布尔值指定是否应解析对象的JSON内容。如果`enabled` 被设定为`false`，该对象的内容不可索引或可搜索，但仍可以从_source字段中检索。默认为`true`。
`properties` | 该对象的字段，可以是任何受支持类型的字段。如果新属性可以动态添加到此对象中`dynamic` 被设定为`true`。

### 这`dynamic` 范围

这`dynamic` 参数指定是否可以将新字段动态添加到已经索引的对象中。

例如，您最初可以使用`patient` 只有一个字段的对象：

```json
PUT testindex1/_mappings
{
    "properties": {
      "patient": { 
        "properties" :
          {
            "name" : {
              "type" : "text"
            }
          }   
      }
    }
}
```
{% include copy-curl.html %}

然后，您将文档索引`id` 字段中`patient`：

```json
PUT testindex1/_doc/1
{ 
  "patient": { 
    "name" : "John Doe",
    "id" : "123456"
  } 
}
```
{% include copy-curl.html %}

结果，该领域`id` 被添加到映射中：

```json
{
  "testindex1" : {
    "mappings" : {
      "properties" : {        
        "patient" : {
          "properties" : {
            "id" : {
              "type" : "text",
              "fields" : {
                "keyword" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                }
              }
            },
            "name" : {
              "type" : "text"
            }
          }
        }
      }
    }
  }
}
```

这`dynamic` 参数具有以下有效值。

价值| 描述
:--- | :--- 
`true` | 新字段可以动态添加到映射中。这是默认值。
`false` | 新字段无法动态地添加到映射中。如果检测到新字段，则无法索引或可搜索。但是，它仍然可以从_source字段中检索。
`strict` | 当动态地将新字段添加到映射到映射时，会抛出一个异常。要将新字段添加到对象中，您必须先将其添加到映射中。

内在对象继承`dynamic` 除非他们自己声明自己的父母的参数价值`dynamic` 参数值。
{: .note}

