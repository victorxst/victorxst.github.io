---
layout: default
title: 引入管道
nav_order: 5
nav_exclude: true
redirect_from:
   - /api-reference/ingest-apis/ingest-pipelines/
---

# 引入管道
**引入 1.0** {：.label .label-purple }

An _引入管道_是在将文档引入索引时应用于文档的序列_处理器_。管道中的每个[processor]({{site.url}}{{site.baseurl}}/ingest-pipelines/processors/index-processors/)管道都执行特定任务，例如筛选、转换或扩充数据。

处理器是可自定义的任务，它们在请求正文中显示时按顺序运行。此顺序很重要，因为每个处理器都依赖于前一个处理器的输出。应用处理器后，修改后的文档将显示在索引中。

引入管道只能使用[引入 API 操作]({{site.url}}{{site.baseurl}}/api-reference/ingest-apis/index/)进行管理。{：.note}

## 先决条件

以下是使用 OpenSearch 摄取管道的先决条件：

- 在生产环境中使用引入时，集群应至少包含一个节点角色权限设置为 `ingest` 的节点。有关在群集中设置节点角色的信息，请参见[集群形成]({{site.url}}{{site.baseurl}}/opensearch/cluster/)。
- 如果启用了 OpenSearch 安全插件，你必须具有 `cluster_manage_pipelines` 管理摄取管道的权限。

## 定义管道

A _管道定义_描述引入管道的顺序，可以用 JSON 格式编写。引入管道由以下部分组成：

```json
{
    "description" : "..."
    "processors" : [...]
}
```

#### 请求正文字段

字段 | 必需 | 类型 | 描述
:--- | :--- | :--- | :---
 `processors` | 必需 | 处理器对象数组 | 在将数据提取到 OpenSearch 中时执行特定数据处理任务的组件。
 `description` | 可选 | 字符串 | 引入管道的说明。

## 后续步骤

了解如何：

- [创建流水线]({{site.url}}{{site.baseurl}}/ingest-pipelines/create-ingest/).
- [测试管道]({{site.url}}{{site.baseurl}}/ingest-pipelines/simulate-ingest/).
- [检索有关管道的信息]({{site.url}}{{site.baseurl}}/ingest-pipelines/get-ingest/).
- [删除管道]({{site.url}}{{site.baseurl}}/ingest-pipelines/delete-ingest/).
