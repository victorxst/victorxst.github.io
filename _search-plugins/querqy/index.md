---
layout: default
title: Querqy
has_children: false
redirect_from:
  - /search-plugins/querqy/
nav_order: 210
---

# querqy

Querqy是一个用于查询重写的社区插件，有助于解决相关性问题，从而使搜索引擎在匹配和评分方面更加精确。

当前仅在OpenSearch 2.3中支持Querqy。
{： 。警告 }

## querqy插件安装

现在可用于OpenSearch 2.3.0。运行以下命令以安装Querqy插件。

````bash
./bin/opensearch-plugin install \
   "https://repo1.maven.org/maven2/org/querqy/opensearch-querqy/1.0.os2.3.0/opensearch-querqy-1.0.os2.3.0.zip"
````

回答`yes` 在安装期间的安全提示下，由于Querqy需要加载查询重写器的其他权限。

安装Querqy插件后，您可以在querqy.org网站上找到全面的文档：[querqy](https://docs.querqy.org/querqy/index.html)

## 路径和HTTP方法

```
POST /myindex/_search
```

## 示例查询

````json
{
   "query": {
       "querqy": {
           "matching_query": {
               "query": "books"
           },
           "query_fields": [ "title^3.0", "words^2.1", "shortSummary"]
       }
   }
}
````
