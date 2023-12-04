---
layout: default
title: 语言分析
nav_order: 10
redirect_from:
  - /query-dsl/analyzers/language-analyzers/
---

# 语言分析

OpenSearch用`analyzer` 选项：
`arabic`，`armenian`，`basque`，`bengali`，`brazilian`，`bulgarian`，`catalan`，`czech`，`danish`，`dutch`，`english`，`estonian`，`finnish`，`french`，`galician`，`german`，`greek`，`hindi`，`hungarian`，`indonesian`，`irish`，`italian`，`latvian`，`lithuanian`，`norwegian`，`persian`，`portuguese`，`romanian`，`russian`，`sorani`，`spanish`，`swedish`，`turkish`， 和`thai`。

要在映射索引时使用分析器，请在查询中指定值。例如，要使用法语分析仪映射您的索引，请指定`french` 分析仪字段的价值：

```json
 "analyzer": "french"
```

#### 示例请求

以下查询指定`french` 索引的语言分析`my-index`：

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "text": { 
        "type": "text",
        "fields": {
          "french": { 
            "type": "text",
            "analyzer": "french"
          }
        }
      }
    }
  }
}
```

<!-- 要做：每个选项都需要自己的部分，其中有一个示例。将表转换为各个部分，然后给出具有有效值的简化列表。--->

