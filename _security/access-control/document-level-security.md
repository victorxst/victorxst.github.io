---
layout: default
title: 文档-级别的安全性
parent: 访问控制
nav_order: 85
redirect_from:
 - /security/access-control/document-level-security/
 - /security-plugin/access-control/document-level-security/
---

# 文档-级别安全性（DLS）

文档-级别的安全性使您可以将角色限制为索引中的文档子集。开始文档的最简单方法- 和字段-级别安全是打开OpenSearch仪表板，然后选择**安全**。然后选择**角色**，创建一个新角色，并审查**指数许可** 部分。

![文档- 和字段-OpenSearch仪表板中的级别安全屏幕]({{site.url}}{{site.baseurl}}/images/security-dls.png)


## 简单的角色

文档-Level Security使用OpenSearch查询DSL来定义哪些记录角色授予访问权限。在OpenSearch仪表板中，选择索引模式并在**文档级别的安全性** 部分：

```json
{
  "bool": {
    "must": {
      "match": {
        "genres": "Comedy"
      }
    }
  }
}
```

此查询指定该角色可以访问文档，`genres` 字段必须包括`Comedy`。

对`_search` API包括`{ "query": { ... } }` 在查询周围，但是在这种情况下，您只需要指定查询本身即可。

在其余API中，您将查询作为字符串提供，因此您必须逃脱引号。此角色允许用户读取字段中任何索引中的任何文档`public` 设置`true`：

```json
PUT _plugins/_security/api/roles/public_data
{
  "cluster_permissions": [
    "*"
  ],
  "index_permissions": [{
    "index_patterns": [
      "pub*"
    ],
    "dls": "{\"term\": { \"public\": true}}",
    "allowed_actions": [
      "read"
    ]
  }]
}
```

这些查询可能会随意进行复杂，但我们建议保持它们简单，以最大程度地减少文档的性能影响-级别安全功能在集群上。
{： 。警告 }

### 关于文本字段中Unicode特殊字符的注释

由于与Unicode特殊字符相关的单词边界，Unicode标准分析仪无法索引[文本字段类型]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/text/) 当包含这些特殊字符之一时，值为整体值。结果，标准分析仪将包含特殊字符的文本字段值解析为由特殊字符隔开的多个值，从而有效地将其两侧的不同元素示意。这可能导致对文档的无意过滤，并可能损害其访问权限的控制。

下面的示例说明了包含特殊字符的值，这些值将由标准分析仪不正确地解析。在此示例中，该值中连字符/减号的存在阻止分析仪区分两个不同的用户`user.id` 并将它们解释为同一：

```json
{
  "bool": {
    "must": {
      "match": {
        "user.id": "User-1"
      }
    }
  }
}
```

```json
{
  "bool": {
    "must": {
      "match": {
        "user.id": "User-2"
      }
    }
  }
}
```

为了避免使用查询DSL或REST API时，您可以使用自定义分析仪或将字段映射为`keyword`，执行确切的-匹配搜索。看[关键字字段类型]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/keyword/) 对于后一个选项。

对于当字段类型为时应避免的字符列表`text`， 看[单词边界](https://unicode.org/reports/tr29/#Word_Boundaries)。


## 参数替代

存在许多变量，您可以根据用户的属性来执行规则。例如，`${user.name}` 被当前用户的名称替换。

该规则允许用户读取任何文档，其中用户名是`readable_by` 场地：

```json
PUT _plugins/_security/api/roles/user_data
{
  "cluster_permissions": [
    "*"
  ],
  "index_permissions": [{
    "index_patterns": [
      "pub*"
    ],
    "dls": "{\"term\": { \"readable_by\": \"${user.name}\"}}",
    "allowed_actions": [
      "read"
    ]
  }]
}
```

该表列出了替换。

学期| 取代
：--- | ：---
`${user.name}` | 用户名。
`${user.roles}` | 逗号-分开，引用用户后端角色列表。
`${user.securityRoles}` | 逗号-分开，引用用户安全角色列表。
`${attr.<TYPE>.<NAME>}` | 名称的属性`<NAME>` 为用户定义。`<TYPE>` 是`internal`，，，，`jwt`，，，，`proxy` 或者`ldap`


## 属性-基于安全性

您可以使用角色和参数替代`terms_set` 查询启用属性-基于安全性。

> 请注意`security_attributes` 索引的类型需要`keyword`。

#### 用户定义

```json
PUT _plugins/_security/api/internalusers/user1
{
  "password": "asdf",
  "backend_roles": ["abac"],
  "attributes": {
    "permissions": "\"att1\", \"att2\", \"att3\""
  }
}
```

#### 角色定义

```json
PUT _plugins/_security/api/roles/abac
{
  "index_permissions": [{
    "index_patterns": [
      "*"
    ],
    "dls": "{\"terms_set\": {\"security_attributes\": {\"terms\": [${attr.internal.permissions}], \"minimum_should_match_script\": {\"source\": \"doc['security_attributes'].length\"}}}}",
    "allowed_actions": [
      "read"
    ]
  }]
}
```
## 使用术语-带DLS的Level Lookup查询（TLQ）

您可以执行术语-带文档的级别查找查询（TLQ）-使用两种模式之一：自适应或过滤器级别。默认模式是自适应的，其中OpenSearch自动在Lucene之间切换-级别或过滤器-级别模式取决于是否有TLQ。Lucene执行了没有TLQ的DLS查询-级别模式，而在过滤器中执行了带有TLQ的DLS查询-级别模式。

默认情况下，安全插件检测到DLS查询是否包含TLQ，并在运行时自动选择适当的模式。

要了解有关OpenSearch查询的更多信息，请参阅[学期-级查询]({{site.url}}{{site.baseurl}}/query-dsl/term/index/)。

### 如何在`opensearch.yml`

默认情况下，DLS评估模式设置为`adaptive`。您也可以明确设置模式`opensearch.yml` 与`plugins.security.dls.mode` 环境。添加一条线`opensearch.yml` 具有所需的评估模式。
例如，要将其设置为过滤级别，请添加以下行：
```
plugins.security.dls.mode: filter-level
```

#### DLS评估模式

| 评估模式| 范围| 描述| 用法|
：--- | ：--- | ：--- | ：--- |
露西恩-DLS级| `lucene-level` | 此设置使所有DLS查询都适用于Lucene级别。| 露西恩-级别DLS直接修改Lucene查询和数据结构。这是最有效的模式，但不允许在包括TLQ在内的DLS查询中某些高级构造。
筛选-DLS级| `filter-level` | 此设置使所有DLS查询都应用于过滤器级别。| 在此模式下，OpenSearch通过修改OpenSearch收到的查询来应用DLS。这允许术语-DLS查询中的Level查找查询，但您只能使用`get`，，，，`search`，，，，`mget`， 和`msearch` 从保护索引中检索数据的操作。另外，交叉-群集搜索在此模式下受到限制。
自适应| `adaptive-level` | 允许OpenSearch自动选择模式的默认设置。| Lucene执行了没有TLQ的DLS查询-级别模式，而在过滤器中执行包含TLQ的DLS查询- 级别模式。

