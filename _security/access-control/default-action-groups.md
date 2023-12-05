---
layout: default
title: 默认操作组
parent: 访问控制
nav_order: 115
redirect_from:
 - /security/access-control/default-action-groups/
 - /security-plugin/access-control/default-action-groups/
---

# 默认操作组

此页面分类所有默认操作组。通常，创建新行动组的最连贯方法是使用这些默认组的组合和[个人许可]({{site.url}}{{site.baseurl}}/security/access-control/permissions/)。


## 一般的

姓名| 描述
:--- | :---
无限| 赠款完全访问。可以在集群上使用- 或索引-等级。等于`"*"`。
{％注释％} kibana_all_read| ASDF
kibana_all_write| ASDF {％端次}



## 簇-等级

姓名| 描述
：---| ：---
cluster_all| 授予所有集群权限。等于`cluster:*`。
cluster_monitor| 授予所有集群监控权限。等于`cluster:monitor/*`。
cluster_composite_ops_ro| 赠款阅读-仅执行请求之类的许可`mget`，`msearch`， 或者`mtv`，加上查询别名的许可。
cluster_composite_ops| 与...一样`CLUSTER_COMPOSITE_OPS_RO`，但也赠款`bulk` 权限和所有别名许可。
manage_snapshots| 授予管理快照和存储库的权限。
cluster_manage_pipelines| 授予管理摄入管道的许可。
cluster_manage_index_templates| 授予管理索引模板的权限。


## 指数-等级

姓名| 描述
:--- | :---
indices_all| 授予索引上的所有权限。等于`indices:*`。
得到| 赠款使用权限`get` 和`mget` 仅动作。
读| 赠款阅读许可，例如搜索，获取现场映射，`get`， 和`mget`。
写| 授予在 *现有索引 *中创建和更新文档的权限。要创建新索引，请参阅`create_index`。
删除| 授予删除文件的许可。
克鲁德| 结合`read`，`write`， 和`delete` 行动组。包括在`data_access` 行动小组。
搜索| 授予搜索文件的许可。包括`suggest`。
建议| 授予使用建议API的许可。包括在`read` 行动小组。
create_index| 授予创建索引和映射的权限。
indices_monitor| 授予执行所有索引监视操作的权限（例如恢复，段信息，索引统计和状态）。
指数| 更有限的版本的`write` 行动小组。
data_access| 结合`crud` 行动小组与`indices:data/*`。
manage_aliases| 授予管理别名的许可。
管理| 授予所有监控和管理权限的索引许可。

