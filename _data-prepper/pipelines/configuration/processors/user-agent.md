---
layout: default
title: 用户代理
parent: 处理器
grand_parent: 管道
nav_order: 130
---

# 用户代理

这`user_agent` 处理器在事件中解析任何用户代理（UA）字符串，然后将解析结果添加到事件的写入数据中。

## 用法

在此示例中，`user_agent` 处理器调用包含UA字符串的源，`ua` 字段，并指示解析字符串写入的关键，`user_agent`，如以下示例所示：

```yaml
  processor:
    - user_agent:
        source: "ua"
        target: "user_agent"
```

以下示例活动包含`ua` 带有提供有关用户信息的字符串的字段：

```json
{
  "ua":  "Mozilla/5.0 (iPhone; CPU iPhone OS 13_5_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.1.1 Mobile/15E148 Safari/604.1"
}
```

这`user_agent` 处理器将字符串解析为与弹性通用架构（EC）兼容的格式，然后将结果添加到指定的目标中，如以下示例所示：

```json
{
  "user_agent": {
    "original": "Mozilla/5.0 (iPhone; CPU iPhone OS 13_5_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.1.1 Mobile/15E148 Safari/604.1",
    "os": {
        "version": "13.5.1",
        "full": "iOS 13.5.1",
        "name": "iOS"
    },
    "name": "Mobile Safari",
    "version": "13.1.1",
    "device": {
        "name": "iPhone"
    }
  },
  "ua":  "Mozilla/5.0 (iPhone; CPU iPhone OS 13_5_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.1.1 Mobile/15E148 Safari/604.1"
}
```

## 配置选项

您可以使用以下配置选项`user_agent` 处理器。

| 选项| 必需的| 描述|
| ：--- | ：--- | ：--- |
| `source` | 是的| 将被解析的领域。
| `target` | 不| 解析事件将写的字段。默认为`user_agent`。
| `exclude_original` | 不| 确定是否从解析结果中排除原始UA字符串。默认为`false`。
| `cache_size` | 不| 兆字节中解析器的缓存大小。默认为`1000`。|
| `tags_on_parse_failure` | 不| 如果是`user_agent` 处理器无法解析UA字符串。|

