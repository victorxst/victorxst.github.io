---
layout: default
title: 索引转换
nav_order: 20
has_children: true
redirect_from: /im-plugin/index-transforms/
has_toc: false
---

# 索引转换

索引汇总作业允许你通过将旧数据汇总到压缩索引中来降低数据粒度，而转换作业允许你创建以特定字段为中心的不同数据汇总视图，以便你可以以不同的方式可视化或分析数据。

例如，假设你的航空公司数据分散在多个字段和类别中，并且你想要查看按航空公司、季度和价格组织的数据摘要。可以使用转换作业创建按这些特定类别组织的新汇总索引。

可以通过两种方式使用转换作业：

1. 使用 OpenSearch 控制面板 UI 指定要转换的索引以及要用于筛选原始索引的任何可选数据筛选条件。然后选择要转换的字段以及要在转换中使用的聚合。最后，为你的工作定义一个时间表。
1. 使用转换 API 指定有关作业的所有详细信息：要转换的索引、希望转换后的索引具有的目标组、要用于对列进行分组的任何聚合以及作业要遵循的计划。

OpenSearch 控制面板提供你创建的任务及其相关信息（例如关联的索引和任务状态）的详细摘要。你可以在创建作业之前查看和编辑作业的详细信息和选择，甚至可以在选择要转换的字段时预览转换后的索引的数据。但是，你也可以使用 REST API 创建转换作业并预览转换作业结果，但你必须知道所有必要的设置和参数才能将它们作为 HTTP 请求正文的一部分提交。将转换任务配置作为 JSON 脚本提交可为你提供更高的可移植性，允许你共享和复制转换任务，而使用 OpenSearch 控制面板则很难做到这一点。

你的用例将帮助你决定使用哪种方法来创建转换作业。

## 创建转换作业

如果你的集群中没有任何数据，则可以使用 OpenSearch 控制面板中的示例航班数据来尝试转换作业。否则，在启动 OpenSearch 控制面板后，选择**索引管理**.选择**转换工作**，然后选择**创建转换作业**。

### 第 1 步：选择索引

1. 在该**职位名称和描述**部分中，指定作业的名称和可选描述。
2. 在该**指标**部分中，选择源索引和目标索引。你可以选择现有目标索引，也可以通过输入新索引的名称来创建新目标索引。如果你只想转换源索引的子集，请选择**编辑数据筛选器**，然后使用 OpenSearch 查询 DSL 指定源索引的子集。有关 OpenSearch 查询 DSL 的更多信息，请参阅[query DSL]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/)。
3. 选择**下一个**。

### 第 2 步：选择要转换的字段

指定索引后，你可以选择要在转换作业中使用的字段，以及是使用分组还是聚合。

你可以使用分组将数据放入转换后的索引中的单独存储桶中。例如，如果要对示例航班数据中的所有机场目的地进行分组，则可以将 `DestAirportID` 该字段分组到字段的目标 `DestAirportID_terms` 字段中，并且可以在转换作业完成后在转换后的索引中找到分组的机场 ID。

另一方面，聚合允许你执行简单的计算。例如，你可以在转换作业中包含聚合，以定义一个新字段，该 `sum_of_total_ticket_price` 字段用于计算所有机票的总和，然后在转换后的索引中分析新的夏季数据。

1. 在数据表中，选择要转换的字段，然后展开列标题中的下拉菜单以选择要使用的分组或聚合。

    目前，转换作业支持直方图、date_histogram 和术语分组。有关分组的详细信息，请参阅[存储桶聚合]({{site.url}}{{site.baseurl}}/opensearch/bucket-agg/)。在聚合方面，你可以从 `sum`、、、、 `min` `max` `value_count` `avg` 和 `percentiles` `scripted_metric` 中进行选择。有关聚合的详细信息，请参阅[指标聚合]({{site.url}}{{site.baseurl}}/opensearch/metric-agg/)。

1. 对要转换的任何其他字段重复步骤 1。
1. 选择要转换的字段并验证转换后，选择**下一个**。

### 第 3 步：指定计划

你可以将转换作业配置为按计划运行一次或多次。默认情况下，转换作业处于启用状态。

1. 选择作业是否应为**连续的**.连续作业在每个**转换执行间隔**存储桶上执行，并以增量方式转换新修改的存储桶，其中可以包括添加到源索引的新数据。非连续作业仅执行一次。
1. 对于**转换执行间隔**，指定转换间隔（以分钟、小时或天为单位）。此间隔表示连续作业的执行频率，非连续作业在间隔过后执行一次。
1. 在下**高深**，指定的**每次执行的页数**可选金额。数字越大，意味着每个搜索请求中处理的数据越多，但也会占用更多的内存并导致更高的延迟。超出允许的内存限制可能会导致发生异常和错误。
1. 选择**下一个**。

### 第 4 步：查看并确认详细信息

确认转换作业的详细信息正确无误后，选择**创建转换作业**。如果要编辑作业的任何部分，请选择**编辑**要更改的部分，然后进行必要的更改。创建作业后，无法更改聚合或分组。

### 第 5 步：搜索转换后的索引。

转换作业完成后，你可以使用 `_search` API 操作搜索目标索引。

```json
GET <target_index>/_search
```

例如，在运行基于 `DestAirportID` 字段转换飞行数据的转换作业后，你可以运行以下请求，该请求返回值为的所有 `SFO` 字段。

**样品申请**

```json
GET finished_flight_job/_search
{
  "query": {
    "match": {
      "DestAirportID_terms" : "SFO"
    }
  }
}
```

**示例响应**

```json
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4,
      "relation" : "eq"
    },
    "max_score" : 3.845883,
    "hits" : [
      {
        "_index" : "finished_flight_job",
        "_id" : "dSNKGb8U3OJOmC4RqVCi1Q",
        "_score" : 3.845883,
        "_source" : {
          "transform._id" : "sample_flight_job",
          "transform._doc_count" : 14,
          "Carrier_terms" : "Dashboards Airlines",
          "DestAirportID_terms" : "SFO"
        }
      },
      {
        "_index" : "finished_flight_job",
        "_id" : "_D7oqOy7drx9E-MG96U5RA",
        "_score" : 3.845883,
        "_source" : {
          "transform._id" : "sample_flight_job",
          "transform._doc_count" : 14,
          "Carrier_terms" : "Logstash Airways",
          "DestAirportID_terms" : "SFO"
        }
      },
      {
        "_index" : "finished_flight_job",
        "_id" : "YuZ8tOt1OsBA54e84WuAEw",
        "_score" : 3.6988301,
        "_source" : {
          "transform._id" : "sample_flight_job",
          "transform._doc_count" : 11,
          "Carrier_terms" : "ES-Air",
          "DestAirportID_terms" : "SFO"
        }
      },
      {
        "_index" : "finished_flight_job",
        "_id" : "W_-e7bVmH6eu8veJeK8ZxQ",
        "_score" : 3.6988301,
        "_source" : {
          "transform._id" : "sample_flight_job",
          "transform._doc_count" : 10,
          "Carrier_terms" : "JetBeats",
          "DestAirportID_terms" : "SFO"
        }
      }
    ]
  }
}

```

## 索引编解码器注意事项

有关索引编解码器注意事项，请参见[索引编解码器]({{site.url}}{{site.baseurl}}/im-plugin/index-codecs/#index-rollups-and-transforms)。