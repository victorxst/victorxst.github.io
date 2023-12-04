---
layout: default
title: 获取存储的脚本上下文
parent: 脚本API
nav_order: 5
---

# 获取存储的脚本上下文
**引入1.0**
{: .label .label-purple }

检索存储脚本的所有上下文。

#### 示例请求

````json
GET _script_context
````
{% include copy-curl.html %}

#### 示例响应

这`GET _script_context` 请求返回以下字段：

````json
{
  "contexts" : [
    {
      "name" : "aggregation_selector",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "boolean",
          "params" : [ ]
        },
        {
          "name" : "getParams",
          "return_type" : "java.util.Map",
          "params" : [ ]
        }
      ]
    },
    {
      "name" : "aggs",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "java.lang.Object",
          "params" : [ ]
        },
        {
          "name" : "getDoc",
          "return_type" : "java.util.Map",
          "params" : [ ]
        },
        {
          "name" : "getParams",
          "return_type" : "java.util.Map",
          "params" : [ ]
        },
        {
          "name" : "get_score",
          "return_type" : "java.lang.Number",
          "params" : [ ]
        },
        {
          "name" : "get_value",
          "return_type" : "java.lang.Object",
          "params" : [ ]
        }
      ]
    },
    {
      "name" : "aggs_combine",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "java.lang.Object",
          "params" : [ ]
        },
        {
          "name" : "getParams",
          "return_type" : "java.util.Map",
          "params" : [ ]
        },
        {
          "name" : "getState",
          "return_type" : "java.util.Map",
          "params" : [ ]
        }
      ]
    },
    {
      "name" : "aggs_init",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "void",
          "params" : [ ]
        },
        {
          "name" : "getParams",
          "return_type" : "java.util.Map",
          "params" : [ ]
        },
        {
          "name" : "getState",
          "return_type" : "java.lang.Object",
          "params" : [ ]
        }
      ]
    },
    {
      "name" : "aggs_map",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "void",
          "params" : [ ]
        },
        {
          "name" : "getDoc",
          "return_type" : "java.util.Map",
          "params" : [ ]
        },
        {
          "name" : "getParams",
          "return_type" : "java.util.Map",
          "params" : [ ]
        },
        {
          "name" : "getState",
          "return_type" : "java.util.Map",
          "params" : [ ]
        },
        {
          "name" : "get_score",
          "return_type" : "double",
          "params" : [ ]
        }
      ]
    },
    {
      "name" : "aggs_reduce",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "java.lang.Object",
          "params" : [ ]
        },
        {
          "name" : "getParams",
          "return_type" : "java.util.Map",
          "params" : [ ]
        },
        {
          "name" : "getStates",
          "return_type" : "java.util.List",
          "params" : [ ]
        }
      ]
    },
    {
      "name" : "analysis",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "boolean",
          "params" : [
            {
              "type" : "org.opensearch.analysis.common.AnalysisPredicateScript$Token",
              "name" : "token"
            }
          ]
        }
      ]
    },
    {
      "name" : "bucket_aggregation",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "java.lang.Number",
          "params" : [ ]
        },
        {
          "name" : "getParams",
          "return_type" : "java.util.Map",
          "params" : [ ]
        }
      ]
    },
    {
      "name" : "field",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "java.lang.Object",
          "params" : [ ]
        },
        {
          "name" : "getDoc",
          "return_type" : "java.util.Map",
          "params" : [ ]
        },
        {
          "name" : "getParams",
          "return_type" : "java.util.Map",
          "params" : [ ]
        }
      ]
    },
    {
      "name" : "filter",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "boolean",
          "params" : [ ]
        },
        {
          "name" : "getDoc",
          "return_type" : "java.util.Map",
          "params" : [ ]
        },
        {
          "name" : "getParams",
          "return_type" : "java.util.Map",
          "params" : [ ]
        }
      ]
    },
    {
      "name" : "ingest",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "void",
          "params" : [
            {
              "type" : "java.util.Map",
              "name" : "ctx"
            }
          ]
        },
        {
          "name" : "getParams",
          "return_type" : "java.util.Map",
          "params" : [ ]
        }
      ]
    },
    {
      "name" : "interval",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "boolean",
          "params" : [
            {
              "type" : "org.opensearch.index.query.IntervalFilterScript$Interval",
              "name" : "interval"
            }
          ]
        }
      ]
    },
    {
      "name" : "moving-function",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "double",
          "params" : [
            {
              "type" : "java.util.Map",
              "name" : "params"
            },
            {
              "type" : "double[]",
              "name" : "values"
            }
          ]
        }
      ]
    },
    {
      "name" : "number_sort",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "double",
          "params" : [ ]
        },
        {
          "name" : "getDoc",
          "return_type" : "java.util.Map",
          "params" : [ ]
        },
        {
          "name" : "getParams",
          "return_type" : "java.util.Map",
          "params" : [ ]
        },
        {
          "name" : "get_score",
          "return_type" : "double",
          "params" : [ ]
        }
      ]
    },
    {
      "name" : "painless_test",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "java.lang.Object",
          "params" : [ ]
        },
        {
          "name" : "getParams",
          "return_type" : "java.util.Map",
          "params" : [ ]
        }
      ]
    },
    {
      "name" : "processor_conditional",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "boolean",
          "params" : [
            {
              "type" : "java.util.Map",
              "name" : "ctx"
            }
          ]
        },
        {
          "name" : "getParams",
          "return_type" : "java.util.Map",
          "params" : [ ]
        }
      ]
    },
    {
      "name" : "score",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "double",
          "params" : [
            {
              "type" : "org.opensearch.script.ScoreScript$ExplanationHolder",
              "name" : "explanation"
            }
          ]
        },
        {
          "name" : "getDoc",
          "return_type" : "java.util.Map",
          "params" : [ ]
        },
        {
          "name" : "getParams",
          "return_type" : "java.util.Map",
          "params" : [ ]
        },
        {
          "name" : "get_score",
          "return_type" : "double",
          "params" : [ ]
        }
      ]
    },
    {
      "name" : "script_heuristic",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "double",
          "params" : [
            {
              "type" : "java.util.Map",
              "name" : "params"
            }
          ]
        }
      ]
    },
    {
      "name" : "similarity",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "double",
          "params" : [
            {
              "type" : "double",
              "name" : "weight"
            },
            {
              "type" : "org.opensearch.index.similarity.ScriptedSimilarity$Query",
              "name" : "query"
            },
            {
              "type" : "org.opensearch.index.similarity.ScriptedSimilarity$Field",
              "name" : "field"
            },
            {
              "type" : "org.opensearch.index.similarity.ScriptedSimilarity$Term",
              "name" : "term"
            },
            {
              "type" : "org.opensearch.index.similarity.ScriptedSimilarity$Doc",
              "name" : "doc"
            }
          ]
        }
      ]
    },
    {
      "name" : "similarity_weight",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "double",
          "params" : [
            {
              "type" : "org.opensearch.index.similarity.ScriptedSimilarity$Query",
              "name" : "query"
            },
            {
              "type" : "org.opensearch.index.similarity.ScriptedSimilarity$Field",
              "name" : "field"
            },
            {
              "type" : "org.opensearch.index.similarity.ScriptedSimilarity$Term",
              "name" : "term"
            }
          ]
        }
      ]
    },
    {
      "name" : "string_sort",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "java.lang.String",
          "params" : [ ]
        },
        {
          "name" : "getDoc",
          "return_type" : "java.util.Map",
          "params" : [ ]
        },
        {
          "name" : "getParams",
          "return_type" : "java.util.Map",
          "params" : [ ]
        },
        {
          "name" : "get_score",
          "return_type" : "double",
          "params" : [ ]
        }
      ]
    },
    {
      "name" : "template",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "java.lang.String",
          "params" : [ ]
        },
        {
          "name" : "getParams",
          "return_type" : "java.util.Map",
          "params" : [ ]
        }
      ]
    },
    {
      "name" : "terms_set",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "java.lang.Number",
          "params" : [ ]
        },
        {
          "name" : "getDoc",
          "return_type" : "java.util.Map",
          "params" : [ ]
        },
        {
          "name" : "getParams",
          "return_type" : "java.util.Map",
          "params" : [ ]
        }
      ]
    },
    {
      "name" : "trigger",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "boolean",
          "params" : [
            {
              "type" : "org.opensearch.alerting.script.QueryLevelTriggerExecutionContext",
              "name" : "ctx"
            }
          ]
        },
        {
          "name" : "getParams",
          "return_type" : "java.util.Map",
          "params" : [ ]
        }
      ]
    },
    {
      "name" : "update",
      "methods" : [
        {
          "name" : "execute",
          "return_type" : "void",
          "params" : [ ]
        },
        {
          "name" : "getCtx",
          "return_type" : "java.util.Map",
          "params" : [ ]
        },
        {
          "name" : "getParams",
          "return_type" : "java.util.Map",
          "params" : [ ]
        }
      ]
    }
  ]
}
````

## 响应字段

这`GET _script_context` 请求返回以下响应字段：

| 场地| 数据类型| 描述| 
:--- | :--- | :---
| 上下文| 列表| 所有上下文的列表。看[脚本对象](#script-context)。|

#### 脚本上下文

| 场地| 数据类型| 描述| 
:--- | :--- | :---
| 姓名| 细绳| 上下文名称。|
|  方法| 列表| 上下文的允许方法列表。看[脚本对象](#context-methods)。|

#### 上下文方法

| 场地| 数据类型| 描述| 
:--- | :--- | :---
| 姓名| 细绳| 方法名称。|
| 姓名| 细绳| 键入该方法返回（`boolean`，`object`，`number`， 等等）。|
| 参数| 列表| 该方法接受的参数列表。看[脚本对象](#method-parameters)。|

#### 方法参数

| 场地| 数据类型| 描述| 
:--- | :--- | :---
| 类型| 细绳| 参数数据类型。| 
| 姓名| 细绳| 参数名称。

