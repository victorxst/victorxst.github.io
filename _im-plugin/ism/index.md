---
layout: default
title: 索引状态管理
nav_order: 16
has_children: true
redirect_from:
  - /im-plugin/ism/
has_toc: false
---

# 索引状态管理

如果分析时序数据，则可能会优先考虑新数据而不是旧数据。你可以定期对较旧的索引执行某些操作，例如减少副本计数或删除它们。

索引状态管理（ISM）是一个插件，它允许你根据索引期限、索引大小或文档数量的变化触发这些定期管理操作，从而自动执行这些操作。使用 ISM 插件，你可以定义*政策*自动处理索引滚动更新或删除以适合你的用例。

例如，你可以定义一个策略，在 30 天后将索引移动到某个 `read_only` 状态，然后在设定的 90 天后将其删除。你还可以将策略设置为在删除索引时向你发送通知消息。

你可能希望在一定时间后执行索引滚动更新，或者在非高峰时段对索引运行操作 `force_merge`，以提高高峰时段的搜索性能。

要使用 ISM 插件，需要将你的用户角色映射到 `all_access` 授予你对群集的完全访问权限的角色。要了解更多信息，请参阅[用户和角色]({{site.url}}{{site.baseurl}}/security/access-control/users-roles/)。
{:.note}

## ISM 入门

要开始使用，请选择**索引管理** In OpenSearch Dashboards。

### 步骤 1：设置策略

策略是一组规则，用于描述应如何管理索引。有关创建策略的信息，请参阅[Policies]({{site.url}}{{site.baseurl}}/im-plugin/ism/policies/)。

你可以使用可视化编辑器或 JSON 编辑器创建策略。与 JSON 编辑器相比，可视化编辑器通过将流程分为创建错误通知、定义 ISM 模板和添加状态，提供了一种更结构化的策略定义方式。如果要查看预定义的字段，例如可以分配给状态的操作，或者状态可以在什么条件下转换为目标状态，建议使用可视化编辑器。

#### 可视化编辑器

1. 选择选项卡**索引策略**。
2. 选择**创建策略**。
3. 选择**可视化编辑器**。
4. 在**政策信息**该部分中，输入策略 ID 和可选描述。
5. 在本节中**错误通知**，设置一个可选的错误通知，每当策略执行失败时都会发送该通知。有关详细信息，请参阅[错误通知]({{site.url}}{{site.baseurl}}/im-plugin/ism/policies#error-notifications)。如果你在策略中使用自动滚动更新，我们建议你设置错误通知，以便在滚动更新失败时通知你意外较大的索引。
6. 在中**ISM 模板**，输入任何 ISM 模板模式以自动将此策略应用于将来的索引。例如，如果指定的模板 `sample-index*`，则 ISM 插件会自动将此策略应用于名称以开头 `sample-index` 的任何索引。你的模式不能包含以下任何字符： `:`、、、 `/` `+`、 `?` `\` `<` `"` `|` `#` `>` 和。
7. 在中**国家**，添加要包含在策略中的任何状态。每个状态都有[actions]({{site.url}}{{site.baseurl}}/im-plugin/ism/policies/#actions)插件在索引进入特定状态时执行，并且[转换]({{site.url}}{{site.baseurl}}/im-plugin/ism/policies/#transitions)，这些状态具有满足时将索引转换为目标状态的条件。你在策略中创建的第一个状态会自动设置为初始状态。每个策略必须至少有一个状态，但操作和转换是可选的。
8. 选择**创造**。


#### JSON 编辑器

1. 选择选项卡**索引策略**。
2. 选择**创建策略**。
3. 选择**JSON 编辑器**。
4. 在该部分中**名称策略**，输入策略 ID。
5. 在该**定义策略**部分中，输入你的策略。
6. 选择**创造**。

创建策略后，下一步是将其附加到一个或多个索引。你可以在策略中设置一个 `ism_template`，以便在创建与 ISM 模板模式匹配的索引时，插件会自动将该策略附加到索引。

以下示例演示如何创建一个策略，该策略会自动附加到名称以开头 `index_name-` 的所有索引。

```json
PUT _plugins/_ism/policies/policy_id
{
  "policy": {
    "description": "Example policy.",
    "default_state": "...",
    "states": [...],
    "ism_template": {
      "index_patterns": ["index_name-*"],
      "priority": 100
    }
  }
}
```

如果有多个模板与索引模式匹配，ISM 将使用优先级值来确定要应用的模板。

有关 ISM 模板策略的示例，请参见[带有 ISM 模板的自动滚动更新策略示例]({{site.url}}{{site.baseurl}}/im-plugin/ism/policies#sample-policy-with-ism-template-for-auto-rollover)。

旧版本的插件包括 `policy_id` 在索引模板中，因此当创建与索引模板模式匹配的索引时，索引将附加策略：

```json
PUT _index_template/<template_name>
{
  "index_patterns": [
    "index_name-*"
  ],
  "template": {
    "settings": {
      "opendistro.index_state_management.policy_id": "policy_id"
    }
  }
}
```

该 `opendistro.index_state_management.policy_id` 设置已弃用。你可以继续使用 ISM 模板字段自动管理新创建的索引。
{:.note}

### 步骤 2：将策略附加到索引

1. 选择**指标**。
2. 选择要将策略附加到的一个或多个索引。
3. 选择**应用策略**。
4. **策略 ID**从菜单中，选择你创建的策略。
你可以查看策略的预览。
5. 如果你的策略包含滚动更新操作，请指定滚动更新别名。
确保你输入的别名已存在。有关滚动更新操作的详细信息，请参阅[rollover]({{site.url}}{{site.baseurl}}/im-plugin/ism/policies#rollover)。
6. 选择**应用**。

将策略附加到索引后，ISM 会创建一个作业，默认情况下每 5 分钟运行一次，以执行策略操作、检查条件以及将索引转换为不同的状态。要更改此作业的默认时间间隔，请参见[Settings]({{site.url}}{{site.baseurl}}/im-plugin/ism/settings/)。

如果群集状态为红色，则 ISM 不会运行作业。

### 步骤 3：管理索引

1. 选择**托管索引**。
2. 要更改策略，请参阅[变更政策]({{site.url}}{{site.baseurl}}/im-plugin/ism/managedindexes#change-policy)。
3. 要将滚动更新别名附加到索引，请选择你的策略，然后选择**添加滚动更新别名**。
确保你输入的别名已存在。有关滚动更新操作的详细信息，请参阅[rollover]({{site.url}}{{site.baseurl}}/im-plugin/ism/policies#rollover)。
4. 要删除策略，请选择你的策略，然后选择**删除策略**。
5. 要重试策略，请选择你的策略，然后选择。**重试策略**

有关管理策略的信息，请参阅[托管索引]({{site.url}}{{site.baseurl}}/im-plugin/ism/managedindexes/)。
