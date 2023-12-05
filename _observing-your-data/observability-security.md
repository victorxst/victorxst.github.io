---
layout: default
title: 可观察性安全性
nav_order: 5
has_children: false
redirect_from:
  - /observing-your-data/security/
---

# 可观察性安全性

您可以在OpenSearch中使用具有可观察性的安全插件来限制非-管理用户对特定操作。例如，您可能希望某些用户仅查看可视化，笔记本电脑和其他可观察性对象，而另一些用户可以创建和修改它们。

## 基本权限

安全插件有两个构建-在涵盖大多数可观察性用例的角色中：`observability_full_access` 和`observability_read_access`。对于每个描述，请参阅[预定义的角色]({{site.url}}{{site.baseurl}}/security/access-control/users-roles#predefined-roles)。如果您在OpenSearch仪表板中看不到这些预定义的角色，则可以使用以下命令创建它们：

```json
PUT _plugins/_security/api/roles/observability_read_access
{
  "cluster_permissions": [
    "cluster:admin/opensearch/observability/get"
  ]
}
```

```json
PUT _plugins/_security/api/roles/observability_full_access
{
  "cluster_permissions": [
    "cluster:admin/opensearch/observability/*"
  ]
}
```

如果这些角色无法满足您的需求，请混合并匹配个人可观察性[权限]({{site.url}}{{site.baseurl}}/security/access-control/permissions/) 适合您的用例。例如，`cluster:admin/opensearch/observability/create` 许可使您可以创建可观察到的对象（可视化，操作面板，笔记本等）。

以下是提供可观察性的示例角色：

```json
PUT _plugins/_security/api/roles/observability_permissions
{
  "cluster_permissions": [
    "cluster:admin/opensearch/observability/create",
    "cluster:admin/opensearch/observability/update",
    "cluster:admin/opensearch/observability/delete",
    "cluster:admin/opensearch/observability/get"
    ],
  "index_permissions": [{
    "index_patterns": [".opensearch-observability"],
    "allowed_actions": ["write", "read", "search"]
  }],
  "tenant_permissions": [{
    "tenant_patterns": ["global_tenant"],
    "allowed_actions": ["opensearch_dashboards_all_write"]
  }]
}
```

