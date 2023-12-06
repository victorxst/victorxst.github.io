---
layout: default
title: ISM API
parent: 索引状态管理
nav_order: 20
---

# ISM API 接口

使用索引状态管理操作以编程方式处理策略和托管索引。

---

#### 目录
- 目录
{:toc}


---


## 创建策略
引入 1.0 {：.label .label-purple }

创建策略。

#### 示例请求

```json
PUT _plugins/_ism/policies/policy_1
{
  "policy": {
    "description": "ingesting logs",
    "default_state": "ingest",
    "states": [
      {
        "name": "ingest",
        "actions": [
          {
            "rollover": {
              "min_doc_count": 5
            }
          }
        ],
        "transitions": [
          {
            "state_name": "search"
          }
        ]
      },
      {
        "name": "search",
        "actions": [],
        "transitions": [
          {
            "state_name": "delete",
            "conditions": {
              "min_index_age": "5m"
            }
          }
        ]
      },
      {
        "name": "delete",
        "actions": [
          {
            "delete": {}
          }
        ],
        "transitions": []
      }
    ]
  }
}
```


#### 响应示例

```json
{
  "_id": "policy_1",
  "_version": 1,
  "_primary_term": 1,
  "_seq_no": 7,
  "policy": {
    "policy": {
      "policy_id": "policy_1",
      "description": "ingesting logs",
      "last_updated_time": 1577990761311,
      "schema_version": 1,
      "error_notification": null,
      "default_state": "ingest",
      "states": [
        {
          "name": "ingest",
          "actions": [
            {
              "rollover": {
                "min_doc_count": 5
              }
            }
          ],
          "transitions": [
            {
              "state_name": "search"
            }
          ]
        },
        {
          "name": "search",
          "actions": [],
          "transitions": [
            {
              "state_name": "delete",
              "conditions": {
                "min_index_age": "5m"
              }
            }
          ]
        },
        {
          "name": "delete",
          "actions": [
            {
              "delete": {}
            }
          ],
          "transitions": []
        }
      ]
    }
  }
}
```


---

## 添加策略
引入 1.0 {：.label .label-purple }

将策略添加到索引。如果索引已有一个策略，则此操作不会更改策略。

#### 示例请求

```json
POST _plugins/_ism/add/index_1
{
  "policy_id": "policy_1"
}
```

#### 响应示例

```json
{
  "updated_indices": 1,
  "failures": false,
  "failed_indices": []
}
```

如果在向索引添加策略时使用通配符 `*`，则 ISM 插件将 `*` 解释为所有索引，包括存储用户、角色和租户的系统索引，例如 `.opendistro-security`。策略中的删除操作可能会意外删除群集中的所有用户角色和租户。不要使用宽 `*` 泛通配符，而是在使用 `_ism/add` API 指定索引时添加前缀，例如 `my-logs*`。{: .warning}

---


## 更新策略
引入 1.0 {：.label .label-purple }

更新策略。 `seq_no` 使用和 `primary_term` 参数更新现有策略。如果这些数字与现有策略不匹配或策略不存在，则 ISM 将引发错误。

当前应用于索引的策略可能不是可用的最新策略。要查看当前应用于索引的策略，请参阅[解释索引]({{site.url}}{{site.baseurl}}/im-plugin/ism/api/#explain-index)。要获取策略的最新版本，请参阅[Get policy]({{site.url}}{{site.baseurl}}/im-plugin/ism/api/#get-policy)。

#### 示例请求

```json
PUT _plugins/_ism/policies/policy_1?if_seq_no=7&if_primary_term=1
{
  "policy": {
    "description": "ingesting logs",
    "default_state": "ingest",
    "states": [
      {
        "name": "ingest",
        "actions": [
          {
            "rollover": {
              "min_doc_count": 5
            }
          }
        ],
        "transitions": [
          {
            "state_name": "search"
          }
        ]
      },
      {
        "name": "search",
        "actions": [],
        "transitions": [
          {
            "state_name": "delete",
            "conditions": {
              "min_index_age": "5m"
            }
          }
        ]
      },
      {
        "name": "delete",
        "actions": [
          {
            "delete": {}
          }
        ],
        "transitions": []
      }
    ]
  }
}
```


#### 响应示例

```json
{
  "_id": "policy_1",
  "_version": 2,
  "_primary_term": 1,
  "_seq_no": 10,
  "policy": {
    "policy": {
      "policy_id": "policy_1",
      "description": "ingesting logs",
      "last_updated_time": 1577990934044,
      "schema_version": 1,
      "error_notification": null,
      "default_state": "ingest",
      "states": [
        {
          "name": "ingest",
          "actions": [
            {
              "rollover": {
                "min_doc_count": 5
              }
            }
          ],
          "transitions": [
            {
              "state_name": "search"
            }
          ]
        },
        {
          "name": "search",
          "actions": [],
          "transitions": [
            {
              "state_name": "delete",
              "conditions": {
                "min_index_age": "5m"
              }
            }
          ]
        },
        {
          "name": "delete",
          "actions": [
            {
              "delete": {}
            }
          ],
          "transitions": []
        }
      ]
    }
  }
}
```


---

## 获取策略
引入 1.0 {：.label .label-purple }

通过 `policy_id` 获取策略。

#### 示例请求

```json
GET _plugins/_ism/policies/policy_1
```


#### 响应示例

```json
{
  "_id": "policy_1",
  "_version": 2,
  "_seq_no": 10,
  "_primary_term": 1,
  "policy": {
    "policy_id": "policy_1",
    "description": "ingesting logs",
    "last_updated_time": 1577990934044,
    "schema_version": 1,
    "error_notification": null,
    "default_state": "ingest",
    "states": [
      {
        "name": "ingest",
        "actions": [
          {
            "rollover": {
              "min_doc_count": 5
            }
          }
        ],
        "transitions": [
          {
            "state_name": "search"
          }
        ]
      },
      {
        "name": "search",
        "actions": [],
        "transitions": [
          {
            "state_name": "delete",
            "conditions": {
              "min_index_age": "5m"
            }
          }
        ]
      },
      {
        "name": "delete",
        "actions": [
          {
            "delete": {}
          }
        ],
        "transitions": []
      }
    ]
  }
}
```

---

## 从索引中删除策略
引入 1.0 {：.label .label-purple }

从索引中删除任何 ISM 策略。

#### 示例请求

```json
POST _plugins/_ism/remove/index_1
```


#### 响应示例

```json
{
  "updated_indices": 1,
  "failures": false,
  "failed_indices": []
}
```

---

## 更新托管索引策略
引入 1.0 {：.label .label-purple }

将托管索引策略更新为新策略（或策略的新版本）。你可以使用索引模式一次更新多个索引。更新多个索引时，你可能希望包含状态筛选器以仅影响某些托管索引。更改策略会筛选出所有现有的托管索引，并仅将更改应用于处于指定状态的索引。你还可以显式指定托管索引在更改策略生效后转换到的状态。

策略更改是一个异步后台进程。更改已排队，后台进程不会立即执行。这种执行延迟可防止当前正在运行的托管索引进入中断状态。如果要更改到的策略只有一些小的配置更改，则更改会立即发生。例如，如果策略将 `min_index_age` 滚动更新条件中的参数从 `1000d` 到 `100d` 更改为，则此更改将在下次执行时立即发生。如果更改修改了索引所处当前状态的状态、操作或操作顺序，则更改将在其当前状态结束时发生，然后再转换为新状态。

在此示例中，应用于索引的策略更改为 `policy_1`，它可以是全新的策略，也可以是其现有策略的 `index_1` 更新版本。仅当索引当前处于该 `searches` 状态时，该过程才应用更改。在策略发生此更改后， `index_1` 将 `delete` 转换为状态。

#### 示例请求

```json
POST _plugins/_ism/change_policy/index_1
{
  "policy_id": "policy_1",
  "state": "delete",
  "include": [
    {
      "state": "searches"
    }
  ]
}
```


#### 响应示例

```json
{
  "updated_indices": 0,
  "failures": false,
  "failed_indices": []
}
```

---

## 重试失败的索引
引入 1.0 {：.label .label-purple }

重试索引的失败操作。要使重试调用成功，ISM 必须管理索引，并且索引必须处于失败状态。你可以使用索引模式（ `*`）重试多个失败的索引。

#### 示例请求

```json
POST _plugins/_ism/retry/index_1
{
  "state": "delete"
}
```


#### 响应示例

```json
{
  "updated_indices": 0,
  "failures": false,
  "failed_indices": []
}
```

---

## 解释索引
引入 1.0 {：.label .label-purple }

获取索引的当前状态。你可以使用索引模式来获取多个索引的状态。

#### 示例请求

```json
GET _plugins/_ism/explain/index_1
```


#### 响应示例

```json
{
  "index_1": {
    "index.plugins.index_state_management.policy_id": "policy_1"
  }
}
```

或者，你可以将参数 `show_policy` 添加到请求的路径中，以获取当前应用于索引的策略，这对于查看应用于索引的策略是否是最新的策略非常有用。要获取最新的策略，请参阅[获取策略 API]({{site.url}}{{site.baseurl}}/im-plugin/ism/api/#get-policy)。

#### 示例请求

```json
GET _plugins/_ism/explain/index_1?show_policy=true
```

#### 响应示例

```json
{
  "index_1": {
    "index.plugins.index_state_management.policy_id": "sample-policy",
    "index.opendistro.index_state_management.policy_id": "sample-policy",
    "index": "index_1",
    "index_uuid": "gCFlS_zcTdih8xyxf3jQ-A",
    "policy_id": "sample-policy",
    "enabled": true,
    "policy": {
      "policy_id": "sample-policy",
      "description": "ingesting logs",
      "last_updated_time": 1647284980148,
      "schema_version": 13,
      "error_notification": null,
      "default_state": "ingest",
      "states": [...],
      "ism_template": null
    }
  },
  "total_managed_indices": 1
}
```

从 ODFE 版本 1.13.0 开始，该 `plugins.index_state_management.policy_id` 设置已弃用。为了保持一致性，我们在响应 API 中保留此字段。

---

## 删除策略
引入 1.0 {：.label .label-purple }

按 `policy_id` 删除策略。

#### 示例请求

```json
DELETE _plugins/_ism/policies/policy_1
```


#### 响应示例

```json
{
  "_index": ".opendistro-ism-config",
  "_id": "policy_1",
  "_version": 3,
  "result": "deleted",
  "forced_refresh": true,
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "_seq_no": 15,
  "_primary_term": 1
}
```

## 防错验证
引入 2.4 {：.label .label-purple }

ISM 允许你自动运行操作。但是，由于各种原因，运行操作可能会失败。可以使用错误防护验证来测试操作，以排除失败。

若要启用错误防护验证，请将 `plugins.index_state_management.validation_service.enabled` 设置设置为 `true`：

```bash
PUT _cluster/settings
{
   "persistent":{
      "plugins.index_state_management.validation_action.enabled": true
   }
}
```

#### 响应示例

```json
{
  "acknowledged" : true,
  "persistent" : {
    "plugins" : {
      "index_state_management" : {
        "validation_action" : {
          "enabled" : "true"
        }
      }
    }
  },
  "transient" : { }
}
```

若要检查防错验证状态和消息，请传递 `validate_action=true` 到 `_plugins/_ism/explain` 终结点：

```bash
GET _plugins/_ism/explain/test-000001?validate_action=true
```

#### 响应示例

响应包含一个附加的验证对象，其中包含验证消息和状态：

```json
{
  "test-000001" : {
    "index.plugins.index_state_management.policy_id" : "test_rollover",
    "index.opendistro.index_state_management.policy_id" : "test_rollover",
    "index" : "test-000001",
    "index_uuid" : "CgKsxFmQSIa8dWqpbSJmyA",
    "policy_id" : "test_rollover",
    "policy_seq_no" : -2,
    "policy_primary_term" : 0,
    "rolled_over" : false,
    "index_creation_date" : 1667410460649,
    "state" : {
      "name" : "rollover",
      "start_time" : 1667410766045
    },
    "action" : {
      "name" : "rollover",
      "start_time" : 1667411127803,
      "index" : 0,
      "failed" : false,
      "consumed_retries" : 0,
      "last_retry_time" : 0
    },
    "step" : {
      "name" : "attempt_rollover",
      "start_time" : 1667411127803,
      "step_status" : "starting"
    },
    "retry_info" : {
      "failed" : true,
      "consumed_retries" : 0
    },
    "info" : {
      "message" : "Previous action was not able to update IndexMetaData."
    },
    "enabled" : false,
    "validate" : {
      "validation_message" : "Missing rollover_alias index setting [index=test-000001]",
      "validation_status" : "re_validating"
    }
  },
  "total_managed_indices" : 1
}
```

如果将值传递或未传递 `validate_action` 到 `_plugins/_ism/explain` 终结点，则响应将 `validate_action=false` 不包含防错验证状态和消息：

```bash
GET _plugins/_ism/explain/test-000001?validate_action=false
```

或：

```bash
GET _plugins/_ism/explain/test-000001
```

#### 响应示例

```json
{
  "test-000001" : {
    "index.plugins.index_state_management.policy_id" : "test_rollover",
    "index.opendistro.index_state_management.policy_id" : "test_rollover",
    "index" : "test-000001",
    "index_uuid" : "CgKsxFmQSIa8dWqpbSJmyA",
    "policy_id" : "test_rollover",
    "policy_seq_no" : -2,
    "policy_primary_term" : 0,
    "rolled_over" : false,
    "index_creation_date" : 1667410460649,
    "state" : {
      "name" : "rollover",
      "start_time" : 1667410766045
    },
    "action" : {
      "name" : "rollover",
      "start_time" : 1667411127803,
      "index" : 0,
      "failed" : false,
      "consumed_retries" : 0,
      "last_retry_time" : 0
    },
    "step" : {
      "name" : "attempt_rollover",
      "start_time" : 1667411127803,
      "step_status" : "starting"
    },
    "retry_info" : {
      "failed" : true,
      "consumed_retries" : 0
    },
    "info" : {
      "message" : "Previous action was not able to update IndexMetaData."
    },
    "enabled" : false
  },
  "total_managed_indices" : 1
}
```