---
layout: default
title: 警报和发现API
parent: API工具
nav_order: 50
---


# 警报和发现API

以下API可用于与警报和发现有关的任务。

---
## Get Alerts

提供了检索与特定检测器类型或检测器ID相关的警报的选项。

### Parameters

请求警报时，您可以指定以下参数。

范围| 描述
：--- | ：---
`detectorId` | 检测器的ID用于获取警报。可选时`detectorType` 指定。否则需要。
`detectorType` | 检测器的类型用于获取警报。可选时`detectorId` 指定。否则需要。
`severityLevel` | 用于通过警报严重程度过滤。选修的。
`alertState` | 用于通过警报状态过滤。可能的值：活动，确认，完成，错误，已删除。选修的。
`sortString` | 该字段指定字符串安全分析用来对警报进行排序的内容。选修的。
`sortOrder` | 用于对发现列表进行排序的顺序`ascending` 或者`descending`。选修的。
`missing` | 没有找到别名映射的字段列表。选修的。
`size` | 响应中返回的最大结果数的可选限制。选修的。
`startIndex` | 分页指示器。选修的。
`searchString` | 您希望在搜索中返回的警报属性。选修的。

### Example request

```json
获取/_plugins/_security_analytics/alerts？detectortepe = Windows
```

### Example response

```json
{
    "alerts"：[{{
        "detector_id"："detector_12345"，，，，
        "id"："alert_id_1"，，，，
        "version"：-3，
        "schema_version"：0，
        "trigger_id"："trigger_id_1"，，，，
        "trigger_name"："my_trigger"，，，，
        "finding_ids"：[["finding_id_1"]，，
        "related_doc_ids"：[["docId1"]，，
        "state"："ACTIVE"，，，，
        "error_message"： 无效的，
        "alert_history"：[]，，
        "severity"： 无效的，
        "action_execution_results"：[{{
            "action_id"："action_id_1"，，，，
            "last_execution_time"：1665693544996，
            "throttled_count"：0
        }]，
        "start_time"："2022-10-13T20:39:04.995023Z"，，，，
        "last_notification_time"："2022-10-13T20:39:04.995028Z"，，，，
        "end_time"："2022-10-13T20:39:04.995027Z"，，，，
        "acknowledged_time"："2022-10-13T20:39:04.995028Z"
    }]，
    "total_alerts"：1，
    "detectorType"："windows"
}
```

#### Response fields

警报持续存在，直到您解决根本原因并具有以下状态：

状态| 描述
：--- | ：---
`ACTIVE` | 该警报正在进行中，并未被确认。警报保持在此状态，直到您确认它们，删除与警报关联的触发器，或完全删除监视器。
`ACKNOWLEDGED` | 有人已经确认警报，但没有修复根本原因。
`COMPLETED` | 警报不再持续。在相应的触发器评估为false之后，警报输入此状态。
`ERROR` | 执行触发器时发生错误。此错误通常是触发器或目的地不良的结果。
`DELETED` | 警报持续时，有人删除了检测器或与此警报相关的触发器。

---
## 确认警报

### 示例请求

```json
POST /_plugins/_security_analytics/<detector_id>/_acknowledge/alerts

{"alerts":["4dc7f5a9-2c82-4786-81ca-433a209d5205"]}
```

### 示例响应

```json
{
  "acknowledged": [
    {
      "detector_id": "8YT5fYQBZ8IUM4axics6",
      "id": "4dc7f5a9-2c82-4786-81ca-433a209d5205",
      "version": 1,
      "schema_version": 4,
      "trigger_id": "1TP5fYQBMkkIGY6Pg-q8",
      "trigger_name": "test-trigger",
      "finding_ids": [
        "2e167f4b-8063-40ef-80f8-2afd9bf095b8"
      ],
      "related_doc_ids": [
        "1|windows"
      ],
      "state": "ACTIVE",
      "error_message": null,
      "alert_history": [],
      "severity": "1",
      "action_execution_results": [
        {
          "action_id": "BopdoIJKXd",
          "last_execution_time": 1668560817925,
          "throttled_count": 0
        }
      ],
      "start_time": "2022-11-16T01:06:57.748Z",
      "last_notification_time": "2022-11-16T01:06:57.748Z",
      "end_time": null,
      "acknowledged_time": null
    }
  ],
  "failed": [],
  "missing": []
}
```

---
## Get Findings

基于检测器属性的获取结果API。

### Example request

```json
get/_plugins/_security_analytics/findings/_search？
{
    "total_findings"：2，
    "findings"：[[
       {
            "detectorId"："12345"，，，，
            "id"："2b9663f4-ae77-4df8-b84f-688a0195723b"，，，，
            "related_doc_ids"：[[
                "5"
            ]，，
            "index"："sbwhrzgdlg"，，，，
            "queries"：[[
                {
                    "id"："f1bff160-587b-4500-b60c-ab22c7abc652"，，，，
                    "name"："3"，，，，
                    "query"："test_field:\"我们-西方-2 \""，，，，
                    "tags"：[[
                        
                    这是给出的
                }
            ]，，
            "timestamp"：166440108804，
            "document_list"：[[
                {
                    "index"："sbwhrzgdlg"，，，，
                    "id"："5"，，，，
                    "found"：真的，
                    "document"："{\n            \"信息\" : \"这是来自IAD区域的错误\",\n            \"test_strict_date_time \" : \"2022-09-28T21：38：02.888Z \",\n            \"test_field \" : \"我们-西方-2 \"\n        }"
                }
            这是给出的
        }，，
        {
            "detectorId"："12345"，，，，
            "id"："f43a2701-0ef5-4931-8254-bdf510f73952"，，，，
            "related_doc_ids"：[[
                "1"
            ]，，
            "index"："sbwhrzgdlg"，，，，
            "queries"：[[
                {
                    "id"："f1bff160-587b-4500-b60c-ab22c7abc652"，，，，
                    "name"："3"，，，，
                    "query"："test_field:\"我们-西方-2 \""，，，，
                    "tags"：[[
                        
                    这是给出的
                }
            ]，，
            "timestamp"：166440108746，
            "document_list"：[[
                {
                    "index"："sbwhrzgdlg"，，，，
                    "id"："1"，，，，
                    "found"：真的，
                    "document"："{\n            \"信息\" : \"这是来自IAD区域的错误\",\n            \"test_strict_date_time \" : \"2022-09-28T21：38：02.888Z \",\n            \"test_field \" : \"我们-西方-2 \"\n        }"
                }
            这是给出的
        }
    这是给出的
}
```


