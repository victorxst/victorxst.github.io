---
layout: default
title: 动作
nav_order: 50
grand_parent: 警报
parent: 监视器
---

# 动作

当满足触发条件时，操作发送通知。看[通知]({{site.url}}{{site.baseurl}}/notifications-plugin/index/) 了解创建通知。如果您不想接收通知，请不要在触发器中添加操作。

## 添加动作

添加一个动作：

1. 在里面**触发器** 面板，选择**添加动作**。
1. 输入动作详细信息，包括操作名称，通知渠道和通知消息主体，**通知** 部分。

    您可以使用[胡子模板](https://mustache.github.io/mustache.5.html)。您可以访问`ctx.action.name`，当前动作的名称，以及所有[动作变量](#actions-variables)。

    如果您的通知频道是一种自定义的网络漫画，期望特定数据格式，请在消息正文中直接包含JSON（或XML）：

    ```json
    {% raw %}{ "text": "Monitor {{ctx.monitor.name}} just entered alert status. Please investigate the issue. - Trigger: {{ctx.trigger.name}} - Severity: {{ctx.trigger.severity}} - Period start: {{ctx.periodStart}} - Period end: {{ctx.periodEnd}}" }{% endraw %}
    ```

    In the preceding example, the message content must conform to the `内容-类型` 标题在[自定义Webhook]({{site.url}}{{site.baseurl}}/notifications-plugin/index/)。

1. 如果您正在使用水桶-级别的监视器，选择显示器是否应为每个执行或每个警报执行诉讼。
1. （可选）使用操作节流以限制您在给定时间范围内收到的通知数量。

    例如，如果监视器每分钟检查触发条件，则每分钟可能会收到一个通知。如果将动作节点设置为60分钟，即使在那个小时内触发了数十次触发条件，也不会收到每小时不超过一个通知。

1. 选择**创造**。

操作发送消息后，该消息的内容留下了[安全分析]({{site.url}}{{site.baseurl}}/security-analytics/index/) 插入。确保访问消息（例如，访问Slack频道）是您的责任。

#### 示例消息

```mustache
{% raw %}Monitor {{ctx.monitor.name}} just entered an alert state. Please investigate the issue.
- Trigger: {{ctx.trigger.name}}
- Severity: {{ctx.trigger.severity}}
- Period start: {{ctx.periodStart}}
- Period end: {{ctx.periodEnd}}{% endraw %}
```

使用`ctx.results` 消息中的变量，使用`{% raw %}{{ctx.results.0}}{% endraw %}` 而不是`{% raw %}{{ctx.results[0]}}{% endraw %}`。这种差异是由于胡须如何处理括号符号。
{:.note}

#### 动作变量

多变的| 数据类型| 描述
:--- | :--- | :---
`ctx.trigger.actions.id` | 细绳| 动作ID。
`ctx.trigger.actions.name` | 细绳| 动作名称。
`ctx.trigger.actions.message_template.source` | 细绳| 发送警报的消息。
`ctx.trigger.actions.message_template.lang` | 细绳| 用于定义消息的脚本语言。必须是胡子。
`ctx.trigger.actions.throttle_enabled` | 布尔| 是否启用了此触发器的节流。看[添加动作](#adding-actions) 有关节流的更多信息。
`ctx.trigger.actions.subject_template.source` | 细绳| 该消息的主题在警报中。
`ctx.trigger.actions.subject_template.lang` | 细绳| 用于定义主题的脚本语言。必须是胡子。

