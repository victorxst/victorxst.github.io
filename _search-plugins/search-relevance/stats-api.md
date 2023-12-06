---
layout: default
title: 搜索相关统计数据API
nav_order: 65
parent: 搜索相关性
has_children: false
---

# 搜索相关统计数据API
引入2.7
{: .label .label-purple }

搜索相关性统计信息API提供了有关[搜索相关插件](https://github.com/opensearch-project/dashboards-search-relevance) 运营。搜索相关插件处理由[比较搜索结果]({{site.url}}{{site.baseurl}}/search-plugins/search-relevance) 仪表板工具。

搜索相关统计数据API捕获了一个统计信息-在收到请求的少量间隔。例如，如果在23：59：59.004收到请求，则收集23：58：00.000的统计信息。--23：58：59.999时间间隔。

要更改收集统计信息的默认时间间隔，请更新`searchRelevanceDashboards.metrics.metricInterval` 设置在`opensearch_dashboards.yml` 用新的时间间隔以毫秒为单位提交。这`opensearch_dashboards.yml` 文件位于`config` OpenSearch仪表板安装的文件夹。例如，以下将间隔设置为一秒钟：

```yml
searchRelevanceDashboards.metrics.metricInterval: 1000 
```

#### 示例请求

您可以通过以下格式提供其URL地址来访问搜索相关性统计信息API：

```
<opensearch-dashboards-endpoint-address>/api/relevancy/stats
```

如果OpenSearch Configuration文件中指定了OpenSearch仪表板端点地址，则可能包含一个端口号。特定的URL格式取决于OpenSearch部署的类型及其托管的网络环境的类型。
{:.note}

您可以通过两种方式查询端点：
  
  - 通过访问端点地址（例如，`http://localhost:5601/api/relevancy/stats`）在浏览器中

  - 通过使用`curl` 终端中的命令：
    ```bash
    卷曲-x获取http：// localhost：5601/api/cessionancy/stats
    ```
    {% include copy.html %}

#### Example response

The following is the response for the preceding request:

```json
{
  "data": {
    "search_relevance": {
      "fetch_index": {
        "200": {
          "response_time_total": 28.79286289215088,
          "count": 1
        }
      },
      "single_search": {
        "200": {
          "response_time_total": 29.817723274230957,
          "count": 1
        }
      },
      "comparison_search": {
        "200": {
          "response_time_total": 13.265346050262451,
          "count": 2
        }
      }
    }
  },
  "overall": {
    "response_time_avg": 17.968983054161072,
    "requests_per_second": 0.06666666666666667
  },
  "counts_by_component": {
    "search_relevance": 4
  },
  "counts_by_status_code": {
    "200": 4
  }
}
```

## Response fields

The following table lists all response fields.

| Field | Data type | Description |
| :--- | :--- | :--- | 
| [`data.search_relevance`](#the-datasearch_relevance-object) | Object | Statistics related to Search Relevance operations. |
| `全面的` | Object | The average statistics for all operations. |
| `总体。Response_time_avg` | 双倍的| 所有操作的平均响应时间为毫秒。|
| `overall.requests_per_second` | 双倍的| 所有操作的平均每秒请求数。|
| `counts_by_component` | 目的| 所有的总和`count` 所有子对象的值`data` 目的。|
| `counts_by_component.search_relevance` | 所有操作的响应总数`search_relevance` 目的。|
| `counts_by_status_code` | 目的| 包含所有响应代码的列表及其对所有搜索相关操作的计数。|

### 这`data.search_relevance` 目的

这`data.search_relevance` 对象包含下表中描述的字段。

| 场地| 数据类型| 描述|
| :--- | :--- | :--- |
| `comparison_search` | 目的| 与比较搜索操作有关的统计数据。比较搜索操作是一个请求，当查询1和查询2都在比较搜索结果工具中输入时，可以比较两个查询。|
| `single_search` | 目的| 与单个搜索操作有关的统计信息。单个搜索操作是在仅查询1或查询2（并非两者）中输入比较搜索结果工具时运行单个查询的请求。|
| `fetch_index` | 目的| 与获取索引或索引的操作有关的统计信息进行比较搜索或单个搜索。|

每一个`comparison_search`，`single_search`， 和`fetch_index` 对象包含HTTP响应代码的列表。下表列出了每个响应代码的字段。

| 场地| 数据类型| 描述|
| :--- | :--- | :--- |
| `response_time_total` | 双倍的| 以毫秒为单位，使用此HTTP代码的响应时间的总和。|
| `count` | 整数| 此HTTP代码的响应总数。|

