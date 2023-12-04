---
layout: default
title: 节点用法
parent: 节点API
nav_order: 40
---

# 节点用法
**引入1.0**
{: .label .label-purple }

节点的使用端点返回低-有关节点上的休息操作使用情况的关卡信息。

## 路径和HTTP方法

```
GET _nodes/usage
GET _nodes/<nodeId>/usage
GET _nodes/usage/<metric>
GET _nodes/<nodeId>/usage/<metric>
```

## 路径参数

您可以在请求中包含以下可选路径参数。

范围| 类型| 描述
:--- | :--- | :---
nodeid| 细绳| 逗号-用于过滤结果的节点的分离列表。支持[节点过滤器]({{site.url}}{{site.baseurl}}/api-reference/nodes-apis/index/#node-filters)。默认为`_all`。
公制| 细绳| 响应中将包含的指标。您可以将字符串设置为`_all` 或者`rest_actions`。`rest_actions` 返回在节点上调用操作的总数。`_all` 从节点返回所有统计数据。默认为`_all`。

## 查询参数

您可以在请求中包含以下可选查询参数。

范围| 类型| 描述
:--- | :---| ：---
暂停| 时间| 设置节点响应的时间限制。默认为`30s`。
cluster_manager_timeout| 时间| 设置群集管理器响应的时间限制。默认为`30s`。

#### 示例请求

以下请求返回所有节点的使用详细信息：

```
GET _nodes/usage
```
{% include copy-curl.html %}

#### 示例响应

以下是一个示例响应：

```json
{
  "_nodes" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "cluster_name" : "opensearch-cluster",
  "nodes" : {
    "t7uqHu4SSuWObK3ElkCRfw" : {
      "timestamp" : 1665695174312,
      "since" : 1663994849643,
      "rest_actions" : {
        "opendistro_get_rollup_action" : 3,
        "nodes_usage_action" : 1,
        "list_dangling_indices" : 1,
        "get_index_template_action" : 258,
        "nodes_info_action" : 152665,
        "get_mapping_action" : 259,
        "get_data_streams_action" : 12,
        "cat_indices_action" : 6,
        "get_indices_action" : 3,
        "ism_explain_action" : 7,
        "nodes_reload_action" : 1,
        "get_policy_action" : 3,
        "PerformanceAnalyzerClusterConfigAction" : 2,
        "index_policy_action" : 1,
        "rank_eval_action" : 3,
        "search_action" : 592,
        "get_aliases_action" : 258,
        "document_mget_action" : 2,
        "document_get_action" : 30,
        "count_action" : 1,
        "main_action" : 1
      },
      "aggregations" : { }
    }
  }
}
```

## 所需的权限

如果使用安全插件，请确保设置以下权限：`cluster:manage/nodes` 或者`cluster:monitor/nodes`

