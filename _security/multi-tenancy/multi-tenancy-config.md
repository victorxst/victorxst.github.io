---
layout: default
title: 多租户配置
parent: OpenSearch 仪表板多租户
nav_order: 145
---


# 多-租赁配置

多-默认情况下启用租赁，但是您可以将其禁用或使用`config/opensearch-security/config.yml`：

```yml
config:
  dynamic:
    kibana:
      multitenancy_enabled: true
      private_tenant_enabled: true
      default_tenant: global tenant
      server_username: kibanaserver
      index: '.kibana'
    do_not_fail_on_forbidden: false
```

| 环境| 描述|
| :--- | :--- |
| `multitenancy_enabled` | 启用或禁用多人-租赁。默认为`true`。|
| `private_tenant_enabled` | 启用或禁用私人租户。默认为`true`。|
| `default_tenant` | 用于设置用户登录时可用的租户。|
| `server_username` | 必须从`opensearch_dashboards.yml`。默认为`kibanaserver`。|
| `index` | 必须匹配OpenSearch仪表板索引的名称`opensearch_dashboards.yml`。默认为`.kibana`。|
| `do_not_fail_on_forbidden` | 什么时候`true`，安全插件可删除不允许用户从搜索结果中看到的任何内容。什么时候`false`，插件返回安全例外。默认为`false`。|

这`opensearch_dashboards.yml` 文件包括其他设置：

```yml
opensearch.username: kibanaserver
opensearch.password: kibanaserver
opensearch.requestHeadersAllowlist: ["securitytenant","Authorization"]
opensearch_security.multitenancy.enabled: true
opensearch_security.multitenancy.tenants.enable_global: true
opensearch_security.multitenancy.tenants.enable_private: true
opensearch_security.multitenancy.tenants.preferred: ["Private", "Global"]
opensearch_security.multitenancy.enable_filter: false
```

| 环境| 描述|
| :--- | :--- |
| `opensearch.requestHeadersAllowlist` | OpenSearch仪表板要求将所有HTTP标头添加到允许列表中，以便将标题传递给OpenSearch。多-租赁使用特定的标头，`securitytenant`，必须与标准一起出现`Authorization` 标题。如果是`securitytenant` 标头不在允许列表上，OpenSearch仪表板从红色状态开始。
| `opensearch_security.multitenancy.enabled` | 启用或禁用多种-OpenSearch仪表板中的租赁。默认为`true`。|
| `opensearch_security.multitenancy.tenants.enable_global` | 启用或禁用全球租户。默认为`true`。|
| `opensearch_security.multitenancy.tenants.enable_private` | 启用或禁用私人租户。默认为`true`。|
| `opensearch_security.multitenancy.tenants.preferred` | 让您更改订单**租户** OpenSearch仪表板的选项卡。默认情况下，列表以全局和私有（如果已启用）开始，然后按字母顺序进行。您可以在此处添加租户将其移至列表的顶部。|
| `opensearch_security.multitenancy.enable_filter` | 如果您有很多租户，则可以在列表的顶部添加搜索栏。默认为`false`。|


## 添加租户

要创建租户，请使用OpenSearch仪表板，REST API或`tenants.yml`。


#### OpenSearch仪表板

1. 开放式搜索仪表板。
1. 选择**安全**，**租户**， 和**创建房客**。
1. 给租户一个名称和描述。
1. 选择**创造**。


#### REST API

看[创建房客]({{site.url}}{{site.baseurl}}/security/access-control/api/#create-tenant)。


#### 租户

```yml
---
_meta：
  类型："tenants"
  config_version：2

## Demo tenants
admin_tenant：
  保留：错误
  描述："Demo tenant for admin user"
```

## Give roles access to tenants

创建租户后，使用OpenSearch仪表板，REST API或`roles.yml`。

- 读-写 （`kibana_all_write`）权限让角色视图并修改租户中的对象。
- 读-仅有的 （`kibana_all_read`）权限让角色查看对象，但不要修改它们。


#### OpenSearch Dashboards

1. 开放式搜索仪表板。
1. 选择**安全**，**角色**和一个角色。
1. 为了**租户许可**，添加租户，按Enter，然后将角色读取和/或写入权限。


#### REST API

请参阅[create prole]（{{site.url}}} {{site.baseurl}}/security/access-控制/API/#创造-角色）。


#### roles.yml

```yml
---
test-role:
  reserved: false
  hidden: false
  cluster_permissions:
  - "cluster_composite_ops"
  - "indices_monitor"
  index_permissions:
  - index_patterns:
    - "movies*"
    dls: ""
    fls: []
    masked_fields: []
    allowed_actions:
    - "read"
  tenant_permissions:
  - tenant_patterns:
    - "human_resources"
    allowed_actions:
    - "kibana_all_read"
  static: false
_meta:
  type: "roles"
  config_version: 2
```


## 管理OpenSearch仪表板索引

OpenSearch仪表板的开源版将所有对象保存到单个索引：`.kibana`。安全插件为全局租户使用此索引，但为其他每个租户使用单独的索引。每个用户也都有一个私人租户，因此您可能会看到大量遵循两种模式的索引：

```
.kibana_<hash>_<tenant_name>
.kibana_<hash>_<username>
```

安全插件擦洗了这些特殊字符的这些索引名称，因此它们可能不是租户名称和用户名的完美匹配。
{: .tip }

要备份您的OpenSearch仪表板数据，[拍摄快照]({{site.url}}{{site.baseurl}}/opensearch/snapshots/snapshot-restore/) 在所有租户索引中使用索引模式，例如`.kibana*`。

