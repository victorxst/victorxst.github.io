---
layout: default
title: JSON支持
parent: SQL
grand_parent: SQL和PPL
nav_order: 8
redirect_from:
  - /search-plugins/sql/partiql/
---

# JSON支持

SQL插件通过关注来支持JSON[partiql](https://partiql.org/) 规格，SQL-兼容查询语言，可让您查询半度-任何数据格式的结构和嵌套数据。SQL插件仅支持PartiQL规范的子集。

## 查询嵌套集合

PartiQl扩展了SQL，使您可以查询和非嵌套的收集。在OpenSearch中，这对于用嵌套对象或字段查询JSON索引非常有用。

要跟随，请使用`bulk` 索引一些示例数据的操作：

```json
POST employees_nested/_bulk?refresh
{"index":{"_id":"1"}}
{"id":3,"name":"Bob Smith","title":null,"projects":[{"name":"SQL Spectrum querying","started_year":1990},{"name":"SQL security","started_year":1999},{"name":"OpenSearch security","started_year":2015}]}
{"index":{"_id":"2"}}
{"id":4,"name":"Susan Smith","title":"Dev Mgr","projects":[]}
{"index":{"_id":"3"}}
{"id":6,"name":"Jane Smith","title":"Software Eng 2","projects":[{"name":"SQL security","started_year":1998},{"name":"Hello security","started_year":2015,"address":[{"city":"Dallas","state":"TX"}]}]}
```

### 示例1：取消嵌套集合

此示例找到嵌套的文档（`projects`）具有场值（`name`）满足谓词（包含`security`）。由于每个父文档都可以具有多个嵌套文档，因此匹配的嵌套文档被扁平。换句话说，最终结果是父母和嵌套文档之间的笛卡尔产品。

```sql
SELECT e.name AS employeeName,
       p.name AS projectName
FROM employees_nested AS e,
       e.projects AS p
WHERE p.name LIKE '%security%'
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
                "nested" : {
                  "query" : {
                    "wildcard" : {
                      "projects.name" : {
                        "wildcard" : "*security*",
                        "boost" : 1.0
                      }
                    }
                  },
                  "path" : "projects",
                  "ignore_unmapped" : false,
                  "score_mode" : "none",
                  "boost" : 1.0,
                  "inner_hits" : {
                    "ignore_unmapped" : false,
                    "from" : 0,
                    "size" : 3,
                    "version" : false,
                    "seq_no_primary_term" : false,
                    "explain" : false,
                    "track_scores" : false,
                    "_source" : {
                      "includes" : [
                        "projects.name"
                      ],
                      "excludes" : [ ]
                    }
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
      "name"
    ],
    "excludes" : [ ]
  }
}
```

结果集：

| 员工姓名| 项目名
：--- | ：---
鲍勃·史密斯| OpenSearch安全
鲍勃·史密斯| SQL安全性
简·史密斯| 您好安全
简·史密斯| SQL安全性

### 示例2：在存在的子查询中不努力

要在子查询中解开嵌套集合以检查它是否满足条件：

```sql
SELECT e.name AS employeeName
FROM employees_nested AS e
WHERE EXISTS (
    SELECT *
    FROM e.projects AS p
    WHERE p.name LIKE '%security%'
)
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
                "nested" : {
                  "query" : {
                    "bool" : {
                      "must" : [
                        {
                          "bool" : {
                            "must" : [
                              {
                                "bool" : {
                                  "must_not" : [
                                    {
                                      "bool" : {
                                        "must_not" : [
                                          {
                                            "exists" : {
                                              "field" : "projects",
                                              "boost" : 1.0
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
                              {
                                "wildcard" : {
                                  "projects.name" : {
                                    "wildcard" : "*security*",
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
                  "path" : "projects",
                  "ignore_unmapped" : false,
                  "score_mode" : "none",
                  "boost" : 1.0
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
      "name"
    ],
    "excludes" : [ ]
  }
}
```

结果集：

| 员工姓名|
：--- | ：---
鲍勃·史密斯|
简·史密斯|

