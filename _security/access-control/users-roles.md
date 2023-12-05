---
layout: default
title: 用户和角色
parent: 访问控制
nav_order: 85
redirect_from:
 - /security/access-control/users-roles/
 - /security-plugin/access-control/users-roles/
---

# 用户和角色

安全插件包括一个内部用户数据库。除了外部身份验证系统（例如LDAP或Active Directory）之外，请使用此数据库代替或除了外部身份验证系统之外。

角色是控制对群集访问的核心方式。角色包含群集的任何组合-广泛的权限，指数-特定权限，文件- 和字段-级别安全和租户。然后，您将用户映射到这些角色，以便用户获得这些权限。

除非您需要创建新的[保留或隐藏用户]({{site.url}}{{site.baseurl}}/security/access-control/api/#reserved-and-hidden-resources)， 我们**高度** 建议使用OpenSearch仪表板或REST API创建新用户，角色和角色映射。这`.yml` 文件用于初始设置，而不是持续使用。
{: .warning }

---

#### Table of contents
1. TOC
{:toc}


---

## 创建用户

您可以使用OpenSearch仪表板创建用户，`internal_users.yml`，或剩下的API。创建用户时，您可以使用`internal_users.yml` 或REST API，但是该功能目前在OpenSearch仪表板中尚未可用。

### OpenSearch仪表板

1. 选择**安全**，**内部用户**， 和**创建内部用户**。
1. 提供用户名和密码。安全插件会自动哈希密码并将其存储在`.opendistro_security` 指数。
1. 如果需要，请指定用户属性。

   属性是可选的用户属性，您可以将其用于索引权限或文档中的可变替换-级别的安全性。

1. 选择**提交**。

### internal_users.yml

看[YAML文件]({{site.url}}{{site.baseurl}}/security/configuration/yaml/#internal_usersyml)。


### REST API

看[创建用户]({{site.url}}{{site.baseurl}}/security/access-control/api/#create-user)。


## 创建角色

就像用户一样，您可以使用OpenSearch仪表板创建角色，`roles.yml`，或剩下的API。


### OpenSearch仪表板

1. 选择**安全**，**角色**， 和**创建角色**。
1. 提供角色名称。
1. 根据需要添加权限。

   例如，您可能会扮演任何角色的群集权限，`read` 两个索引的许可，`unlimited` 获得第三个索引的权限，并将其阅读给`analysts` 租户。

1. 选择**提交**。


### 角色

看[YAML文件]({{site.url}}{{site.baseurl}}/security/configuration/yaml/#rolesyml)。


### REST API

看[创建角色]({{site.url}}{{site.baseurl}}/security/access-control/api/#create-role)。


## 将用户映射到角色

如果您在创建用户时没有指定角色，则可以将角色映射到之后。

就像用户和角色一样，您可以使用OpenSearch仪表板创建角色映射，`roles_mapping.yml`，或剩下的API。

### OpenSearch仪表板

1. 选择**安全**，**角色**和一个角色。
1. 选择**映射的用户** 标签和**管理映射**。
1. 指定用户或外部身份（也称为后端角色）。
1. 选择**地图**。


### roles_mapping.yml

看[YAML文件]({{site.url}}{{site.baseurl}}/security/configuration/yaml/#roles_mappingyml)。


### REST API

看[创建角色映射]({{site.url}}{{site.baseurl}}/security/access-control/api/#create-role-mapping)。


## 预定义的角色

安全插件包括几个预定义的角色，这些角色可作为有用的默认值。

| **角色** | **描述** |
| :--- | :--- |
| `alerting_ack_alerts` | 授予查看和确认警报的许可，但不能修改目的地或监视器。|
| `alerting_full_access` | 授予所有警报行动的完全许可。|
| `alerting_read_access` | 授予查看警报，目的地和监视器的权限，但不承认警报或修改目的地或监视器。|
| `anomaly_full_access` | 授予所有异常检测动作的完全许可。|
| `anomaly_read_access` | 授予查看检测器的权限，但不能创建，修改或删除检测器。|
| `all_access` | 授予完全进入集群的访问，包括所有集群-广泛的操作，允许写入所有集群索引的许可以及向所有租户写信的权限。有关使用REST API访问的更多信息，请参见[API的访问控制]({{site.url}}{{site.baseurl}}/security/access-control/api/#access-control-for-the-api)。|
| `cross_cluster_replication_follower_full_access` | 赠款完全访问表演十字架-群集复制操作在追随者群集上。|
| `cross_cluster_replication_leader_full_access` | 赠款完全访问表演十字架-群集复制动作对领导者群集。|
| `observability_full_access` | 授予对可观察到的对象（例如可视化对象，笔记本和操作面板）的全面访问。|
| `observability_read_access` | 授予查看可观察到的对象（例如可视化对象，笔记本和操作面板）的权限，但不会创建，修改或删除它们。|
| `kibana_read_only` | 一个特殊的角色，可防止用户更改可视化，仪表板和其他OpenSearch仪表板对象。启用阅读-仅在仪表板中的模式，添加`opensearch_security.readonly_mode.roles` 设置为`opensearch_dashboards.yml` 文件并将角色作为设置值。看到[示例配置]({{site.url}}{{site.baseurl}}/dashboards/branding/#sample-configuration) 在仪表板文档中。|
| `kibana_user` | 使用OpenSearch仪表板的赠款权限：集群-广泛的搜索，索引监视并写入各种OpenSearch仪表板索引。|
| `logstash` | 授予LogStash与群集交互的权限：群集-广泛的搜索，群集监视并写入各种LogStash索引。|
| `manage_snapshots` | 授予管理快照存储库，拍摄快照并恢复快照的权限。|
| `readall` | 集群的赠款权限-广泛的搜索`msearch` 并搜索所有索引的权限。|
| `readall_and_monitor` | 与...一样`readall` 但是，随着群集的监视权限。|
| `security_rest_api_access` | 允许访问RETS API的特殊角色。看`plugins.security.restapi.roles_enabled` 在`opensearch.yml` 和[API的访问控制]({{site.url}}{{site.baseurl}}/security/access-control/api/#access-control-for-the-api)。|
| `reports_read_access` | 赠款的许可以生成-需求报告，下载现有报告并查看报告定义，但不是为了创建报告定义。|
| `reports_instances_read_access` | 赠款的许可以生成-要求报告并下载现有报告，但不要查看或创建报告定义。|
| `reports_full_access` | 授予报告的全部许可。|
| `asynchronous_search_full_access` | 授予所有异步搜索动作的完全权限。|
| `asynchronous_search_read_access` | 授予权限以查看异步搜索，但不要提交，修改或删除它们。|
| `index_management_full_access` | 授予所有索引管理措施，包括索引状态管理（ISM），变换和汇总。|
| `snapshot_management_full_access` | 授予所有快照管理措施的完全权限。|
| `snapshot_management_read_access` | 授予查看策略的权限，但不要创建，修改，启动，停止或删除它们。|
| `point_in_time_full_access` | 授予所有时间操作的全部许可。|
| `security_analytics_full_access` | 授予所有安全分析功能的全部权限。|
| `security_analytics_read_access` | 授予查看安全分析中各个组件的权限，例如检测器，警报和调查结果。它还包括允许用户搜索检测器和规则的权限。此角色不允许用户执行诸如修改或删除检测器之类的操作。|
| `security_analytics_ack_alerts` | 授予查看和确认警报的权限。|

有关每个角色的权限的更详细摘要，请参考其行动组反对描述[默认操作组]({{site.url}}{{site.baseurl}}/security/access-control/default-action-groups/)。


## 样本角色

以下示例说明了您如何设置读取-唯一的角色和批量访问角色。

### 设置阅读-只有OpenSearch仪表板中的用户

创建一个新的`read_only_index` 角色：

1. 开放式搜索仪表板。
1. 选择**安全**，**角色**。
1. 创建一个名为新角色`read_only_index`。
1. 为了**集群权限**，添加`cluster_composite_ops_ro` 行动小组。
1. 为了**指数许可**，添加索引模式。例如，您可以指定`my-index-*`。
1. 对于索引许可，请添加`read` 行动小组。
1. 选择**创造**。

将三个角色映射到阅读-只有用户：

1. 选择**映射的用户** 标签和**管理映射**。
1. 为了**内部用户**，添加您的阅读-只有用户。
1. 选择**地图**。
1. 重复以下步骤`opensearch_dashboards_user` 和`opensearch_dashboards_read_only` 角色。


### 在OpenSearch仪表板中设置批量访问角色

创建一个新的`bulk_access` 角色：

1. 开放式搜索仪表板。
1. 选择**安全**，**角色**。
1. 创建一个名为新角色`bulk_access`。
1. 为了**集群权限**，添加`cluster_composite_ops` 行动小组。
1. 为了**指数许可**，添加索引模式。例如，您可以指定`my-index-*`。
1. 对于索引许可，请添加`write` 行动小组。
1. 选择**创造**。

将角色映射给您的用户：

1. 选择**映射的用户** 标签和**管理映射**。
1. 为了**内部用户**，添加您的批量访问用户。
1. 选择**地图**。

