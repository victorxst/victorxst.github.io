---
layout: default
title: 相关引擎API
parent: API工具
nav_order: 55
---


# 相关引擎API

相关引擎API允许您在特定时间窗口内创建新的相关规则，查看发现和相关性，并执行其他任务。

---
## Create correlation rules between log types

此API用于创建相关规则：

```json
post/_plugins/_security_analytics/corserReation/Rules
```

### Request fields

| 场地| 类型| 描述|
| ：--- | ：--- |：--- |
| `index` | 细绳| 索引的名称用作日志源。|
| `query` | 细绳| 用于过滤安全日志的查询以进行关联。|
| `category` | 细绳| 与日志源关联的日志类型。|

#### Example request

```json
post/_plugins/_security_analytics/corserReation/Rules
{
  "correlate"：[[
    {
      "index"："vpc_flow"，，，，
      "query"："dstaddr:4.5.6.7 or dstaddr:4.5.6.6"，，，，
      "category"："network"
    }，，
    {
      "index"："windows"，，，，
      "query"："winlog.event_data.SubjectDomainName:NTAUTHORI*"，，，，
      "category"："windows"
    }，，
    {
      "index"："ad_logs"，，，，
      "query"："ResultType:50126"，，，，
      "category"："ad_ldap"
    }，，
    {
      "index"："app_logs"，，，，
      "query"："endpoint:/customer_records.txt"，，，，
      "category"："others_application"
    }
  这是给出的
}
```
{% include copy-curl.html %}

#### Example response

```json
{
  "_id"："DxKEUIkBpIjg64IK4nXg"，，，，
  "_version"：1，
  "rule"：{
    "name"： 无效的，
    "correlate"：[[
      {
        "index"："vpc_flow"，，，，
        "query"："dstaddr:4.5.6.7 or dstaddr:4.5.6.6"，，，，
        "category"："network"
      }，，
      {
        "index"："windows"，，，，
        "query"："winlog.event_data.SubjectDomainName:NTAUTHORI*"，，，，
        "category"："windows"
      }，，
      {
        "index"："ad_logs"，，，，
        "query"："ResultType:50126"，，，，
        "category"："ad_ldap"
      }，，
      {
        "index"："app_logs"，，，，
        "query"："endpoint:/customer_records.txt"，，，，
        "category"："others_application"
      }
    这是给出的
  }
}
```

### Response fields

| 场地| 类型| 描述|
| ：--- | ：--- |：--- |
| `_id` | 细绳| 新规则的ID。|

---
## 在一个时间窗口中列出所有发现及其相关性

该API在指定的时间窗口中提供了所有发现及其相关性的列表：

```json
GET /_plugins/_security_analytics/correlations?start_timestamp=<start time in milliseconds>&end_timestamp=<end time in milliseconds>
```

### 查询参数

| 范围| 类型| 描述|
| ：--- | ：--- |：--- |
| `start_timestamp` | 数字| 开始时间窗口的时间，以毫秒为单位。|
| `end_timestamp` | 数字| 时间窗口的结束时间，以毫秒为单位。|

#### 示例请求

```json
GET /_plugins/_security_analytics/correlations?start_timestamp=1689289210000&end_timestamp=1689300010000
```
{％包含副本-curl.html％}

#### 示例响应

```json
{
  "findings": [
    {
      "finding1": "931de5f0-a276-45d5-9cdb-83e1045a3630",
      "logType1": "network",
      "finding2": "1e6f6a12-83f1-4a38-9bb8-648f196859cc",
      "logType2": "test_windows",
      "rules": [
        "nqI2TokBgL5wWFPZ6Gfu"
      ]
    }
  ]
}
```

### 响应字段

| 场地| 类型| 描述|
| ：--- | ：--- |：--- |
| `finding1` | 细绳| 相关性第一个发现的ID。|
| `logType1` | 细绳| 与第一个发现关联的日志类型。|
| `finding2` | 细绳| 相关性中第二个发现的ID。|
| `logType2` | 细绳| 与第二个发现关联的日志类型。|
| `rules` | 大批| 与相关发现相关的相关规则ID列表。|

---
## List correlations for a finding belonging to a log type

此API用于列出特定发现和与之关联的日志类型的相关性：

```json
get/_plugins/_ security_analytics/findings/closelate？查找= 425DCE0B-F5EE-4889-B0C0-7d15669f0871＆detector_type = ad_ldap＆nearby_findings = 20＆time_window = 10m
```

### Query parameters

| 范围| 类型| 描述|
| ：--- | ：--- |：--- |
| `finding` | 细绳| 查找ID。|
| `detector_type` | 细绳| 检测器的日志类型。|
| `nearby_findings` | 数字| 附近的发现相对于给定的发现ID的数量。|
| `time_window` | 细绳| 设置一个时间窗口，其中所有相关都必须一起发生。|


#### Example request

```json
get/_plugins/_ security_analytics/findings/closelate？查找= 425DCE0B-F5EE-4889-B0C0-7d15669f0871＆detector_type = ad_ldap＆nearby_findings = 20＆time_window = 10m
```
{% include copy-curl.html %}

#### Example response

```json
{
  "findings"：[[
    {
      "finding"："5c661104-aaa9-484b-a91f-9cad4ae6d5f5"，，，，
      "detector_type"："others_application"，，，，
      "score"：0.000015182109564193524
    }，，
    {
      "finding"："2485b623-6573-42f4-a055-9b927e38a65f"，，，，
      "detector_type"："ad_ldap"，，，，
      "score"：0.000001615897872397909
    }，，
    {
      "finding"："051e00ad-5996-4c41-be20-f992451d1331"，，，，
      "detector_type"："windows"，，，，
      "score"：0.000016230604160227813
    }，，
    {
      "finding"："f11ca8a3-50d7-4074-a951-51439aa9e67b"，，，，
      "detector_type"："s3"，，，，
      "score"：0.000001759401811796124
    }，，
    {
      "finding"："9b86980e-5fb7-4c5a-bd1b-879a1e3baf12"，，，，
      "detector_type"："network"，，，，
      "score"：0.0000016306962606904563
    }，，
    {
      "finding"："e7dea5a1-164f-48f9-880e-4ba33e508713"，，，，
      "detector_type"："network"，，，，
      "score"：0.00001632626481296029
    }
  这是给出的
}
```

### Response fields

| 场地| 类型| 描述|
| ：--- | ：--- |：--- |
| `finding` | 细绳| 查找ID。|
| `detector_type` | 细绳| 与发现关联的日志类型。|
| `score` | 数字| 相关发现的相关得分。该分数基于相关场景中相关场景中相关发现的接近性。|


