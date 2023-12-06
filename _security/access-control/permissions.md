---
layout: default
title: 权限
parent: 访问控制
nav_order: 110
redirect_from:
  - /security-plugin/access-control/permissions/
---

# 权限

安全插件中的每个权限都会控制对OpenSearch集群可以执行的操作的访问，例如索引文档或检查集群健康。

大多数权限是自我的-描述。例如，`cluster:admin/ingest/pipeline/get` 让您检索有关摄入管道的信息。_在许多情况下，权限与特定的REST API操作有关，例如`GET _ingest/pipeline`。

尽管有这种相关性，但权限确实**不是** 直接映射到REST API操作。操作，例如`POST _bulk` 和`GET _msearch` 可以访问许多索引并在单个请求中执行许多操作。甚至一个简单的请求，例如`GET _cat/nodes`，执行多个动作以生成其响应。

简而言之，控制对REST API的访问不足。相反，安全插件控制对基础搜索操作的访问。

例如，考虑以下内容`_bulk` 要求：

```json
POST _bulk
{ "delete": { "_index": "test-index", "_id": "tt2229499" } }
{ "index": { "_index": "test-index", "_id": "tt1979320" } }
{ "title": "Rush", "year": 2013 }
{ "create": { "_index": "test-index", "_id": "tt1392214" } }
{ "title": "Prisoners", "year": 2013 }
{ "update": { "_index": "test-index", "_id": "tt0816711" } }
{ "doc" : { "title": "World War Z" } }

```

为了获得成功的要求，您必须拥有以下权限`test-index`：

- 指数：数据/写入/批量*
- 指数：数据/写入/删除
- 索引：数据/写入/索引
- 指数：数据/写入/更新

这些权限还允许您添加，更新或删除文档（例如`PUT test-index/_doc/tt0816711`），因为它们控制着索引和删除文档的基础搜索动作，而不是特定的API路径和HTTP方法。


## 测试权限

如果您希望用户具有执行某些功能所需的绝对最低权限集 - [特权的原则](https://en.wikipedia.org/wiki/Principle_of_least_privilege) - 最好的方法是作为新的测试用户发送代表请求。如果发生权限错误，则安全插件非常明确地丢失了哪些权限。考虑此请求和回应：

```json
GET _cat/shards?v

{
  "error": {
    "root_cause": [{
      "type": "security_exception",
      "reason": "no permissions for [indices:monitor/stats] and User [name=test-user, backend_roles=[], requestedTenant=null]"
    }]
  },
  "status": 403
}
```

[创建用户和角色]({{site.url}}{{site.baseurl}}/security/access-control/users-roles/)，将角色映射到用户，然后开始使用卷曲，邮递员或任何其他客户端发送签名请求。然后在遇到错误时逐渐向角色添加权限。即使解决一个权限错误，同一请求也可能会产生新的错误。该插件仅返回其遇到的第一个错误，因此请继续尝试直到请求成功。

您通常可以使用默认操作组的组合来实现所需的安全姿势，而不是个人权限。看[默认操作组]({{site.url}}{{site.baseurl}}/security/access-control/default-action-groups/) 为了描述每个组授予的权限。
{: .tip }


## 系统索引许可

系统索引许可与其他权限相比是唯一的，因为它们扩展了一些传统的管理-仅访问非-管理用户。这些权限使普通用户能够修改其映射角色或角色中指定的任何系统索引。此例外是安全系统索引，`.opendistro_security`，用于存储安全插件的配置YAML文件，并且仅适用于使用管理员证书的管理员。

除了标准索引许可外，您还指定了系统索引权限`roles.yml` 配置文件下`index_permissions` （看[角色]({{site.url}}{{site.baseurl}}/security/configuration/yaml/#rolesyml)）。这涉及两个-步骤过程：1）在`index_patterns` 第2节）指定`system:admin/system_index` 在角色中`allowed_actions` 部分。

例如，允许用户权限修改系统索引的系统索引权限，该索引存储了警报插件的配置，由索引模式定义`.opendistro-alerting-config`，其允许的行动被定义为`system:admin/system_index`。以下角色显示了该系统索引权限如何以及其他属性的配置：

```yml
alerting-role:
  reserved: true
  hidden: false
  cluster_permissions:
    - 'cluster:admin/opendistro/alerting/alerts/ack'
    - 'cluster:admin/opendistro/alerting/alerts/get'
  index_permissions:
    - index_patterns:
        - .opendistro-alerting-config
    - allowed_actions:
        - 'system:admin/system_index'
```
{% include copy.html %}

系统索引权限还可以与通配符一起使用，以包括部分系统索引名称的所有变体。这可能很有用，但应谨慎使用，以避免对系统索引进行无意的访问。指定角色的系统索引时，请牢记以下注意事项：

*指定系统索引的全名限制仅访问该索引：`.opendistro-alerting-config`。
*与通配符一起指定系统索引的部分名称，可访问以该名称开头的所有系统索引：`.opendistro-anomaly-detector*`。
*虽然不建议---给定宽-达到此角色定义授予的访问权限---使用`*` 对于索引模式以及`system:admin/system_index` 作为允许的动作，可以授予所有系统索引的访问。

  进入通配符`*` 本身以下`allowed_actions` 不会自动授予对系统索引的访问。允许的行动`system:admin/system_index` 必须明确添加。
  {: .note}

以下示例显示了授予所有系统索引访问的角色：

```yml
index_permissions:
    - index_patterns:
        - '*'
    - allowed_actions:
        - 'system:admin/system_index'
```


### 验证系统索引访问

您可以使用[猫指数]({{site.url}}{{site.baseurl}}/api-reference/cat/cat-indices/) 操作以查看与权限配置中任何索引模式相关联的所有索引，并验证权限是否提供您预期的访问权限。例如，如果要验证以前缀开头的系统索引的权限`.kibana`，您可以运行`GET /_cat/indices/.kibana*` 致电以返回与该前缀关联的所有索引。

以下示例响应显示了与索引模式相关的三个系统索引`.kibana*`：

```json
health | status | index | uuid | pri | rep | docs.count | docs.deleted | store.size | pri.store.size
green open .kibana_1 XmTePICFRoSNf5O5uLgwRw 1 1 220 0 468.3kb 232.1kb
green open .kibana_2 XmTePICFRoSNf5O5uLgwRw 1 1 220 0 468.3kb 232.1kb
green open .kibana_3 XmTePICFRoSNf5O5uLgwRw 1 1 220 0 468.3kb 232.1kb
```


### 启用系统索引许可

有许可的用户[`restapi:admin/roles`]({{site.url}}{{site.baseurl}}/security/access-control/api/#access-control-for-the-api) 能够以与在集群或索引许可中相同的方式向所有用户映射系统索引权限`roles.yml` 文件。但是，为了保留对此许可的控制，`plugins.security.system_indices.permissions.enabled` 设置允许您启用或禁用系统索引权限功能。默认情况下禁用此设置。要启用系统索引权限功能，请设置`plugins.security.system_indices.permissions.enabled` 到`true`。有关此设置的更多信息，请参阅[启用用户访问系统索引]({{site.url}}{{site.baseurl}}/security/configuration/yaml/#enabling-user-access-to-system-indexes)。

请记住，向普通用户启用此功能和映射系统索引权限使这些用户可以访问可能包含敏感信息和群集健康所必需的配置的索引。我们还建议您在映射用户到`restapi:admin/roles` 因为此权限不仅为用户提供了将系统索引权限分配给另一个用户的能力-分配对任何系统索引的访问。
{: .warning }


## 集群权限

这些权限是用于集群的，不能粒状应用。例如，您要么有权拍摄快照（`cluster:admin/snapshot/create`）否则你不。因此，群集权限不能授予用户特权来拍摄一组选择索引的快照，同时阻止用户拍摄他人的快照。

叉-以下权限中对API文档的引用仅旨在提供对权限的理解。如本节开头所述，权限通常与API相关，但不会直接映射到它们。
{: .note}


### 摄入API许可

看[摄入的API]({{site.url}}{{site.baseurl}}/api-reference/ingest-apis/index/)。

- 集群：管理/摄入/管道/删除
- 集群：管理/摄入/管道/获取
- 集群：管理/摄入/管道/put
- 集群：管理员/摄入/管道/模拟
- 集群：管理员/摄入/处理器/grok/get

### 异常检测许可

看[异常检测API]({{site.url}}{{site.baseurl}}/observing-your-data/ad/api/)。

- 群集：admin/opendistro/ad/detector/delete
- 群集：admin/opendistro/ad/detector/info
- 群集：admin/opendistro/ad/detector/jobmanagement
- 群集：admin/opendistro/ad/detector/preview
- 群集：admin/opendistro/ad/detector/run
- 群集：admin/opendistro/ad/detector/搜索
- 群集：admin/opendistro/ad/detector/stats
- 群集：admin/opendistro/ad/detector/Write
- 群集：admin/opendistro/ad/detector/valuties
- 群集：admin/opendistro/ad/detector/get
- 群集：admin/opendistro/ad/result/搜索
- 群集：admin/opendistro/ad/result/topanomalies
- 群集：admin/opendistro/ad/task/search

### 警报权限

看[提醒API]({{site.url}}{{site.baseurl}}/observing-your-data/alerting/api/)。

- 群集：admin/opendistro/artering/alerts/ack
- 群集：admin/opendistro/artering/nervers/get
- 集群：admin/opendistro/nervering/destination/delete
- 群集：admin/opendistro/stalling/destination/email_account/delete
- 群集：admin/opendistro/nervering/destination/email_account/获取
- 群集：admin/opendistro/nervering/destination/email_account/搜索
- 群集：admin/opendistro/natering/destination/email_account/Write
- 群集：admin/opendistro/stalling/destination/email_group/delete
- 群集：admin/opendistro/natering/destination/email_group/获取
- 群集：admin/opendistro/stalling/destination/email_group/搜索
- 群集：admin/opendistro/nervering/destination/email_group/Write
- 群集：admin/opendistro/nervering/destination/get
- 群集：admin/opendistro/nervering/destination/Write
- 群集：admin/opendistro/nervering/monitor/delete
- 群集：admin/opendistro/nervering/monitor/ecerute
- 群集：admin/opendistro/nervering/monitor/get
- 群集：admin/opendistro/nervering/Monitor/搜索
- 群集：admin/opendistro/nervering/monitor/Write

### 异步搜索权限

看[异步搜索]({{site.url}}{{site.baseurl}}/search-plugins/async/index/)。

- 群集：admin/opendistro/asynchronous_search/stats
- 群集：admin/opendistro/asynchronous_search/delete
- 群集：admin/opendistro/asynchronous_search/get
- 群集：admin/opendistro/asynchronous_search/提交

### 索引州管理许可

看[ISM API]({{site.url}}{{site.baseurl}}/im-plugin/ism/api/)。

- 群集：admin/opendistro/ism/managedIndex/add
- 群集：admin/opendistro/ism/managedIndex/change
- 群集：admin/opendistro/ism/managedIndex/删除
- 群集：admin/opendistro/ism/managedIndex/解释
- 群集：admin/opendistro/ism/managedIndex/retry
- 群集：admin/opendistro/ism/policy/write
- 群集：admin/opendistro/ism/policy/get
- 群集：admin/opendistro/ism/policy/search
- 群集：admin/opendistro/ism/policy/delete

### 索引汇总许可

看[索引汇总API]({{site.url}}{{site.baseurl}}/im-plugin/index-rollups/rollup-api/)。

- 集群：admin/opendistro/crolup/index
- 群集：admin/opendistro/crolup/get
- 群集：admin/opendistro/lollup/搜索
- 群集：admin/opendistro/lollup/delete
- 群集：admin/opendistro/crolup/start
- 群集：admin/opendistro/crolup/stop
- 群集：admin/opendistro/crolup/解释

### 报告权限

看[使用仪表板接口创建报告]({{site.url}}{{site.baseurl}}/dashboards/reporting/)。

- 群集：admin/opendistro/报告/定义/创建
- 群集：admin/opendistro/报告/定义/更新
- 群集：admin/opendistro/Reports/definity/on_demand
- 群集：admin/opendistro/报告/定义/删除
- 群集：admin/opendistro/报告/定义/获取
- 群集：admin/opendistro/报告/定义/列表
- 群集：admin/opendistro/revorys/实例/列表
- 群集：admin/opendistro/revorys/实例/获取
- 群集：admin/opendistro/revorys/菜单/下载

### 改变工作许可

看[转换API]({{site.url}}{{site.baseurl}}/im-plugin/index-transforms/transforms-apis/)

- 群集：admin/opendistro/transform/index
- 群集：admin/opendistro/transform/get
- 群集：admin/opendistro/transform/preview
- 群集：admin/opendistro/transform/delete
- 群集：admin/opendistro/transform/start
- 群集：admin/opendistro/transform/stop
- 群集：admin/opendistro/transform/解释

### 可观察性许可

看[可观察性安全性]({{site.url}}{{site.baseurl}}/observing-your-data/observability-security/)。

- 群集：管理/opensearch/observability/create
- 集群：管理员/opensearch/observability/update
- 群集：admin/opensearch/observability/delete
- 群集：admin/opensearch/observability/get

### 叉-集群复制

看[叉-集群复制安全性]({{site.url}}{{site.baseurl}}/tuning-your-cluster/replication-plugin/permissions/)。

- 群集：admin/插件/复制/autofollow/更新

### reindex

看[Reindex文档]({{site.url}}{{site.baseurl}}/api-reference/document-apis/reindex/)。

- 群集：管理员/索引/重新树

### 快照存储库权限

看[快照API]({{site.url}}{{site.baseurl}}/api-reference/snapshots/index/)。

- 群集：管理员/存储库/删除
- 集群：管理/存储库/GET
- 集群：管理/存储库/put
- 群集：管理员/存储库/验证

### 重新路由

看[集群管理器任务节流]({{site.url}}{{site.baseurl}}/tuning-your-cluster/cluster-manager-task-throttling/)。

- 群集：管理员/重新路由

### 脚本权限

看[脚本API]({{site.url}}{{site.baseurl}}/api-reference/script-apis/index/)。

- 集群：管理/脚本/删除
- 集群：管理/脚本/获取
- 集群：管理/脚本/put

### 更新设置权限

看[更新设置]({{site.url}}{{site.baseurl}}api-reference/index-apis/update-settings/) 在索引API页面上。

- 集群：管理/设置/更新

### 快照权限

看[快照API]({{site.url}}{{site.baseurl}}/api-reference/snapshots/index/)。

- 群集：管理员/快照/创建
- 群集：管理员/快照/删除
- 集群：管理员/快照/获取
- 集群：管理员/快照/还原
- 集群：管理员/快照/状态
- 群集：管理员/快照/状态*

### 任务权限

看[任务]({{site.url}}{{site.baseurl}}/api-reference/tasks/) 在API参考部分中。

- 集群：管理员/任务/取消
- 集群：管理员/任务/测试
- 群集：管理员/任务/testunblock

### 安全分析许可

看[API工具]({{site.url}}{{site.baseurl}}/security-analytics/api-tools/index/)。

| **允许** | **描述** |
| :--- | :--- |
| 群集：admin/opensearch/securityAnalytics/警报/获取| 获得警报的权限|
| 群集：admin/opensearch/securityAnalytics/arters/ack| 确认警报的权限|
| 集群：admin/opensearch/securityAnalytics/detector/get| 获得探测器的许可|
| 集群：Admin/OpenSearch/SecurityAnalytics/detector/搜索| 许可搜索探测器|
| 群集：admin/opensearch/securityAnalytics/detector/Write| 允许创建和更新检测器|
| 集群：Admin/OpenSearch/SecurityAnalytics/detector/Delete| 许可删除检测器|
| 群集：admin/opensearch/securityAnalytics/findings/get| 获得调查结果的许可|
| 群集：admin/opensearch/securityAnalytics/映射/获取| 允许通过索引获取现场映射|
| 群集：admin/opensearch/securityAnalytics/映射/查看/获取| 允许通过索引获取字段映射，并查看映射和未绘制的字段|
| 集群：admin/opensearch/securityAnalytics/映射/创建| 允许创建现场映射|
| 群集：admin/opensearch/securityAnalytics/映射/更新| 允许更新字段映射|
| 群集：admin/opensearch/securityAnalytics/“规则/类别”| 获得所有规则类别的许可|
| 集群：admin/opensearch/securityAnalytics/rule/Write| 允许创建和更新规则|
| 集群：admin/opensearch/securityAnalytics/rule/search| 许可搜索规则|
| 群集：admin/opensearch/securityAnalytics/ulues/validate| 许可验证规则|
| 集群：admin/opensearch/securityAnalytics/rule/delete| 删除规则的权限|

### 监视权限

监视群集的集群权限适用于阅读-只有操作，例如检查群集健康和获取有关在群集中运行的节点或任务使用的信息。

看[REST API参考]({{site.url}}{{site.baseurl}}/api-reference/index/)。

- 群集：监视/分配/解释
- 集群：监视/健康
- 群集：监视器/主
- 群集：监视器/节点/hot_threads
- 群集：监视/节点/信息
- 群集：显示器/节点/livesice
- 群集：显示器/节点/统计
- 群集：显示/节点/用法
- 群集：监视/远程/信息
- 集群：监视/状态
- 群集：监视器/统计
- 群集：监视/任务
- 群集：监视/任务/获取
- 集群：监视/任务/列表

### 索引模板

索引模板权限适用于索引，但全球适用于群集。

看[索引模板]({{site.url}}{{site.baseurl}}/im-plugin/index-templates/)。

- 索引：admin/index_template/delete
- 索引：admin/index_template/get
- 索引：admin/index_template/put
- 索引：admin/index_template/仿真
- 索引：admin/index_template/simulate_index


## 指数许可

这些权限适用于索引或索引模式。您可能希望用户阅读对所有索引的访问（也就是说`*`），但仅写几个访问权限（例如，`web-logs` 和`product-catalog`）。

- 指数：管理员/别名
- 指数：管理/别名/存在
- 指数：管理/别名/获取
- 指数：管理/分析
- 指数：管理/缓存/清除
- 指数：管理/关闭
- 指数：管理/关闭*
- 索引：管理/创建（创建索引）
- 指数：admin/data_stream/创建
- 索引：admin/data_stream/delete
- 指数：admin/data_stream/get
- 索引：admin/delete（删除索引）
- 指数：管理/存在
- 指数：管理员/齐平
- 指数：管理/齐平*
- 指数：管理员/Forcemerge
- 索引：admin/get（检索索引和映射）
- 指数：管理/映射/放置
- 索引：管理员/映射/字段/获取
- 索引：admin/mappings/fields/get*
- 指数：管理/映射/获取
- 指数：管理/打开
- 索引：管理/插件/复制/索引/设置/验证
- 索引：admin/插件/复制/索引/开始
- 索引：管理/插件/复制/索引/暂停
- 索引：管理/插件/复制/索引/简历
- 索引：管理/插件/复制/索引/停止
- 索引：管理/插件/复制/索引/更新
- 索引：管理/插件/复制/索引/status_check
- 指数：管理/刷新
- 指数：管理/刷新*
- 索引：管理员/解决/索引
- 指数：管理/翻滚
- 索引：admin/seq_no/global_checkpoint_sync
- 指数：管理/设置/更新
- 索引：管理/碎片/search_shards
- 指数：管理/收缩
- 指数：管理员/同步_flush
- 指数：管理/模板/删除
- 指数：管理/模板/获取
- 指数：管理/模板/put
- 指数：管理/类型/存在
- 指数：管理/升级
- 指数：管理/验证/查询
- 指数：数据/读取/解释
- 索引：数据/读/field_caps
- 索引：数据/读/field_caps*
- 指数：数据/读/获取
- 索引：数据/读取/mget
- 索引：数据/读/mget*
- 指数：数据/读/搜索
- 索引：数据/读/搜索/模板
- 索引：数据/读/MTV（Multi-载体）
- 索引：数据/读/MTV*
- 索引：数据/读/插件/复制/file_chunk
- 指数：数据/读/插件/复制/更改
- 索引：数据/读/滚动
- 索引：数据/读/滚动/清除
- 指数：数据/读/搜索
- 索引：数据/读/搜索*
- 索引：数据/读/搜索/模板
- 指数：数据/读/电视（术语向量）
- 指数：数据/写入/批量
- 指数：数据/写入/批量*
- 指数：数据/写入/删除（删除文档）
- 索引：数据/写入/删除/ByQuery
- 指数：数据/写入/插件/复制/更改
- 索引：数据/写入/索引（在现有索引中添加文档）
- 索引：数据/写入/索引
- 指数：数据/写入/更新
- 索引：数据/写入/更新/ByQuery
- 指数：Monitor/data_stream/stats
- 指数：监视/恢复
- 指数：监视/段
- 索引：监视/设置/获取
- 指数：Monitor/Shard_stores
- 指数：显示器/统计
- 指数：监视/升级


## 安全休息权限

这些权限适用于REST API，以控制对端点的访问。授予对任何一个的访问权限，将允许用户获得安全插件的基本操作组件的权限。
允许访问这些端点有可能触发集群中的操作变化。谨慎行事。
{: .warning }

- RESTAPI：管理/行动组
- RESTAPI：管理/允许列表
- RESTAPI：管理员/内部使用者
- RESTAPI：admin/nodesdn
- RESTAPI：管理员/角色
- RESTAPI：管理员/角色图
- RESTAPI：admin/ssl/cert/info
- RESTAPI：Admin/SSL/CERTS/RELOOAD
- RESTAPI：管理员/租户

