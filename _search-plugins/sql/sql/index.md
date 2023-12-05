---
layout: default
title: SQL
parent: SQL和PPL
nav_order: 4
has_children: true
has_toc: false
redirect_from:
  - /search-plugins/sql/sql/index/

---

# SQL

OpenSearch中的SQL桥接传统关系数据库概念与OpenSearch文档的灵活性之间的差距-定向数据存储。这种集成使您能够利用您的SQL知识从OpenSearch数据查询，分析和提取见解。

## SQL和OpenSearch术语

这是核心SQL概念映射到OpenSearch的方式：

SQL| OpenSearch
:--- | :---
桌子| 指数
排| 文档
柱子| 场地

## REST API

有关SQL插件的完整REST API参考，请参见[SQL/PPL API]({{site.url}}{{site.baseurl}}/search-plugins/sql/sql-ppl-api/)。

要将SQL插件与您自己的应用一起使用，请将请求发送到`_plugins/_sql` 端点：

```json
POST _plugins/_sql
{
  "query": "SELECT * FROM my-index LIMIT 50"
}
```
{% include copy-curl.html %}

您可以使用逗号查询多个索引-分开列表：

```json
POST _plugins/_sql
{
  "query": "SELECT * FROM my-index1,myindex2,myindex3 LIMIT 50"
}
```
{% include copy-curl.html %}

您可以指定具有通配符表达式的索引模式：

```json
POST _plugins/_sql
{
  "query": "SELECT * FROM my-index* LIMIT 50"
}
```
{% include copy-curl.html %}

要在命令行中运行前面的查询，请使用[卷曲](https://curl.haxx.se/) 命令：

```bash
curl -XPOST https://localhost:9200/_plugins/_sql -u 'admin:admin' -k -H 'Content-Type: application/json' -d '{"query": "SELECT * FROM my-index* LIMIT 50"}'
```
{% include copy.html %}

您可以指定[响应格式]({{site.url}}{{site.baseurl}}/search-plugins/sql/response-formats/) 作为JDBC，标准OpenSearch JSON，CSV或RAW。默认情况下，查询以JDBC格式返回数据。以下查询将格式设置为JSON：

```json
POST _plugins/_sql?format=json
{
  "query": "SELECT * FROM my-index LIMIT 50"
}
```
{% include copy-curl.html %}

有关请求参数，设置，支持操作和工具的更多信息，请参阅下面的相关主题[SQL]({{site.url}}{{site.baseurl}}/search-plugins/sql/sql/index/)。

