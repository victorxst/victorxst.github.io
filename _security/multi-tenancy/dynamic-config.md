---
layout: default
title: OpenSearch仪表板中的动态配置
parent: OpenSearch 仪表板多租户
nav_order: 147
---


# OpenSearch仪表板中的动态配置

多-租赁包括OpenSearch仪表板中的动态配置选项，因此您可以管理租赁的通用设置，而无需更改每个节点上的配置YAML文件，然后重新启动群集。您可以使用仪表板接口或REST API来利用此功能。以下列表包括动态配置当前涵盖的选项的描述：

- **禁用或启用多重-租赁** - 管理员可以禁用并启用多款-租赁动态。禁用多重-租赁不会构成数据丢失的风险。如果管理员选择重新租赁时，则保留并提供所有先前保存的对象。默认值为`true`。
  
  此设置对全球租户没有影响，该租户始终保持启用。
  {:.note}

- **禁用或启用私人租户** - 此选项允许管理员禁用和启用私人租户。与启用多数-租赁设置，当重新确定私人租户时，将保留所有先前保存的对象并提供。
- **默认租户** - 此选项允许管理员在用户登录时选择全局，私人或自定义租户作为默认设置。在用户无法访问默认租户的情况下（例如，如果用户无法使用自定义租户，则被指定为默认值），默认过渡到优先租户，该租户由`opensearch_security.multitenancy.tenants.preferred` 设置在`opensearch-dashboards.yml` 文件。看[多-租赁配置]({{site.url}}{{site.baseurl}}/security/multi-tenancy/multi-tenancy-config/) 有关此设置的更多信息。

取决于对多人的具体更改-使用动态配置的租赁，一旦保存更改，可能会将某些用户从其仪表板会话中记录出来。例如，如果管理员用户禁用了多个-租赁，拥有私人或自定义租户的用户将被登录其选定的租户，并且需要重新注销。同样，如果管理员用户禁用私人租户，则选择了拥有私人租户的用户，将需要登记，并且需要登录。

但是，全球租户是一个特殊情况。由于该租户从未被禁用，因此选择全球租户的用户被选为他们的活跃租户，不会在会议上遇到任何中断。此外，更改默认租户对用户的会话没有影响。


## 配置Multi-OpenSearch仪表板的租赁

配置多型-仪表板的租赁，请执行以下步骤：

1. 首先选择**安全** 在仪表板主页菜单中。然后选择**租赁** 从屏幕左侧的安全菜单中。这**多-租赁** 显示页面。
1. 默认情况下，**管理** 标签显示。选择**配置** 选项卡以显示多个的动态设置-租赁。
   * 在里面**多-租赁** 字段，选择**启用租赁** 复选框以启用Multi-租赁。清除复选框以禁用该功能。默认值为`true`。
   * 在里面**租户** 字段，您可以为用户启用或禁用私人租户。默认情况下，选择了复选框，并启用了该功能。
   * 在里面**默认租户** 字段，使用下拉菜单选择默认租户。菜单包括用户可用的全球，私人和任何其他自定义租户。
1. 进行首选更改后，选择**保存更改** 在窗户的右下角。流行音乐-出现向上窗口，列出您更改的配置项目，并要求您查看更改。
1. 选择要确认的项目旁边的复选框，然后选择**应用更改**。更改是动态实现的。


## 配置Multi-其余API的租约

除了使用仪表板接口外，您还可以使用REST API管理动态配置。

### 获取租赁配置

Get呼叫检索设置的动态配置：

```json
GET /_plugins/_security/api/tenancy/config
```
{% include copy-curl.html %}

#### 示例响应

```json
{
    "mulitenancy_enabled": true,
    "private_tenant_enabled": true,
    "default_tenant": "global tenant"
}
```

### 更新租户配置

用于动态配置的PUT呼叫更新设置：

```json
PUT /_plugins/_security/api/tenancy/config
{
    "default_tenant": "custom tenant 1",
    "private_tenant_enabled": false,
    "mulitenancy_enabled": true
}
```
{% include copy-curl.html %}

### 示例响应

```json
{
    "mulitenancy_enabled": true,
    "private_tenant_enabled": false,
    "default_tenant": "custom tenant 1"
}
```

### Dashboardsinfo API

您也可以使用dashboardsinfo api检索Multi的状态-登录仪表板的用户的租赁设置：

```json
GET /_plugins/_security/dashboardsinfo
```
{% include copy-curl.html %}

### 示例响应

```json
{
  "user_name" : "admin",
  "not_fail_on_forbidden_enabled" : false,
  "opensearch_dashboards_mt_enabled" : true,
  "opensearch_dashboards_index" : ".kibana",
  "opensearch_dashboards_server_user" : "kibanaserver",
  "multitenancy_enabled" : true,
  "private_tenant_enabled" : true,
  "default_tenant" : "Private"
}
```


