---
layout: default
title: 升级 OpenSearch
nav_order: 20
has_children: true
redirect_from:
  - /upgrade-opensearch/index/
---

# 升级 OpenSearch

OpenSearch 项目会定期发布更新，其中包括新功能、增强功能和错误修复。OpenSearch 使用[语义版本控制](https://semver.org/)，这意味着仅在主要版本发布之间引入重大更改。若要了解即将推出的功能和修补程序，请查看[OpenSearch 项目路线图](https://github.com/orgs/opensearch-project/projects/1) GitHub。要查看先前版本的列表或了解有关 OpenSearch 如何使用版本控制的更多信息，请参阅[发布计划和维护政策]({{site.url}}/releases.html)。

我们认识到，用户对升级 OpenSearch 以享受最新功能感到兴奋，我们将继续扩展这些升级和迁移文档，以涵盖其他主题，例如升级 OpenSearch 控制面板和保留自定义配置，例如插件。要查看接下来会发生什么或对未来内容提出请求，请在 GitHub 上[OpenSearch 项目](https://github.com/opensearch-project)发表评论[升级和迁移文档元问题](https://github.com/opensearch-project/documentation-website/issues/2830)。

如果你希望添加特定流程或想要做出贡献，[创建问题](https://github.com/opensearch-project/documentation-website/issues)请在 GitHub 上。请参阅以[贡献者指南](https://github.com/opensearch-project/documentation-website/blob/main/CONTRIBUTING.md)了解如何提供帮助。{: .tip}

## 工作流注意事项

在对群集进行任何更改之前，请花点时间规划该过程。例如，请考虑以下问题：

- 升级过程需要多长时间？
- 如果你的集群在生产中使用，停机的影响有多大？
- 在将新集群迁移到生产环境之前，你是否有适当的基础结构来在测试或开发环境中建立新集群，或者是否需要直接升级生产主机？

此类问题的答案将帮助你确定哪种升级路径最适合你的环境。

你至少应该：

- [查看中断性变更](#reviewing-breaking-changes).
- [查看 OpenSearch 工具兼容性矩阵](#reviewing-the-opensearch-tools-compatibility-matrices).
- [查看插件兼容性](#reviewing-plugin-compatibility).
- [备份配置文件](#backing-up-configuration-files).
- [创建快照](#creating-a-snapshot).

在开始升级过程之前停止任何不必要的索引，以消除在执行升级时对群集的不必要的资源需求。{: .tip}

### 查看中断性变更

确定新版本的 OpenSearch 将如何与你的环境集成非常重要。在开始任何升级过程之前，请查看[中断性变更]({{site.url}}{{site.baseurl}}/breaking-changes/)以确定是否需要对工作流进行调整。例如，可能需要修改上游或下游组件以与 API 更改兼容（请参阅元问题[#2589](https://github.com/opensearch-project/OpenSearch/issues/2589)）。

### 查看 OpenSearch 工具兼容性矩阵

如果你的 OpenSearch 集群与环境中的其他服务（如 Logstash 或 Beats）交互，则应检查以确定[OpenSearch 工具兼容性矩阵]({{site.url}}{{site.baseurl}}/tools/index/#compatibility-matrices)是否需要升级其他组件。

### 查看插件兼容性

查看你用于确定与目标 OpenSearch 版本的兼容性的插件。官方 OpenSearch 项目插件可在 GitHub 上的[OpenSearch 项目](https://github.com/opensearch-project)存储库中找到。如果你使用任何第三方插件，则应检查这些插件的文档以确定它们是否兼容。

转到[可用的插件]({{site.url}}{{site.baseurl}}/install-and-configure/plugins/#available-plugins)查看参考表，其中突出显示了捆绑的 OpenSearch 插件的版本兼容性。

主要、次要和补丁插件版本必须与 OpenSearch 主要版本、次要版本和补丁版本匹配才能兼容。例如，插件版本 2.3.0.x 仅适用于 OpenSearch 2.3.0. {：.important}

### 备份配置文件

通过在开始升级之前备份任何重要文件来降低数据丢失的风险。通常，这些文件将位于以下两个目录中的任何一个目录中：

-  `opensearch/config`
-  `opensearch-dashboards/config`

一些示例包括 `opensearch.yml`、 `opensearch_dashboards.yml`、插件配置文件和 TLS 证书。确定要备份的文件后，请将它们复制到远程存储以确保安全。

如果你使用安全功能，请务必阅读[一句警告]({{site.url}}{{site.baseurl}}/security-plugin/configuration/security-admin/#a-word-of-caution)有关备份和恢复安全设置的信息。

### 创建快照

我们建议你使用[snapshots]({{site.url}}{{site.baseurl}}/opensearch/snapshots/index/)备份集群状态和索引。如果需要将群集回滚到其原始版本，则可以将升级前拍摄的快照用作还原点。

通过将快照存储在外部存储（例如挂载的网络文件系统（NFS）或下表中列出的云存储解决方案）上，可以进一步降低数据丢失的风险。

|快照存储库位置| Required OpenSearch plugin |
| :--- | :--- |
| [Amazon Simple Storage Service (Amazon S3)](https://aws.amazon.com/s3/) |[存储库-S3](https://github.com/opensearch-project/OpenSearch/tree/{{site.opensearch_version}}/plugins/repository-s3)|
| [Google Cloud Storage (GCS)](https://cloud.google.com/storage) |[存储库-GCS](https://github.com/opensearch-project/OpenSearch/tree/{{site.opensearch_version}}/plugins/repository-gcs)|
| [Apache Hadoop Distributed File System (HDFS)](https://hadoop.apache.org/) |[存储库-hdfs](https://github.com/opensearch-project/OpenSearch/tree/{{site.opensearch_version}}/plugins/repository-hdfs)|
| [Microsoft Azure Blob Storage](https://azure.microsoft.com/en-us/products/storage/blobs) |[repository-azure](https://github.com/opensearch-project/OpenSearch/tree/{{site.opensearch_version}}/plugins/repository-azure)|

## 升级方法

根据你的要求，选择将集群升级到新版 OpenSearch 的适当方法：

- A [滚动升级](#rolling-upgrade)在不停止集群的情况下一次升级一个节点。
- A [集群重启升级](#cluster-restart-upgrade)在集群停止时升级服务。

由于需要重新编制索引，跨 OpenSearch 的多个主要版本进行升级将需要额外的工作。有关详细信息，请参阅[Reindex]({{site.url}}{{site.baseurl}}/api-reference/document-apis/reindex/) API。请参阅本指南后面包含的[索引兼容性参考](#index-compatibility-reference)表，以获取有关规划数据迁移的帮助。

### 滚动升级

如果要在整个过程中保持集群正常运行，滚动升级是一个不错的选择。当节点单独停止、升级和重新启动时，可能会继续引入、分析和查询数据。滚动升级的变体（称为“节点替换”）遵循完全相同的过程，只是主机和容器不会重新用于新节点。如果你还要升级底层主机，则可以执行节点替换。

如果集群管理器运行的 OpenSearch 版本比请求成员资格的节点更新，则 OpenSearch 节点无法加入集群。为避免此问题，请最后升级符合 cluster-manager 条件的节点。


有关该过程的详细信息，请参阅[滚动升级]({{site.url}}{{site.baseurl}}/install-and-configure/upgrade-opensearch/rolling-upgrade/)。

### 集群重启升级

OpenSearch 管理员可能出于多种原因选择执行集群重启升级，例如，如果管理员不想对正在运行的集群执行维护，或者集群要迁移到其他环境。

与一次只有一个节点脱机的滚动升级不同，集群重启升级要求你在继续之前停止集群中所有节点上的 OpenSearch 和 OpenSearch 控制面板。节点停止后，将安装新版本的 OpenSearch。然后启动 OpenSearch，集群引导到新版本。

## 兼容性

OpenSearch 节点与在同一*主要*版本版本中运行任何其他*次要*版本的其他 OpenSearch 节点兼容。例如，1.1.0 与 1.3.7 兼容，因为它们属于同一*主要*版本（1.x）。此外，OpenSearch 节点和索引向后兼容之前的主要版本。这意味着，例如，由运行任何 1.x 版本的 OpenSearch 节点创建的索引可以从快照还原到运行任何 2.x 版本的 OpenSearch 集群。

OpenSearch 1.x 节点与运行 Elasticsearch 7.x 的节点兼容，但混合版本环境的寿命不应超出集群升级活动。{: .tip}

索引兼容性由创建索引的[Apache Lucene](https://lucene.apache.org/)版本决定。如果索引是由运行版本 1.0.0 的 OpenSearch 集群创建的，则该索引可由运行到最新 1.x 或 2.x 版本的任何其他 OpenSearch 集群使用。请参阅表格，[索引兼容性参考](#index-compatibility-reference)了解在 OpenSearch 1.0.0 及更高版本以及[Elasticsearch 的](https://www.elastic.co/) 6.8 及更高版本中运行的 Lucene 版本。

如果你的升级路径跨越多个主要版本，并且你希望保留任何现有索引，则可以在升级之前使用[Reindex]({{site.url}}{{site.baseurl}}/api-reference/document-apis/reindex/) API 使索引与 OpenSearch 的目标版本兼容。例如，如果你的集群当前运行的是 Elasticsearch 6.8，并且你想要升级到 OpenSearch 2.x，则必须先升级到 OpenSearch 1.x，使用[Reindex]({{site.url}}{{site.baseurl}}/api-reference/document-apis/reindex/) API 重新创建索引，最后升级到 2.x。重新编制索引的一种替代方法是从源重新引入数据，例如重播数据流或从数据库引入数据。

### 索引兼容性参考

如果你计划在 OpenSearch 版本升级后保留旧索引，则可能需要重新编制索引或重新摄取数据。请参阅下表，了解最新 OpenSearch 和 Elasticsearch 版本中的 Lucene 版本。

<style>
table {
    border-collapse: collapse;
    table-layout: fixed;
}
th {
  background-color: #F5F7F7;
}
th,
td {
  text-align: center;
  padding: 0.5em 1em;
}
</style>
<table>
    <tr>
        <th>Lucene Version</th>
        <th>OpenSearch Version</th>
        <th>Elasticsearch Version</th>
    </tr>
    <tr>
        <td>9.4.2</td>
        <td>2.5.0<br>2.4.1</td>
        <td>8.6</td>
    </tr>
    <tr>
        <td>9.4.1</td>
        <td>2.4.0</td>
        <td>&#8212;</td>
    </tr>
    <tr>
        <td>9.4.0</td>
        <td>&#8212;</td>
        <td>8.5</td>
    </tr>
    <tr>
        <td>9.3.0</td>
        <td>2.3.0<br>2.2.x</td>
        <td>8.4</td>
    </tr>
    <tr>
        <td>9.2.0</td>
        <td>2.1.0</td>
        <td>8.3</td>
    </tr>
    <tr>
        <td>9.1.0</td>
        <td>2.0.x</td>
        <td>8.2</td>
    </tr>
    <tr>
        <td>9.0.0</td>
        <td>&#8212;</td>
        <td>8.1<br>8.0</td>
    </tr>
    <tr>
        <td>8.11.1</td>
        <td>&#8212;</td>
        <td>7.17</td>
    </tr>
    <tr>
        <td>8.10.1</td>
        <td>1.3.x<br>1.2.x</td>
        <td>7.16</td>
    </tr>
    <tr>
        <td>8.9.0</td>
        <td>1.1.0</td>
        <td>7.15<br>7.14</td>
    </tr>
    <tr>
        <td>8.8.2</td>
        <td>1.0.0</td>
        <td>7.13</td>
    </tr>
    <tr>
        <td>8.8.0</td>
        <td>&#8212;</td>
        <td>7.12</td>
    </tr>
    <tr>
        <td>8.7.0</td>
        <td>&#8212;</td>
        <td>7.11<br>7.10</td>
    </tr>
    <tr>
        <td>8.6.2</td>
        <td>&#8212;</td>
        <td>7.9</td>
    </tr>
    <tr>
        <td>8.5.1</td>
        <td>&#8212;</td>
        <td>7.8<br>7.7</td>
    </tr>
    <tr>
        <td>8.4.0</td>
        <td>&#8212;</td>
        <td>7.6</td>
    </tr>
    <tr>
        <td>8.3.0</td>
        <td>&#8212;</td>
        <td>7.5</td>
    </tr>
    <tr>
        <td>8.2.0</td>
        <td>&#8212;</td>
        <td>7.4</td>
    </tr>
    <tr>
        <td>8.1.0</td>
        <td>&#8212;</td>
        <td>7.3</td>
    </tr>
    <tr>
        <td>8.0.0</td>
        <td>&#8212;</td>
        <td>7.2<br>7.1</td>
    </tr>
    <tr>
        <td>7.7.3</td>
        <td>&#8212;</td>
        <td>6.8</td>
    </tr>
</table>
<p style="text-align:right"><sub><em>短划线（—）表示没有包含指定版本的 Apache Lucene 的产品版本。</em></sub></p>
