---
layout: default
title: 创建快照
parent: 快照API
nav_order: 5
---

# 创建快照
**引入1.0**
{: .label .label-purple }

在现有存储库中创建快照。

*要了解有关快照的更多信息，请参阅[快照]({{site.url}}{{site.baseurl}}/opensearch/snapshots/index)。

*要查看您的存储库列表，请参阅[获取快照存储库]({{site.url}}{{site.baseurl}}/api-reference/snapshots/get-snapshot-repository)。

## 路径和HTTP方法

```json
PUT /_snapshot/<repository>/<snapshot>
POST /_snapshot/<repository>/<snapshot>
```

## 路径参数

范围| 数据类型| 描述
:--- | :--- | :---
存储库| 细绳| repostory名称包含快照。|
快照| 细绳| 快照的名称要创建。|

## 查询参数

范围| 数据类型| 描述
:--- | :--- | :---
wait_for_completion| 布尔|  是否要等待快照创建才能完成。如果包含此参数，则在完成后返回快照定义。|

## 请求字段

请求主体是可选的。

场地| 数据类型| 描述
:--- | :--- | :---
`indices` | 细绳| 您要在快照中包含的索引。您可以使用`,` 要创建索引列表，`*` 指定索引模式，并`-` 排除某些指数。不要在项目之间放置空间。默认是所有索引。
`ignore_unavailable` | 布尔| 如果是`indices` 列表不存在，是否忽略它而不是失败快照。默认值为false。
`include_global_state` | 布尔| 是否将群集状态包括在快照中。默认是正确的。
`partial` | 布尔| 是否允许部分快照。默认值为false，如果一个或多个碎片无法存放，则会使整个快照失败

#### 示例请求

##### 没有身体的要求

以下请求创建了一个名为的快照`my-first-snapshot` 在一个称为的S3存储库中`my-s3-repository`。不包括请求主体，因为它是可选的。

```json
POST _snapshot/my-s3-repository/my-first-snapshot
```
{% include copy-curl.html %}

##### 与身体请求

您还可以添加一个请求主体以包括或排除某些索引或指定其他设置：

```json
PUT _snapshot/my-s3-repository/2
{
  "indices": "opensearch-dashboards*,my-index*,-my-index-2016",
  "ignore_unavailable": true,
  "include_global_state": false,
  "partial": false
}
```
{% include copy-curl.html %}

#### 示例响应

成功后，响应内容取决于您是否包括`wait_for_completion` 查询参数。

##### `wait_for_completion` 不包含

```json
{
  "accepted": true
}
```

要验证创建快照，请使用[获取快照]({{site.url}}{{site.baseurl}}/api-reference/snapshots/get-snapshot) API，将快照名称传递给`snapshot` 路径参数。
{: .note}

##### `wait_for_completion` 包括

快照定义将返回。

```json
{
  "snapshot" : {
    "snapshot" : "5",
    "uuid" : "ZRH4Zv7cSnuYev2JpLMJGw",
    "version_id" : 136217927,
    "version" : "2.0.1",
    "indices" : [
      ".opendistro-reports-instances",
      ".opensearch-observability",
      ".kibana_1",
      "opensearch_dashboards_sample_data_flights",
      ".opensearch-notifications-config",
      ".opendistro-reports-definitions",
      "shakespeare"
    ],
    "data_streams" : [ ],
    "include_global_state" : true,
    "state" : "SUCCESS",
    "start_time" : "2022-08-10T16:52:15.277Z",
    "start_time_in_millis" : 1660150335277,
    "end_time" : "2022-08-10T16:52:18.699Z",
    "end_time_in_millis" : 1660150338699,
    "duration_in_millis" : 3422,
    "failures" : [ ],
    "shards" : {
      "total" : 7,
      "failed" : 0,
      "successful" : 7
    }
  }
}
```

#### 响应字段

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
| 状态| 细绳| 快照状态。可能的值：`IN_PROGRESS`，`SUCCESS`，`FAILED`，`PARTIAL`。|
| 远程_store_index_shallow_copy| 布尔| 远程存储索引的快照是否被捕获为浅副本。默认为`false`。

