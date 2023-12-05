---
layout: default
title: 复杂的查询
parent: SQL
grand_parent: SQL和PPL
nav_order: 6
Redirect_from:
  - /search-plugins/sql/complex/
---

# 复杂的查询

除了简单的sfw（`SELECT-FROM-WHERE`）查询，SQL插件支持复杂的查询，例如子查询，加入，联合和减去。这些查询在多个OpenSearch索引上运行。要检查这些查询如何在幕后执行，请使用`explain` 手术。


## 加入

OpenSearch SQL支持内部连接，交叉连接并左外连接。

### 约束

加入有许多限制：

1. 您只能加入两个索引。
1. 您必须使用别名进行索引（例如，`people p`）。
1. 在ON子句中，您只能使用和条件。
1. 在某个陈述中，不要结合包含多个索引的树。例如，以下语句有效：

   ```
   WHERE (a.type1 > 3 OR a.type1 < 0) AND (b.type2 > 4 OR b.type2 < -1)
   ```

   以下声明没有：

   ```
   其中（a.type1> 3或b.type2 <0）和（a.type1> 4或b.type2 <-1）
   ```

1. You can't use GROUP BY or ORDER BY for results.
1. LIMIT with OFFSET (e.g. `限制25偏移25`) is not supported.

### Description

The `加入` clause combines columns from one or more indexes using values common to each.

### Syntax

Rule `Tablesource`:

![tableSource]({{site.url}}{{site.baseurl}}/images/tableSource.png)

Rule `加入PART`:

![joinPart]({{site.url}}{{site.baseurl}}/images/joinPart.png)

### Example 1: Inner join

Inner join creates a new result set by combining columns of two indexes based on your join predicates. It iterates the two indexes and compares each document to find the ones that satisfy the join predicates. You can optionally precede the `加入` clause with an `内` 关键词。

JOIN谓词由ON子句指定。

SQL查询：

```sql
SELECT
  a.account_number, a.firstname, a.lastname,
  e.id, e.name
FROM accounts a
JOIN employees_nested e
 ON a.account_number = e.id
```

解释：

这`explain` 输出很复杂，因为`JOIN` 子句与在单独的查询计划者框架中执行的两个OpenSearch DSL查询相关联。您可以通过检查`Physical Plan` 和`Logical Plan` 对象。

```json
{
  "Physical Plan" : {
    "Project [ columns=[a.account_number, a.firstname, a.lastname, e.name, e.id] ]" : {
      "Top [ count=200 ]" : {
        "BlockHashJoin[ conditions=( a.account_number = e.id ), type=JOIN, blockSize=[FixedBlockSize with size=10000] ]" : {
          "Scroll [ employees_nested as e, pageSize=10000 ]" : {
            "request" : {
              "size" : 200,
              "from" : 0,
              "_source" : {
                "excludes" : [ ],
                "includes" : [
                  "id",
                  "name"
                ]
              }
            }
          },
          "Scroll [ accounts as a, pageSize=10000 ]" : {
            "request" : {
              "size" : 200,
              "from" : 0,
              "_source" : {
                "excludes" : [ ],
                "includes" : [
                  "account_number",
                  "firstname",
                  "lastname"
                ]
              }
            }
          },
          "useTermsFilterOptimization" : false
        }
      }
    }
  },
  "description" : "Hash Join algorithm builds hash table based on result of first query, and then probes hash table to find matched rows for each row returned by second query",
  "Logical Plan" : {
    "Project [ columns=[a.account_number, a.firstname, a.lastname, e.name, e.id] ]" : {
      "Top [ count=200 ]" : {
        "Join [ conditions=( a.account_number = e.id ) type=JOIN ]" : {
          "Group" : [
            {
              "Project [ columns=[a.account_number, a.firstname, a.lastname] ]" : {
                "TableScan" : {
                  "tableAlias" : "a",
                  "tableName" : "accounts"
                }
              }
            },
            {
              "Project [ columns=[e.name, e.id] ]" : {
                "TableScan" : {
                  "tableAlias" : "e",
                  "tableName" : "employees_nested"
                }
              }
            }
          ]
        }
      }
    }
  }
}
```

结果集：

| A.ACCOUNT_NUMBER| A.FirstName| a.lastname| E.ID| E.NAME
：--- | ：--- | ：--- | ：--- | ：---
6| 哈蒂| 纽带| 6| 简·史密斯

### 示例2：交叉加入

Cross Join（也称为笛卡尔联接）将第一个索引中的每个文档与第二个文档结合在一起。
结果集是两个索引文档的笛卡尔产品。
此操作类似于没有内在的连接`ON` 指定联接条件的条款。

在两个大型甚至中等大小的索引上进行交叉连接是有风险的。它可能会触发断路器，该断路器终止查询以避免存储器耗尽。
{： 。警告 }

SQL查询：

```sql
SELECT
  a.account_number, a.firstname, a.lastname,
  e.id, e.name
FROM accounts a
JOIN employees_nested e
```

结果集：

| A.ACCOUNT_NUMBER| A.FirstName| a.lastname| E.ID| E.NAME
：--- | ：--- | ：--- | ：--- | ：---
1| 琥珀色| 公爵| 3| 鲍勃·史密斯
1| 琥珀色| 公爵| 4| 苏珊·史密斯
1| 琥珀色| 公爵| 6| 简·史密斯
6| 哈蒂| 纽带| 3| 鲍勃·史密斯
6| 哈蒂| 纽带| 4| 苏珊·史密斯
6| 哈蒂| 纽带| 6| 简·史密斯
13| Nanette| 贝茨| 3| 鲍勃·史密斯
13| Nanette| 贝茨| 4| 苏珊·史密斯
13| Nanette| 贝茨| 6| 简·史密斯
18| 戴尔| 亚当斯| 3| 鲍勃·史密斯
18| 戴尔| 亚当斯| 4| 苏珊·史密斯
18| 戴尔| 亚当斯| 6| 简·史密斯

### 示例3：左外连接

如果不满足联接谓词，则使用左外连接以将行从第一个索引保留。关键字`OUTER` 是可选的。

SQL查询：

```sql
SELECT
  a.account_number, a.firstname, a.lastname,
  e.id, e.name
FROM accounts a
LEFT JOIN employees_nested e
 ON a.account_number = e.id
```

结果集：

| A.ACCOUNT_NUMBER| A.FirstName| a.lastname| E.ID| E.NAME
：--- | ：--- | ：--- | ：--- | ：---
1| 琥珀色| 公爵| 无效的| 无效的
6| 哈蒂| 纽带| 6| 简·史密斯
13| Nanette| 贝茨| 无效的| 无效的
18| 戴尔| 亚当斯| 无效的| 无效的

## 子查询

子查询是完整的`SELECT` 在另一个语句中使用的语句，并包含在括号中。
从解释输出中，您可以看到一些子查询实际上被转换为等效的加入查询以执行。

### 示例1：表子查询

SQL查询：

```sql
SELECT a1.firstname, a1.lastname, a1.balance
FROM accounts a1
WHERE a1.account_number IN (
  SELECT a2.account_number
  FROM accounts a2
  WHERE a2.balance > 10000
)
```

解释：

```json
{
  "Physical Plan" : {
    "Project [ columns=[a1.balance, a1.firstname, a1.lastname] ]" : {
      "Top [ count=200 ]" : {
        "BlockHashJoin[ conditions=( a1.account_number = a2.account_number ), type=JOIN, blockSize=[FixedBlockSize with size=10000] ]" : {
          "Scroll [ accounts as a2, pageSize=10000 ]" : {
            "request" : {
              "size" : 200,
              "query" : {
                "bool" : {
                  "filter" : [
                    {
                      "bool" : {
                        "adjust_pure_negative" : true,
                        "must" : [
                          {
                            "bool" : {
                              "adjust_pure_negative" : true,
                              "must" : [
                                {
                                  "bool" : {
                                    "adjust_pure_negative" : true,
                                    "must_not" : [
                                      {
                                        "bool" : {
                                          "adjust_pure_negative" : true,
                                          "must_not" : [
                                            {
                                              "exists" : {
                                                "field" : "account_number",
                                                "boost" : 1
                                              }
                                            }
                                          ],
                                          "boost" : 1
                                        }
                                      }
                                    ],
                                    "boost" : 1
                                  }
                                },
                                {
                                  "range" : {
                                    "balance" : {
                                      "include_lower" : false,
                                      "include_upper" : true,
                                      "from" : 10000,
                                      "boost" : 1,
                                      "to" : null
                                    }
                                  }
                                }
                              ],
                              "boost" : 1
                            }
                          }
                        ],
                        "boost" : 1
                      }
                    }
                  ],
                  "adjust_pure_negative" : true,
                  "boost" : 1
                }
              },
              "from" : 0
            }
          },
          "Scroll [ accounts as a1, pageSize=10000 ]" : {
            "request" : {
              "size" : 200,
              "from" : 0,
              "_source" : {
                "excludes" : [ ],
                "includes" : [
                  "firstname",
                  "lastname",
                  "balance",
                  "account_number"
                ]
              }
            }
          },
          "useTermsFilterOptimization" : false
        }
      }
    }
  },
  "description" : "Hash Join algorithm builds hash table based on result of first query, and then probes hash table to find matched rows for each row returned by second query",
  "Logical Plan" : {
    "Project [ columns=[a1.balance, a1.firstname, a1.lastname] ]" : {
      "Top [ count=200 ]" : {
        "Join [ conditions=( a1.account_number = a2.account_number ) type=JOIN ]" : {
          "Group" : [
            {
              "Project [ columns=[a1.balance, a1.firstname, a1.lastname, a1.account_number] ]" : {
                "TableScan" : {
                  "tableAlias" : "a1",
                  "tableName" : "accounts"
                }
              }
            },
            {
              "Project [ columns=[a2.account_number] ]" : {
                "Filter [ conditions=[AND ( AND account_number ISN null, AND balance GT 10000 ) ] ]" : {
                  "TableScan" : {
                    "tableAlias" : "a2",
                    "tableName" : "accounts"
                  }
                }
              }
            }
          ]
        }
      }
    }
  }
}
```

结果集：

| a1.firstname| a1.lastname| a1.平衡
：--- | ：--- | ：---
琥珀色| 公爵| 39225
Nanette| 贝茨| 32838

### 示例2：来自子查询

SQL查询：

```sql
SELECT a.f, a.l, a.a
FROM (
  SELECT firstname AS f, lastname AS l, age AS a
  FROM accounts
  WHERE age > 30
) AS a
```

解释：

```json
{
  "from" : 0,
  "size" : 200,
  "query" : {
    "bool" : {
      "filter" : [
        {
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
        }
      ],
      "adjust_pure_negative" : true,
      "boost" : 1.0
    }
  },
  "_source" : {
    "includes" : [
      "firstname",
      "lastname",
      "age"
    ],
    "excludes" : [ ]
  }
}
```

结果集：

| F| l| A
：--- | ：--- | ：---
琥珀色| 公爵| 32
戴尔| 亚当斯| 33
哈蒂| 纽带| 36

