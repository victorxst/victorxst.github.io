---
layout: default
title: 删除
parent: SQL
grand_parent: SQL和PPL
nav_order: 12
Redirect_from:
  - /search-plugins/sql/delete/
---


# 删除

这`DELETE` 声明删除满足谓词的文档`WHERE` 条款。
如果您不指定`WHERE` 条款，所有文档均已删除。

### 环境

这`DELETE` 语句默认情况下是禁用的。启用`DELETE` SQL中的功能，您需要通过发送以下请求来更新配置：

```json
PUT _plugins/_query/settings
{
  "transient": {
    "plugins.sql.delete.enabled": "true"
  }
}
```

### 句法

规则`singleDeleteStatement`：

![单骨架]({{site.url}}{{site.baseurl}}/images/singleDeleteStatement.png)

### 例子

SQL查询：

```sql
DELETE FROM accounts
WHERE age > 30
```

解释：

```json
{
  "size" : 1000,
  "query" : {
    "bool" : {
      "must" : [
        {
          "range" : {
            "age" : {
              "from" : 30,
              "to" : null,
              "include_lower" : false,
              "include_upper" : true,
              "boost" : 1.0
            }
          }
        }
      ],
      "adjust_pure_negative" : true,
      "boost" : 1.0
    }
  },
  "_source" : false
}
```

结果集：

```json
{
  "schema" : [
    {
      "name" : "deleted_rows",
      "type" : "long"
    }
  ],
  "total" : 1,
  "datarows" : [
    [
      3
    ]
  ],
  "size" : 1,
  "status" : 200
}
```

这`datarows` 字段显示已删除的文档数量。

