---
layout: default
title: 托管索引
nav_order: 3
parent: 索引状态管理
has_children: false
redirect_from: 
 - /im-plugin/ism/managedindices/
---

# 托管索引

你可以使用托管索引操作更改或更新策略。

下表列出了托管索引操作的字段。

参数 | 描述 | 类型 | 必需 | 只读
:--- | :--- |:--- |:--- |
 `name` | 托管索引策略的名称。| `string` | 是 | 不
 `index` | 此策略管理的托管索引的名称。| `string` | 是 | 不
 `index_uuid`  | 索引的 uuid。| `string` | 是 | 不
 `enabled` | 时 `true`，托管索引由调度程序调度和运行。| | `boolean` 是 | 不
 `enabled_time` | 上次启用托管索引的时间。如果禁用了托管索引进程，则此值为 null。| `timestamp` | 是 | 是的
 `last_updated_time` | 上次更新托管索引的时间。| `timestamp` | 是 | 是的
 `schedule` | 托管索引作业的计划。| `object` | 是 | 不
 `policy_id` | 此托管索引使用的策略的名称。| `string` | 是 | 不
 `policy_seq_no` | 此托管索引使用的策略的序列号。| `number` | 是 | 不
 `policy_primary_term` | 此托管索引使用的策略的主要术语。| `number` | 是 | 不
 `policy_version` | 此托管索引使用的策略版本。| `number` | 是 | 是的
 `policy` | 在运行期间使用的策略 `policy_version` 的缓存 JSON。如果策略为 null，则表示这是作业的首次执行，并且读入/保存了最新的策略文档。| `object` | 否 | 不
 `change_policy` | 有关要更改为的策略和状态的信息。| `object` | 否 | 不
 `policy_name` | 要更新到的策略的名称。要更新到最新版本，请将其设置为与当前 `policy_name` 相同。| | `string` 否 | 是的
 `state` | 托管索引完成更新后的状态。如果未指定状态，则假定策略结构未更改。| `string` | 否 | 是的

以下示例显示了托管索引策略：

```json
{
  "managed_index": {
    "name": "my_index",
    "index": "my_index",
    "index_uuid": "sOKSOfkdsoSKeofjIS",
    "enabled": true,
    "enabled_time": 1553112384,
    "last_updated_time": 1553112384,
    "schedule": {
      "interval": {
        "period": 1,
        "unit": "MINUTES",
        "start_time": 1553112384
      }
    },
    "policy_id": "log_rotation",
    "policy_version": 1,
    "policy": {...},
    "change_policy": null
  }
}
```

## 更改策略

你可以更改任何托管索引策略，但 ISM 有一些约束来确保策略更改不会破坏索引。

如果索引停滞在其当前状态，从不继续，并且你希望立即更新其策略，请确保新策略包含与旧策略相同的状态---相同的名称、相同的操作、相同的顺序---。在这种情况下，即使策略正在执行操作，ISM 也会应用新策略。

如果更新策略时未包含相同的状态，则 ISM 仅在当前状态下的所有操作执行完毕后才更新策略。或者，你可以在旧策略中选择特定状态，在此之后，你希望新策略生效。

要使用 OpenSearch 控制面板更改策略，请执行以下操作：

- 在下**索引管理**，选择要将新策略附加到的索引。
- 要将新策略附加到特定状态的索引，请选择**选择状态筛选器**，然后选择这些状态。
- 在下**选择“新建策略”**，选择新策略。
- 要为处于当前状态的索引启动新策略，请选择**策略生效后，将索引保持当前状态**。
- 要在特定状态下启动新策略，请选择**更改策略后从所选状态开始**，然后选择新策略中的默认启动状态。
