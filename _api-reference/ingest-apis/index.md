---
layout: default
title: 摄入的API
has_children: false
nav_order: 40
redirect_from:
  - /opensearch/rest-api/ingest-apis/index/
---

# 摄入的API
**引入1.0**
{: .label .label-purple }

摄入API是将数据加载到系统中的宝贵工具。摄入的API与[摄取管道]({{site.url}}{{site.baseurl}}/api-reference/ingest-apis/ingest-pipelines/) 和[摄入的处理器]({{site.url}}{{site.baseurl}}/api-reference/ingest-apis/ingest-processors/) 从多种来源和各种格式处理或转换数据。

## 摄入管道API

用以下API简化，安全并扩展您的OpenSearch数据摄入：

- [创建管道]({{site.url}}{{site.baseurl}}/api-reference/ingest-apis/create-ingest/)：使用此API创建或更新管道配置。
- [获取管道]({{site.url}}{{site.baseurl}}/api-reference/ingest-apis/get-ingest/)：使用此API检索管道配置。
- [模拟管道]({{site.url}}{{site.baseurl}}/api-reference/ingest-apis/simulate-ingest/)：使用此管道测试管道配置。
- [删除管道]({{site.url}}{{site.baseurl}}/api-reference/ingest-apis/delete-ingest/)：使用此API删除管道配置。

