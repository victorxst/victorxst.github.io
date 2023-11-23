---
layout: default
title: 刷新搜索分析器
nav_order: 50
has_toc: false
redirect_from: 
  - /query-dsl/analyzers/refresh-analyzer/
  - /im-plugin/refresh-analyzer/index/
---

# 刷新搜索分析器

安装 ISM 后，你可以使用以下 API 实时刷新搜索分析器：

```json
POST /_plugins/_refresh_search_analyzers/<index or alias or wildcard>
```
例如，如果在分析器中更改同义词列表，则更改将生效，而无需关闭并重新打开索引。

若要工作，令牌筛选器必须具有 `updateable` 以下 `true` 标志：

```json
{
  "analyzer": {
    "my_synonyms": {
      "tokenizer": "whitespace",
      "filter": [
        "synonym"
      ]
    }
  },
  "filter": {
    "synonym": {
      "type": "synonym_graph",
      "synonyms_path": "synonyms.txt",
      "updateable": true
    }
  }
}
```
