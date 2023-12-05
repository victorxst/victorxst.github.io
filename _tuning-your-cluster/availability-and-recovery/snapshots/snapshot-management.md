---
layout: default
title: 快照管理
parent: 快照
nav_order: 20
has_children: false
grand_parent: 可用性和恢复
redirect_from: 
  - /opensearch/snapshots/snapshot-management/
---

# 快照管理

快照管理（SM）使您可以自动化[拍摄快照]({{site.url}}{{site.baseurl}}/opensearch/snapshots/snapshot-restore#take-snapshots)。要使用此功能，您需要安装[索引管理（IM）插件]({{site.url}}{{site.baseurl}}/im-plugin)。快照以自上次快照以来仅存储增量更改。因此，虽然拍摄初始快照可能是一个重型操作，但随后的快照的开销很少。要设置自动快照，您必须创建具有所需的SM计划和配置的SM策略。

当您创建SM策略时，其文档ID将获得名称`<policy_name>-sm-policy`。因此，SM政策必须遵守以下规则：

- SM政策必须具有独特的名称。

- 您无法在其创建后更新策略名称。

SM-创建的快照具有格式的名称`<policy_name>-<date>-<random number>`。不同政策同时创建的两个快照总是有不同的名称`<policy_name>` 字首。为了避免在同一策略中的名称碰撞，每个快照的名称都包含一个随机字符串后缀。

每个策略都有关联的元数据，该元数据存储策略状态。快照管理将SM策略和元数据保存在系统索引中，并从系统索引中读取它们。因此，快照管理取决于OpenSearch集群的索引和搜索功能。该政策的元数据仅保留有关最新创建和删除的信息。在运行每个计划的作业之前，请阅读元数据，以便SM可以继续从上一个作业的状态执行。您可以使用[解释API]({{site.url}}{{site.baseurl}}/opensearch/snapshots/sm-api#explain)。

SM时间表是自定义[克朗]({{site.url}}{{site.baseurl}}/monitoring-plugins/alerting/cron) 表达。它由两个部分组成：创建时间表和删除时间表。您必须设置一个创建时间表，以指定快照创建的频率和时机。可选地，您可以设置单独的时间表以删除快照。

SM配置包括快照的索引和存储库，并支持您可以定义的所有参数[创建快照]({{site.url}}{{site.baseurl}}/opensearch/snapshots/snapshot-restore#take-snapshots) 使用API。此外，您可以为快照名称中使用的日期指定格式和时区。


## 表现

一个快照可以包含与群集中一样多的索引。我们预计最多可以在一个集群中进行数十个SM策略，但是快照存储库可以安全地扩展到数千个快照。但是，要管理其元数据，一个大存储库需要在群集管理器节点上进行更多内存。

快照管理取决于作业调度程序插件来安排定期运行的作业。每个SM策略对应于一个SM-计划的工作。预定的作业很轻巧，因此SM的负担取决于快照创建频率和运行快照操作本身的负担。

## 并发

SM策略不支持并发快照操作，因为太多此类操作可能会降低集群。快照操作（创建或删除）是异步执行的。在先前的异步操作完成之前，SM不会启动新操作。

我们不建议在一个集群中创建几个具有相同时间表和重叠索引的SM策略，因为它会在相同的索引上同时进行快照创建并阻碍性能。
{: .warning }


我们不建议在不同集群中为具有相同时间表的多个SM策略设置相同的存储库，因为这可能会导致此存储库中的负担突然激增。
{: .warning }

## 故障管理

如果快照操作失败，则最多要重新进行三次。故障消息保存在`metadata.latest_execution` 并在随后的快照操作启动时被覆盖。您可以使用[解释API]({{site.url}}{{site.baseurl}}/opensearch/snapshots/sm-api#explain)。使用OpenSearch仪表板时，您可以在[政策详细信息页面]({{site.url}}{{site.baseurl}}/dashboards/admin-ui-index/sm-dashboards#view-edit-or-delete-an-sm-policy)。失败的可能原因包括红色索引状态和碎片重新分配。

## 安全

安全插件有两个构建-在快照管理措施中的角色：`snapshot_management_full_access` 和`snapshot_management_read_access`。对于每个描述，请参阅[预定义的角色]({{site.url}}{{site.baseurl}}/security/access-control/users-roles#predefined-roles)。

下表列出了每个快照管理API所需的权限。

功能| API| 允许
:--- | :--- | :---
获取政策| 获取_plugins/_sm/策略<br>获取_plugins/_sm/policies/`policy_name` | 群集：admin/opensearch/snapshot_management/polity/get <br> cluster：admin/opensearch/snapshot_management/polition/polity/seach
创建/更新策略| 发布_plugins/_sm/策略/`policy_name`<br> put _plugins/_sm/policies/`policy_name`？if_seq_no = 1＆if_primary_term = 1| 群集：admin/opensearch/snapshot_management/polity/write
删除政策| 删除_plugins/_sm/policies/`policy_name` | 群集：admin/opensearch/snapshot_management/polity/delete
解释| 获取_plugins/_sm/策略/`policy_names`/_解释| 群集：admin/opensearch/snapshot_management/polity/light
开始| 发布_plugins/_sm/策略/`policy_name`/_开始| 群集：admin/opensearch/snapshot_management/polity/start
停止| 发布_plugins/_sm/策略/`policy_name`/_停止| 群集：admin/opensearch/snapshot_management/polity/stop


## API

下表列出了所有[快照管理API]({{site.url}}{{site.baseurl}}/opensearch/snapshots/sm-api) 功能。

功能| API| 描述
:--- | :--- | :---
[创建策略]({{site.url}}{{site.baseurl}}/opensearch/snapshots/sm-api#create-or-update-a-policy) | 发布_plugins/_sm/策略/`policy_name` | 创建SM策略。
[更新策略]({{site.url}}{{site.baseurl}}/opensearch/snapshots/sm-api#create-or-update-a-policy) | 放置_plugins/_sm/策略/`policy_name`？if_seq_no =`sequence_number`＆if_primary_term =`primary_term` | 修改`policy_name` 政策。
[获取所有政策]({{site.url}}{{site.baseurl}}/opensearch/snapshots/sm-api#get-policies) | 获取_plugins/_sm/策略| 返回所有SM政策。
[获取政策`policy_name`]({{site.url}}{{site.baseurl}}/opensearch/snapshots/sm-api#get-policies) | 获取_plugins/_sm/策略/`policy_name` | 返回`policy_name` SM政策。
[删除政策]({{site.url}}{{site.baseurl}}/opensearch/snapshots/sm-api#delete-a-policy) | 删除_plugins/_sm/policies/`policy_name` | 删除`policy_name` 政策。
[解释]({{site.url}}{{site.baseurl}}/opensearch/snapshots/sm-api#explain) | 获取_plugins/_sm/策略/`policy_names`/_解释| 为所有策略提供了启用/禁用状态和元数据`policy_names`。
[开始政策]({{site.url}}{{site.baseurl}}/opensearch/snapshots/sm-api#start-a-policy) | 发布_plugins/_sm/策略/`policy_name`/_开始| 开始`policy_name` 政策。
[停止政策]({{site.url}}{{site.baseurl}}/opensearch/snapshots/sm-api#stop-a-policy)| 发布_plugins/_sm/策略/`policy_name`/_停止| 停止`policy_name` 政策

