---
layout: default
title: API速率限制
parent: 配置
nav_order: 30
---


# API速率限制

API速率限制通常用于限制用户在设定的时间内可以进行的API调用数量，从而有助于管理API流量的速率。出于安全目的，通过限制登录失败的尝试来限制限制功能，有可能防御拒绝服务（DOS）攻击或重复登录尝试，旨在通过反复试验和错误获得访问权限。

您可以选择为用户名限制，IP地址率限制或两者配置安全插件。这些配置是在`config.yml` 文件。有关每种类型的速率限制配置的信息，请参见以下各节。


## 用户名率限制

用户名限制配置限制了用户名的登录尝试。当登录失败时，用户名被网络中的任何机器都阻止。以下示例显示`config.yml` 为用户名率限制配置的文件设置：

```yml
auth_failure_listeners:
      internal_authentication_backend_limiting:
        type: username
        authentication_backend: internal
        allowed_tries: 3
        time_window_seconds: 60
        block_expiry_seconds: 60
        max_blocked_clients: 100000
        max_tracked_clients: 100000
```
{％include copy.html％}

下表描述了此类配置的各个设置。

| 环境| 描述|
| ：--- | ：--- |
| `type` | 利率限制的类型。在这种情况下，`username`。|
| `authentication_backend` | 内部后端。进入`internal`。|
| `allowed_tries` | 在阻止登录尝试之前允许的登录尝试数。请注意，增加数量会增加用法。|
| `time_window_seconds` | 时间窗口的价值`allowed_tries` 被执行。例如，如果`allowed_tries` 是`3` 和`time_window_seconds` 是`60`，一个用户名有3次尝试在60个中成功登录-登录尝试之前的第二个时间跨度被阻止。|
| `block_expiry_seconds` | 登录失败后，登录尝试保持阻塞的时间窗口。在这段时间之后，登录被重置，用户名可以尝试再次登录。|
| `max_blocked_clients` | 最大阻止用户名的数量。这限制了堆的用法，以避免潜在的DOS攻击。|
| `max_tracked_clients` | 登录尝试失败的跟踪用户名的最大数量。这限制了堆的用法，以避免潜在的DOS攻击。|


## IP地址率限制

IP地址限制配置限制IP地址登录尝试。当登录失败时，特定于用于登录的机器的IP地址将被阻止。

配置IP地址限制涉及两个步骤。首先，设置`challenge` 设置为`false` 在里面`http_authenticator` 部分`config.yml` 文件：

```yml
http_authenticator:
  type: basic
  challenge: false
```

有关此设置的更多信息，请参阅[HTTP基本身份验证]({{site.url}}{{site.baseurl}}/security/authentication-backends/basic-authc/)。

其次，配置IP地址速率限制设置。以下示例显示了完整的配置：

```yml
auth_failure_listeners:
      ip_rate_limiting:
        type: ip
        allowed_tries: 1
        time_window_seconds: 20
        block_expiry_seconds: 180
        max_blocked_clients: 100000
        max_tracked_clients: 100000
```
{％include copy.html％}

下表描述了此类配置的各个设置。

| 环境| 描述|
| ：--- | ：--- |
| `type` | 利率限制的类型。在这种情况下，`ip`。|
| `allowed_tries` | 在阻止登录尝试之前允许的登录尝试数。请注意，增加数量会增加用法。|
| `time_window_seconds` | 时间窗口的价值`allowed_tries` 被执行。例如，如果`allowed_tries` 是`3` 和`time_window_seconds` 是`60`，IP地址有3次尝试在60中成功登录-登录尝试之前的第二个时间跨度被阻止。|
| `block_expiry_seconds` | 登录失败后，登录尝试保持阻塞的时间窗口。在此时间之后，重置登录，IP地址可以尝试再次登录。|
| `max_blocked_clients` | 最大阻止IP地址的数量。这限制了堆的用法，以避免潜在的DOS攻击。|
| `max_tracked_clients` | 登录尝试失败的最大跟踪IP地址数量。这限制了堆的用法，以避免潜在的DOS攻击。|


