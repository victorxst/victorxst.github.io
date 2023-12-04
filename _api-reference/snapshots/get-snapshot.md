---
layout: default
title: 获取快照
parent: 快照API
nav_order: 6
---

# 获取快照。
**引入1.0**
{: .label .label-purple }

检索有关快照的信息。

## 路径参数

| 范围| 数据类型| 描述|
| :--- | :--- | :--- |
| 存储库| 细绳| 包含要检索的快照的存储库。|
| 快照| 细绳| 快照要检索。

## 查询参数

| 范围| 数据类型| 描述| 
:--- | :--- | :---
| 冗长| 布尔| 是显示全部，还是仅显示基本快照信息。如果`true`，返回所有信息。如果`false`，省略诸如开始/结束时间，失败和碎片之类的信息。可选，默认为`true`。|
| ignore_unavailable| 布尔| 如何处理不可用的快照（损坏或暂时无法返回）。如果`true` 快照不可用，请求不会返回快照。如果`false` 快照不可用，请求返回错误。可选，默认为`false`。|

#### 示例请求

以下请求检索信息`my-first-snapshot` 位于`my-opensearch-repo` 存储库：

````json
GET _snapshot/my-opensearch-repo/my-first-snapshot
````
{% include copy-curl.html %}

#### 示例响应

成功后，响应返回快照信息：

````json
{
  "snapshots" : [
    {
      "snapshot" : "my-first-snapshot",
      "uuid" : "3P7Qa-M8RU6l16Od5n7Lxg",
      "version_id" : 136217927,
      "version" : "2.0.1",
      "indices" : [
        ".opensearch-observability",
        ".opendistro-reports-instances",
        ".opensearch-notifications-config",
        "shakespeare",
        ".opendistro-reports-definitions",
        "opensearch_dashboards_sample_data_flights",
        ".kibana_1"
      ],
      "data_streams" : [ ],
      "include_global_state" : true,
      "state" : "SUCCESS",
      "start_time" : "2022-08-11T20:30:00.399Z",
      "start_time_in_millis" : 1660249800399,
      "end_time" : "2022-08-11T20:30:14.851Z",
      "end_time_in_millis" : 1660249814851,
      "duration_in_millis" : 14452,
      "failures" : [ ],
      "shards" : {
        "total" : 7,
        "failed" : 0,
        "successful" : 7
      }
    }
  ]
}
````
## 响应字段

| 场地| 数据类型| 描述|
| :--- | :--- | :--- | 
| 快照| 细绳| 快照名称。|
| UUID| 细绳| 快照的普遍唯一标识符（UUID）。|
| version_id| int| 构建创建快照的打开搜索版本的ID。|
| 版本| 漂浮| 打开创建快照的搜索版本。|
| 指数| 大批| 快照中的索引。|
| data_streams| 大批| 快照中的数据流。|
| 包括_global_state| 布尔| 快照中是否包含当前的群集状态。|
| 开始时间| 细绳| 快照创建过程开始的日期/时间。|
| start_time_in_millis| 长的| 时间（以毫秒为单位）开始快照创建过程。|
| 时间结束| 细绳| 快照创建过程结束的日期/时间。|
| end_time_in_millis| 长的| 时间（以毫秒为单位）当快照创建过程结束时。|
| lisation_in_millis| 长的| 快照创建过程持续的总时间（以毫秒为单位）。|
| 失败| 大批| 快照创建期间发生的失败（如果有）。|
| 碎片| 目的| 创建的碎片总数以及成功和失败的碎片数量。|
| 状态| 细绳| 快照状态。可能的值：`IN_PROGRESS`，`SUCCESS`，`FAILED`，`PARTIAL`。

