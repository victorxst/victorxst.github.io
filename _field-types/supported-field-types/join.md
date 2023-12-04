---
layout: default
title: 加入
nav_order: 44
has_children: false
parent: Object字段类型
grand_parent: 支持的字段类型
redirect_from:
  - /opensearch/supported-field-types/join/
  - /field-types/join/
---

# 加入现场类型

JOIN字段类型在同一索引中建立文档之间的父/子关系。

## 例子

创建映射以建立父母-产品与其品牌之间的儿童关系：

```json
PUT testindex1
{
  "mappings": {
    "properties": {
      "product_to_brand": { 
        "type": "join",
        "relations": {
          "brand": "product" 
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

然后，将父文档索引带有JOIN字段类型：

```json
PUT testindex1/_doc/1
{
  "name": "Brand 1",
  "product_to_brand": {
    "name": "brand" 
  }
}
```
{% include copy-curl.html %}

您还可以使用无物体表示法的快捷方式来索引父文档：

```json
PUT testindex1/_doc/1
{
  "name": "Brand 1",
  "product_to_brand" : "brand" 
}
```
{% include copy-curl.html %}

索引子文件时，您必须指定`routing` 查询参数是因为必须在同一碎片上索引父母和子女文档。每个孩子的文件都指其父母的ID`parent` 场地。

索引两个子文件，每个父母一个：

```json
PUT testindex1/_doc/3?routing=1
{
  "name": "Product 1",
  "product_to_brand": {
    "name": "product", 
    "parent": "1" 
  }
}
```
{% include copy-curl.html %}

```json
PUT testindex1/_doc/4?routing=1
{
  "name": "Product 2",
  "product_to_brand": {
    "name": "product", 
    "parent": "1" 
  }
}
```
{% include copy-curl.html %}

## 查询联接字段

当您查询加入字段时，响应包含指定返回文档是父母还是子女的子字段。对于子对象，还返回父ID。

### 搜索所有文档

```json
GET testindex1/_search
{
  "query": {
    "match_all": {}
  }
}
```
{% include copy-curl.html %}

响应表明文件是父母还是孩子：

```json
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "testindex1",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "Brand 1",
          "product_to_brand" : {
            "name" : "brand"
          }
        }
      },
      {
        "_index" : "testindex1",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 1.0,
        "_routing" : "1",
        "_source" : {
          "name" : "Product 1",
          "product_to_brand" : {
            "name" : "product",
            "parent" : "1"
          }
        }
      },
      {
        "_index" : "testindex1",
        "_type" : "_doc",
        "_id" : "4",
        "_score" : 1.0,
        "_routing" : "1",
        "_source" : {
          "name" : "Product 2",
          "product_to_brand" : {
            "name" : "product",
            "parent" : "1"
          }
        }
      }
    ]
  }
}
```

### 寻找父母的所有孩子

查找与品牌1相关的所有产品：

```json
GET testindex1/_search
{
  "query" : {
    "has_parent": {
      "parent_type":"brand",
      "query": {
        "match" : {
          "name": "Brand 1"
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

响应包含与品牌1相关的产品1和产品2：

```json
{
  "took" : 7,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "testindex1",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 1.0,
        "_routing" : "1",
        "_source" : {
          "name" : "Product 1",
          "product_to_brand" : {
            "name" : "product",
            "parent" : "1"
          }
        }
      },
      {
        "_index" : "testindex1",
        "_type" : "_doc",
        "_id" : "4",
        "_score" : 1.0,
        "_routing" : "1",
        "_source" : {
          "name" : "Product 2",
          "product_to_brand" : {
            "name" : "product",
            "parent" : "1"
          }
        }
      }
    ]
  }
}
```

### 寻找孩子的父母

找到产品1的父母：

```json
GET testindex1/_search
{
  "query" : {
    "has_child": {
      "type":"product",
      "query": {
        "match" : {
            "name": "Product 1"
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

响应将品牌1作为产品1的父母返回：

```json
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "testindex1",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "Brand 1",
          "product_to_brand" : {
            "name" : "brand"
          }
        }
      }
    ]
  }
}
```

## 父母有很多孩子

一位父母可以有很多孩子。与多个孩子创建映射：

```json
PUT testindex1
{
  "mappings": {
    "properties": {
      "parent_to_child": {
        "type": "join",
        "relations": {
          "parent": ["child 1", "child 2"]  
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

## 加入现场类型注释

- 索引中只能有一个联接字段映射。
- 在检索，更新或删除子文档时，您需要提供路由参数。这是因为必须在同一碎片上索引父母和子女文件。
- 不支持多个父母。
- 您只有在现有文档已经标记为父母时，才可以将子文档添加到现有文档中。
- 您可以将新的关系添加到现有的联接字段。

