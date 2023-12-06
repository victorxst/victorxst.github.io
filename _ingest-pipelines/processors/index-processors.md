---
layout: default
title: 摄入的处理器
nav_order: 30
has_children: true
redirect_from:
   - /api-reference/ingest-apis/ingest-processors/
---

# 摄入的处理器
**引入1.0**
{: .label .label-purple }

摄入处理器是[摄取管道]({{site.url}}{{site.baseurl}}/ingest-pipelines/index/)。他们在索引之前预处理文件。例如，您可以删除字段，从文本中提取值，转换数据格式或附加其他信息。

OpenSearch在您的OpenSearch安装中提供了一组标准的摄入处理器。对于OpenSearch中可用的处理器列表，请使用[节点信息]({{site.url}}{{site.baseurl}}/api-reference/nodes-apis/nodes-info/) API操作：

```json
GET /_nodes/ingest?filter_path=nodes.*.ingest.processors
```
{% include copy-curl.html %}

要设置和部署摄入的处理器，请确保您拥有必要的权限和访问权限。看[安全插件REST API]({{site.url}}{{site.baseurl}}/security/access-control/api/) 了解更多。
{:.note}

处理器类型及其所需或可选参数因您的特定用例而异。看到[摄入的处理器]({{site.url}}{{site.baseurl}}/api-reference/ingest-apis/ingest-processors/) 一节了解有关处理器类型的更多信息，并在管道中定义和配置它们。

