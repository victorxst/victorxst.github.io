---
layout: default
title: PPL
parent: SQL和PPL
nav_order: 5
has_children: true
has_toc: false
redirect_from:
  - /search-plugins/sql/ppl/
  - /search-plugins/ppl/
  - /observability-plugin/ppl/
  - /search-plugins/ppl/index/
  - /search-plugins/ppl/endpoint/
  - /search-plugins/ppl/protocol/
---

# ppl

管道处理语言（PPL）是一种查询语言，专注于在连续的，步骤中处理数据-经过-步骤态。PPL使用管道（`|`）操作员结合命令以查找和检索数据。这是在OpenSearch中使用可观察性的主要语言，并支持多种语言-数据查询。

## ppl语法

以下示例显示了基本的PPL语法：

```sql
search source=<index-name> | <command_1> | <command_2> | ... | <command_n>
```
{% include copy.html %}

看[句法]({{site.url}}{{site.baseurl}}/search-plugins/sql/ppl/syntax/) 对于特定的PPL语法示例。

## ppl命令

PPL过滤器，使用一系列命令来转换和汇总数据。看[命令]({{site.url}}{{site.baseurl}}/search-plugins/sql/ppl/functions/) 对于每个命令的描述和示例。

## 在OpenSearch中使用PPL

要使用PPL，您必须已经安装了OpenSearch仪表板。PPL可在[查询工作台工具](https://playground.opensearch.org/app/opensearch-query-workbench#/)。看到[查询工作台]({{site.url}}{{site.baseurl}}/dashboards/query-workbench/) 有关在OpenSearch中使用PPL的教程的文档。

## 开发人员文档

开发人员可以在以下资源中找到信息：

- [管道处理语言](https://github.com/opensearch-project/piped-processing-language) 规格
- [OpenSearch PPL参考手册](https://github.com/opensearch-project/sql/blob/main/docs/user/ppl/index.rst)
- [可观察性](https://github.com/opensearch-project/dashboards-observability/) 使用[ppl-基于可视化](https://github.com/opensearch-project/dashboards-observability#event-analytics)
- ppl[数据类型](https://github.com/opensearch-project/sql/blob/main/docs/user/ppl/general/datatypes.rst)

