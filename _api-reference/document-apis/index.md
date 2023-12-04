---
layout: default
title: 文档API
has_children: true
nav_order: 25
redirect_from:
  - /opensearch/rest-api/document-apis/index/
---

# 文档API
**引入1.0**
{: .label .label-purple }

文档API允许您处理相对于索引的文档，例如添加，更新和删除文档。

文档API分为两个类别：单个文档操作和多个-文件操作。并发-文档操作提供比提交许多个人请求的绩效优势，因此，只要实用，我们建议您使用多个请求-文件操作。

## 单个文档操作

- 指数
- 得到
- 删除
- 更新

## 并发-文件操作

- 大部分
- 多获取
- 通过查询删除
- 通过查询更新
- reindex

