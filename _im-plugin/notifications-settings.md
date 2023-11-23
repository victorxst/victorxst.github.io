---
layout: default
title: 通知设置
nav_order: 100
---

# 通知设置

引入 2.8 {：.label .label-purple }

你可以使用通知设置来配置有关长时间运行的索引操作的通知。当长时间运行的索引操作通过[在 OpenSearch 控制面板中使用通知]({{site.url}}{{site.baseurl}}/dashboards/im-dashboards/notifications/) API 完成或通过 API 完成时，自动[通知]({{site.url}}{{site.baseurl}}/observing-your-data/notifications/index/)设置。

配置通知设置对于长时间运行的索引操作（如 `open`、、 `reindex` `resize` 和 `force merge`）非常有用。当你发送这些操作的请求并将参数 `false` 设置为时，该操作会立即返回， `wait_for_completion` 并且响应包含任务 ID。你可以使用该任务 ID 配置此操作的通知。

## 配置通知设置

你可以通过 API `task_id` 使用和 `action_name` 参数配置长时间运行的操作通知：

- **一次性设置**：如果传 `task_id` 入 `lron_config` 对象，则任务将运行一次，并在任务结束时自动删除该设置。如果同时 `task_id` 传递和 `action_name`，则会忽略， `action_name` 但可能对搜索和调试通知设置有用。
- **全局、持久设置**：如果传递 `action_name` 但不在 `task_id` 对象中 `lron_config`，则该任务是全局且持久的，并应用于此操作类型的所有操作。

下表列出了长时间运行的索引操作通知的参数。

| Parameter | Type |描述|
| :--- | :--- | :--- |
| `lron_config` | Object |长时间运行的索引操作通知配置。|
| `task_id` | String |要收到通知的任务的任务 ID。自选。必须指定其中之一 `task_id` `action_name`。|
| `action_name` | String |要通知的操作类型。提供 `action_name` 但不 `task_id` 通知此类型的所有操作。支持的值为 `indices:data/write/reindex`、、 `indices:admin/resize` `indices:admin/forcemerge` 和 `indices:admin/open`。自选。必须指定其中之一 `task_id` `action_name`。|
| `lron_condition` | Object |指定要通知哪些事件。自选。如果未提供，你将收到操作成功和失败的通知。|
| `lron_condition.success` | Boolean |将此参数设置为 `true` 在操作成功时收到通知。自选。缺省值为 `true`。|
| `lron_condition.failure` | Boolean |设置该参数为 `true` 在操作失败或超时时通知。自选。缺省值为 `true`。|
| `channels` | Object |支持的通信渠道包括 Amazon Chime、Amazon Simple Notification Service（Amazon SNS）、Amazon Simple Email Service（Amazon SES）、通过 SMTP 发送电子邮件、Slack 和自定义 Webhook。如果或 `lron_condition.success` `lron_condition.failure` 是 `true`， `channels` 则必须包含至少一个通道。了解如何在中[通知]({{site.url}}{{site.baseurl}}/observing-your-data/notifications/index/)配置通知通道。|

## 创建通知设置

以下示例请求设置重新索引任务失败时的通知：

```json
POST /_plugins/_im/lron
{
  "lron_config": {
      "task_id":"dQlcQ0hQS2mwF-AQ7icCMw:12354",
      "action_name":"indices:data/write/reindex",
      "lron_condition": {
        "success": false,
        "failure": true
      },
      "channels":[
          {"id":"channel1"},
          {"id":"channel2"}
      ]
  }
}
```
{% include copy-curl.html %}

上述请求将产生以下响应：

```json
{
  "_id": "LRON:dQlcQ0hQS2mwF-AQ7icCMw:12354",
  "lron_config": {
    "lron_condition": {
      "success": false,
      "failure": true
    },
    "task_id": "dQlcQ0hQS2mwF-AQ7icCMw:12354",
    "action_name": "indices:data/write/reindex",
    "channels": [
      {
        "id": "channel1"
      },
      {
        "id": "channel2"
      }
    ]
  }
}
```

### 通知设置 ID

响应在字段中返回通知设置 `_id` 的 ID。你可以使用此 ID 读取、更新或删除此通知设置。对于全局 `lron_config`，ID 的格式 `LRON:<action_name>` 为（例如， `LRON:indices:data/write/reindex`）。

可能 `action_name` 包含斜杠字符（ `/`），必须对其进行 HTTP 编码，就像在开发工具控制台中使用它一样 `%2F`。例如， `LRON:indices:data/write/reindex` 变为 `LRON:indices:data%2Fwrite%2Freindex`.{：.important}

对于任务 `lron_config`，ID 的格式 `LRON:<task ID>` 为。

## 检索通知设置

以下示例检索当前配置的通知设置。

使用以下请求检索具有指定[通知设置 ID](#notification-setting-id)：

```json
 GET /_plugins/_im/lron/<lronID>
```
{% include copy-curl.html %}

例如，以下请求检索操作的 `reindex` 通知设置：

```json
{
  "lron_configs": [
    {
      "_id": "LRON:indices:data/write/reindex",
      "lron_config": {
        "lron_condition": {
          "success": false,
          "failure": true
        },
        "action_name": "indices:data/write/reindex",
        "channels": [
          {
            "id": "my_chime"
          }
        ]
      }
    }
  ],
  "total_number": 1
}
```
{% include copy-curl.html %}

使用以下请求检索所有通知设置：

```json
GET /_plugins/_im/lron
```
{% include copy-curl.html %}

响应包含所有已配置的通知设置及其 ID：

```json
{
  "lron_configs": [
    {
      "_id": "LRON:indices:admin/open",
      "lron_config": {
        "lron_condition": {
          "success": false,
          "failure": false
        },
        "action_name": "indices:admin/open",
        "channels": []
      }
    },
    {
      "_id": "LRON:indices:data/write/reindex",
      "lron_config": {
        "lron_condition": {
          "success": false,
          "failure": true
        },
        "action_name": "indices:data/write/reindex",
        "channels": [
          {
            "id": "my_chime"
          }
        ]
      }
    }
  ],
  "total_number": 2
}
```

## 更新通知设置

以下示例使用指定的[通知设置 ID](#notification-setting-id)：

```json
PUT /_plugins/_im/lron/<lronID>
{
  "lron_config": {
      "task_id":"dQlcQ0hQS2mwF-AQ7icCMw:12354",
      "action_name":"indices:data/write/reindex",
      "lron_condition": {
        "success": false,
        "failure": true
      },
      "channels":[
          {"id":"channel1"},
          {"id":"channel2"}
      ]
  }
}
```
{% include copy-curl.html %}

响应包含更新的设置：

```json
{
  "_id": "LRON:dQlcQ0hQS2mwF-AQ7icCMw:12354",
  "lron_config": {
    "lron_condition": {
      "success": false,
      "failure": true
    },
    "task_id": "dQlcQ0hQS2mwF-AQ7icCMw:12354",
    "action_name": "indices:data/write/reindex",
    "channels": [
      {
        "id": "channel1"
      },
      {
        "id": "channel2"
      }
    ]
  }
}
```

## 删除通知设置

以下示例删除具有指定[通知设置 ID](#notification-setting-id)：

```json
DELETE /_plugins/_im/lron/<lronID>
```
{% include copy-curl.html %}

例如，以下请求删除操作 `reindex` 的通知设置：

```json
DELETE _plugins/_im/lron/LRON:indices:data%2Fwrite%2Freindex
```
{% include copy-curl.html %}

## 后续步骤

- 了解有关[ISM API]({{site.url}}{{site.baseurl}}/im-plugin/ism/api/).
- 了解有关该应用程序[通知]({{site.url}}{{site.baseurl}}/observing-your-data/notifications/index/)的更多信息。
