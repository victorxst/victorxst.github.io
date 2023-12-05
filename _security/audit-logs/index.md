---
layout: default
title: 审核日志
nav_order: 125
has_children: true
has_toc: false
redirect_from:
  - /security/audit-logs/index/
  - /security-plugin/audit-logs/index/
---

# 审核日志

---

<详细信息关闭的markdown ="block">
  <summary>
    目录
  </summary>
  {: .text-delta }
- TOC
{:toc}
</delect>

---

审核日志可让您跟踪对OpenSearch集群的访问，并且可用于合规性或安全漏洞后。您可以配置要记录的类别，已记录消息的详细级别以及存储日志的位置。

启用审核记录：

1. 将以下行添加到`opensearch.yml` 在每个节点上：

   ```yml
   plugins.security.audit.type: internal_opensearch
   ```

   此设置将审核日志存储在当前群集上。有关其他存储选项，请参阅[审核日志存储类型]({{site.url}}{{site.baseurl}}/security/audit-logs/storage-types/)。

2. 重新启动每个节点。

在此初始设置之后，您可以使用OpenSearch仪表板来管理您的审核日志类别和其他设置。在OpenSearch仪表板中，选择**安全** 进而**审核日志**。

另一种选择是指定初始设置以审核`audit.yml` 和`opensearch.yml` 文件（哪个文件取决于设置---看[审核日志设置](#audit-log-settings)）。此后，您可以使用仪表板或[审核日志]({{site.url}}{{site.baseurl}}/security/access-control/api/#audit-logs) API管理和更新设置。


## 追踪事件

审核记录记录以两种方式记录事件：http请求（休息）和传输层。下表提供了跟踪事件的描述，以及它们是否在其余部分或运输层登录。

事件| 登录休息| 登录运输| 描述
:--- | :--- | :--- | :---
`FAILED_LOGIN` | 是的| 是的| 无法验证请求的凭据，这很可能是因为用户不存在或密码不正确。
`AUTHENTICATED` | 是的| 是的| 用户成功身份验证。
`MISSING_PRIVILEGES` | 不| 是的| 用户没有提出请求的要求的权限。
`GRANTED_PRIVILEGES` | 不| 是的| 用户成功地请求进行搜索。
`SSL_EXCEPTION` | 是的| 是的| 试图在没有有效的SSL/TLS证书的情况下访问Opensearch。
`opensearch_SECURITY_INDEX_ATTEMPT` | 不| 是的| 试图修改安全插件内部用户和特权索引，而无需权限或TLS管理员证书。
`BAD_HEADERS` | 是的| 是的| 试图欺骗要求使用安全插件内部标头进行搜索的请求。


## 审核日志设置

以下默认日志设置在大多数用例中都可以正常工作。但是，您可以更改设置以节省存储空间或根据您的确切需求调整信息。


### 审核中的设置

以下设置存储在`audit.yml` 文件。


#### 排除类别

要排除类别，请在以下设置中列出它们：

```yml
plugins.security.audit.config.disabled_rest_categories: <disabled categories>
plugins.security.audit.config.disabled_transport_categories: <disabled categories>
```

例如：

```yml
plugins.security.audit.config.disabled_rest_categories: AUTHENTICATED, opensearch_SECURITY_INDEX_ATTEMPT
plugins.security.audit.config.disabled_transport_categories: GRANTED_PRIVILEGES
```

如果要在所有类别中记录事件，请使用`NONE`：

```yml
plugins.security.audit.config.disabled_rest_categories: NONE
plugins.security.audit.config.disabled_transport_categories: NONE
```


#### 禁用休息或运输层

默认情况下，安全插件在REST和运输层上都记录事件。您可以禁用任何一种类型：

```yml
plugins.security.audit.config.enable_rest: false
plugins.security.audit.config.enable_transport: false
```


#### 禁用请求身体记录

默认情况下，安全插件包括REST和运输层的请求正文（如果有）。如果您不需要或需要请求主体，则可以将其禁用：

```yml
plugins.security.audit.config.log_request_body: false
```


#### 日志索引名称

默认情况下，安全插件记录所有受请求影响的索引。由于索引名称可以是别名并且包含通配符/日期模式，因此安全插件记录了用户提交的 *和 *其解决的实际索引名称的索引名称。

例如，如果您使用别名或通配符，审计事件可能看起来像：

```json
audit_trace_indices: [
  "human*"
],
audit_trace_resolved_indices: [
  "humanresources"
]
```

您可以通过设置禁用此功能：

```yml
plugins.security.audit.config.resolve_indices: false
```

此功能仅在`plugins.security.audit.config.log_request_body` 也设置为`false`。
{: .note }


#### 配置批量请求处理

批量请求可以包含许多索引操作。默认情况下，安全插件仅记录单个批量请求，而不是每个单独的操作。

可以将安全插件配置为将每个索引操作记录为一个单独的事件：

```yml
plugins.security.audit.config.resolve_bulk_requests: true
```

此更改可以在审核日志中创建大量的事件，因此，如果您经常使用该设置，我们不建议启用此设置`_bulk` API。


#### 排除请求

您可以通过为运输请求和/或HTTP请求路径配置操作来将某些请求排除在记录中（ret）：

```yml
plugins.security.audit.config.ignore_requests: ["indices:data/read/*", "SearchRequest"]
```


#### 排除用户

默认情况下，安全插件登录所有用户的事件，但不包括内部OpenSearch仪表板服务器用户`kibanaserver`。您可以排除其他用户：

```yml
plugins.security.audit.config.ignore_users:
  - kibanaserver
  - admin
```

如果应记录所有用户的请求，请使用`NONE`：

```yml
plugins.security.audit.config.ignore_users: NONE
```


#### 排除标题

您可以将敏感的标头排除在日志中---例如，`Authorization:` 标头：

```yml
plugins.security.audit.config.exclude_sensitive_headers: true
```


### opensearch.yml中的设置

以下设置存储在`opensearch.yml` 文件。


#### 配置审核日志索引名称

默认情况下，安全插件将审计事件存储在名为的每日滚动索引中`auditlog-YYYY.MM.dd`：

```yml
plugins.security.audit.config.index: myauditlogindex
```

使用索引名称中的日期模式每天配置，每周或每月滚动索引：

```yml
plugins.security.audit.config.index: "'auditlog-'YYYY.MM.dd"
```

有关日期模式格式的参考，请参见[Joda DateTimeFormat文档](https://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)。


#### （高级）调整线程池

搜索插件异步地记录事件，从而最大程度地减少了对群集的性能影响。该插件使用固定的线程池进行日志事件：

```yml
plugins.security.audit.config.threadpool.size: <integer>
```

默认设置为`10`。将此值设置为`0` 禁用线程池，这意味着插件同步记录事件。要设置每个线程的最大队列长度：

```yml
plugins.security.audit.config.threadpool.max_queue_len: 100000
```


