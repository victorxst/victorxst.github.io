---
layout: default
title: 在开发工具控制台中运行查询
parent: 开发工具
nav_order: 10
redirect_from:
  - /dashboards/run-queries/
---

# 在开发工具控制台中运行查询

开发工具控制台可用于将查询发送到OpenSearch。要访问控制台，请转到OpenSearch仪表板主菜单，然后选择**管理** >**开发工具**。
## 编写查询

OpenSearch提供了一个查询域-特定语言（DSL）称为[查询DSL]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/)。这是一种具有JSON接口的灵活语言。

要编写查询，请使用控制台左侧的编辑窗格。要将查询发送到OpenSearch，请通过将光标放置在查询文本中，然后选择“ play图标”（{:: :: NOMARKDOWN} <img src =来选择查询。"{{site.url}}{{site.baseurl}}/images/icons/play-icon.png" class ="inline-icon" alt ="play icon"/> {:/}）在请求的右上角或按`Ctrl/Cmd+Enter`。OpenSearch的响应显示在控制台右侧的响应窗格中。要同时运行多个命令，请在编辑器窗格中选择所有命令，然后选择“播放”图标或按下`Ctrl/Cmd+Enter`。

以下图像显示了查询和响应窗格的一个示例。

<img src="{{site.url}}{{site.baseurl}}/images/dashboards/query-request-ui.png" alt="Console UI with query and request">

### 查询选项

在使用控制台编写查询时，有一些常见的操作可以帮助您更有效，准确地编写查询。下表描述了这些功能以及如何使用它们。

特征| 如何使用|
--------|------------|
**崩溃或扩展查询** | 要隐藏或显示查询的详细信息，请选择“ expander箭头”（{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/icons/arrow-down-icon.png" class ="inline-icon" alt ="arrow down icon"/> {:/}）在行号旁边。|
**自动缩进** | 要使用自动缩进，请选择要格式化的查询，然后选择扳手图标（{::nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/icons/wrench-icon.png" class ="inline-icon" alt ="wrench icon"/> {:/}），然后选择**自动凹痕**。|
**自动完成** | 要定义您对自动完成建议的偏好，请在**设置**。|
**请求历史记录** | 要查看请求历史记录，请选择**历史** 从顶部菜单。如果选择要从左窗格查看的请求，则查询在右窗格中显示。要将查询复制到编辑器窗格中，请选择查询文本，然后选择**申请**。要清理历史，请选择**清除**。|
**键盘快捷键** | 要查看所有可用的键盘快捷键，请选择**帮助** 从顶部菜单。|
**从控制台访问文档** | 要从控制台访问OpenSearch文档，请选择“扳手”图标（{::nomarkdown} <IMG SRC ="{{site.url}}{{site.baseurl}}/images/icons/wrench-icon.png" class ="inline-icon" alt ="wrench icon"/> {:/}），然后选择**打开文档**。|

## 以卷发和控制台格式工作

该控制台使用简化的语法将REST请求格式化，而不是`curl` 命令。如果你粘贴`curl` 命令直接进入控制台，将命令自动转换为控制台使用的格式。要以卷曲格式导入查询，请选择查询，然后选择扳手图标（{:: :: nomarkdown} <img src ="{{site.url}}{{site.baseurl}}/images/icons/wrench-icon.png" class ="inline-icon" alt ="wrench icon"/> {:/}），然后选择**复制为卷发**。

例如，以下`curl` 命令运行搜索查询：

```bash
curl -XGET http://localhost:9200/shakespeare/_search?pretty -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "text_entry": "To be, or not to be"
    }
  }
}'
```
{% include copy.html %}

同一查询具有控制台格式的简化语法，如下所示：

```json
GET shakespeare/_search
{
  "query": {
    "match": {
      "text_entry": "To be, or not to be"
    }
  }
}
```
{% include copy-curl.html %}}

