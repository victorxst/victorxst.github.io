---
layout: default
title: 从 Elasticsearch OSS 迁移到 OpenSearch
nav_order: 15
---

# 从 Elasticsearch OSS 迁移到 OpenSearch

如果你想从现有的 Elasticsearch OSS 集群迁移到 OpenSearch，但发现没有吸引力，[快照方法]({{site.url}}{{site.baseurl}}/upgrade-to/snapshot-migrate/)你可以将现有节点从 Elasticsearch OSS 迁移到 OpenSearch。

如果你现有的集群运行的是旧版本的 Elasticsearch OSS，第一步是升级到 6.x 或 7.x 版本。

在决定升级的 Elasticsearch OSS 版本之前，请参阅[迁移到 OpenSearch 以及对嵌套 JSON 对象数量的限制]({{site.url}}{{site.baseurl}}/breaking-changes/#migrating-to-opensearch-and-limits-on-the-number-of-nested-json-objects)中断性变更中的文档，了解该问题是否会对你的集群产生影响，从而影响你对升级和迁移的决策。{：.important }

Elasticsearch OSS 支持两种升级方式：滚动升级和集群重启。

- 通过滚动升级，你可以一次关闭一个节点，以最大程度地减少服务中断。

  滚动升级在次要版本（例如，6.5 到 6.8）之间工作，并且还支持指向下一个主要版本（例如，6.8 到 7.10.2）的单个路径。执行这些升级可能需要中间升级才能达到所需的版本，并且可能会在节点离开和重新加入时影响集群性能，但集群在整个过程中仍然可用。

- 群集重新启动升级要求你关闭所有节点，执行升级，然后重新启动群集。

  群集重启升级在次要版本（例如，6.5 到 6.8）和下一个主要版本（例如，6.x 到 7.10.2）之间工作。群集重新启动升级的执行速度更快，需要的中间升级更少，但需要停机。

要将 Elasticsearch （7.11+）的分叉后版本迁移到 OpenSearch，你可以使用 Logstash。你需要在 Logstash 中使用 Elasticsearch 输入插件从 Elasticsearch 集群中提取数据，并将[Logstash 输出 OpenSearch 插件](https://github.com/opensearch-project/logstash-output-opensearch#configuration-for-logstash-output-opensearch-plugin)数据写入 OpenSearch 2.x 集群。我们建议使用 Logstash 版本 7.13.4 或更早版本，因为较新版本在与 OpenSearch 建立连接时可能会遇到兼容性问题，因为 Elasticsearch 在分叉后引入了更改。我们强烈建议用户使用自己的数据测试此解决方案，以确保有效性。
{: .note}

## 迁移路径

Elasticsearch OSS 版本 | 滚动升级路径 | 群集重启升级路径
:--- | :--- | :---
5.x | 升级到 5.6，升级到 6.8，重新索引所有 5.x 索引，升级到 7.10.2，然后迁移到 OpenSearch。| 升级到 6.8，重新索引所有 5.x 索引，然后迁移到 OpenSearch。
6.x | 升级到 6.8，升级到 7.10.2，然后迁移到 OpenSearch。| 迁移到 OpenSearch。
7.x | 迁移到 OpenSearch。| 迁移到 OpenSearch。

如果你要迁移 Open Distro for Elasticsearch 集群，我们建议你先升级到 ODFE 1.13，然后再迁移到 OpenSearch。
{: .note}


## 升级 Elasticsearch OSS

1. 关闭分片分配，防止 Elasticsearch OSS 在关闭节点时复制分片：

   ```json
   PUT _cluster/settings
   {
     "persistent": {
       "cluster.routing.allocation.enable": "primaries"
     }
   }
   ```

1. 在一个节点（滚动升级）或所有节点（集群重启升级）上停止 Elasticsearch OSS。

   在使用 systemd 的 Linux 发行版上，使用以下命令：

   ```bash
   sudo systemctl stop elasticsearch.service
   ```

   对于 tarball 安装，找到进程 ID（）并终止它（ `ps aux` `kill <pid>`）。

1. 升级节点（滚动）或所有节点（群集重启）。

   确切的命令因包管理器而异，但可能如下所示：

   ```bash
   sudo yum install elasticsearch-oss-7.10.2 --enablerepo=elasticsearch
   ```

   对于 tarball 安装，请解压缩到新目录以确保**不覆盖** `config`、 `data` 和 `logs` 目录。理想情况下，这些目录应该有自己的独立路径，并与*不* Elasticsearch 应用程序目录位于同一位置。然后将 `ES_PATH_CONF` 环境变量设置为包含 `elasticsearch.yml`（例如， `/etc/elasticsearch/`）。在中 `elasticsearch.yml`，将和 设置为 `path.data` 和 `logs` `data` 目录（例如， `/var/lib/elasticsearch` 和 `path.logs` `/var/log/opensearch`）。

1. 在节点（滚动）或所有节点（集群重启）上重启 Elasticsearch OSS。

   在使用 systemd 的 Linux 发行版上，使用以下命令：

   ```bash
   sudo systemctl start elasticsearch.service
   ```

   对于 tarball 安装，请运行 `./bin/elasticsearch -d`。

1. 等待节点重新加入集群（滚动）或集群启动（集群重启）。 `_nodes` 检查摘要以验证所有节点是否都可用并运行预期版本：

   ```bash
   # Elasticsearch OSS
   curl -XGET 'localhost:9200/_nodes/_all?pretty=true'
   # Open Distro for Elasticsearch with Security plugin enabled
   curl -XGET 'https://localhost:9200/_nodes/_all?pretty=true' -u 'admin:admin' -k
   ```

   具体而言，请检查 `nodes.<node-id>.version` 响应的部分。此外，检查 `_cat/indices?v` 所有索引的绿色状态。

1. （滚动）重复步骤 2--5，直到所有节点都使用新版本。

1. 所有节点都使用新版本后，重新启用分片分配：

   ```json
   PUT _cluster/settings
   {
     "persistent": {
       "cluster.routing.allocation.enable": "all"
     }
   }
   ```

1. 如果从 5.x 升级到 6.x，[reindex]({{site.url}}{{site.baseurl}}/opensearch/reindex-data/)则所有索引。

1. 根据需要重复所有步骤，直到获得所需的 Elasticsearch OSS 版本。


## 迁移到 OpenSearch

1. 关闭分片分配，防止 Elasticsearch OSS 在关闭节点时复制分片：

   ```json
   PUT _cluster/settings
   {
     "persistent": {
       "cluster.routing.allocation.enable": "primaries"
     }
   }
   ```

1. 在一个节点（滚动升级）或所有节点（集群重启升级）上停止 Elasticsearch OSS。

   在使用 systemd 的 Linux 发行版上，使用以下命令：

   ```bash
   sudo systemctl stop elasticsearch.service
   ```

   对于 tarball 安装，找到进程 ID（）并终止它（ `ps aux` `kill <pid>`）。

1. 升级节点（滚动）或所有节点（群集重启）。

   1. 将 OpenSearch 压缩包解压缩到新目录，以确保你的**不覆盖** Elasticsearch OSS `config`、 `data` 和 `logs` 目录。

   1. （可选）将 Elasticsearch OSS `data` 和 `logs` 目录复制或移动到新路径。例如，你可以移动到 `/var/lib/elasticsearch` `/var/lib/opensearch`。

   1. 将 `OPENSEARCH_PATH_CONF` 环境变量设置为包含 `opensearch.yml`（例如， `/etc/opensearch`）。

   1. 在 `opensearch.yml`、和 `path.data` `path.logs` 中。你可能还想暂时禁用安全插件。 `opensearch.yml` 可能看起来像这样：

      ```yml
      path.data: /var/lib/opensearch
      path.logs: /var/log/opensearch
      plugins.security.disabled: true
      ```

   1. 将你的设置从 `elasticsearch.yml` 移植到 `opensearch.yml`。大多数设置使用相同的名称。至少指定 `cluster.name`、、 `node.name` `discovery.seed_hosts` 和 `cluster.initial_cluster_manager_nodes`。

   1. （可选）如果你使用检查特定版本号的旧客户端（例如 Logstash OSS）主动连接到集群，请将 a [兼容性设置]({{site.url}}{{site.baseurl}}/tools/index/)添加到 `opensearch.yml`：

      ```yml
      compatibility.override_main_response_version: true
      ```

   1. （可选）将证书添加到 `config` 目录，将其 `opensearch.yml` 添加到，然后初始化 Security 插件。

1. 在节点（滚动）或所有节点（集群重启）上启动 OpenSearch。

   对于 tarball，请运行 `./bin/opensearch -d`.

1. 等待 OpenSearch 节点重新加入集群（滚动）或集群启动（集群重启）。 `_nodes` 检查摘要以验证所有节点是否都可用并运行预期版本：

   ```bash
   # Security plugin disabled
   curl -XGET 'localhost:9200/_nodes/_all?pretty=true'
   # Security plugin enabled
   curl -XGET -k -u 'admin:admin' 'https://localhost:9200/_nodes/_all?pretty=true'
   ```

   具体而言，请检查 `nodes.<node-id>.version` 响应的部分。此外，检查 `_cat/indices?v` 所有索引的绿色状态。

1. （滚动）重复步骤 2--5，直到所有节点都使用 OpenSearch。

1. 所有节点都使用新版本后，重新启用分片分配：

   ```json
   PUT _cluster/settings
   {
     "persistent": {
       "cluster.routing.allocation.enable": "all"
     }
   }
   ```

## 升级工具

该 `opensearch-upgrade` 工具可让你自动执行中的某些步骤[迁移到 OpenSearch]({{site.url}}{{site.baseurl}}/upgrade-to/upgrade-to/#migrate-to-opensearch)，而无需进行容易出错的手动操作。

该 `opensearch-upgrade` 工具执行以下功能：

- 导入任何现有配置并将其应用于 OpenSearch 的新安装。
- 安装任何现有的核心插件。

### 局限性

该工具不执行端到端升级：The `opensearch-upgrade` tool doesn't perform an end-to-end upgrade：

- 作为升级过程的一部分，你需要在群集的每个节点上单独运行该工具。
- 升级节点后，该工具不提供回滚选项，因此请确保遵循最佳实践并进行备份。
- 你必须手动安装所有社区插件（如果可用）。
- 该工具仅在服务启动时验证任何密钥库设置，因此你必须手动删除任何不受支持的设置才能启动服务。

### 使用升级工具

要使用发行版执行滚动升级，[OpenSearch 压缩包]({{site.url}}{{site.baseurl}}/opensearch/install/tar/)请执行以下操作：

检查[迁移路径]({{site.url}}{{site.baseurl}}/upgrade-to/upgrade-to/#migration-paths)以确保你要升级到的版本受支持，以及是否需要先升级到受支持的 Elasticsearch OSS 版本。
{: .note}

1. 关闭分片分配，防止 Elasticsearch OSS 在关闭节点时复制分片：

   ```json
   PUT _cluster/settings
   {
     "persistent": {
       "cluster.routing.allocation.enable": "primaries"
     }
   }
   ```

1. 在任何一个节点上，下载 OpenSearch tarball 并将其解压缩到新目录。

1. 请确保设置了以下环境变量：

    -  `ES_HOME` - 现有 Elasticsearch 安装主页的路径。

      ```bash
      export ES_HOME=/home/workspace/upgrade-demo/node1/elasticsearch-7.10.2
      ```

    -  `ES_PATH_CONF` - 现有 Elasticsearch 配置目录的路径。

      ```bash
      export ES_PATH_CONF=/home/workspace/upgrade-demo/node1/os-config
      ```

    -  `OPENSEARCH_HOME` - OpenSearch 安装主页的路径。

      ```bash
      export OPENSEARCH_HOME=/home/workspace/upgrade-demo/node1/opensearch-1.0.0
      ```

    -  `OPENSEARCH_PATH_CONF` - OpenSearch 配置目录的路径。

      ```bash
      export OPENSEARCH_PATH_CONF=/home/workspace/upgrade-demo/node1/opensearch-config
      ```

1. 该 `opensearch-upgrade` 工具位于 `bin` 发行版的目录中。从分发主目录运行以下命令：

   请确保你以运行当前 Elasticsearch 服务的同一用户身份运行此工具。
{: .note}

   ```json
   ./bin/opensearch-upgrade
   ```

1. 在节点上停止 Elasticsearch OSS。

   在使用 systemd 的 Linux 发行版上，使用以下命令：

   ```bash
   sudo systemctl stop elasticsearch.service
   ```

   对于 tarball 安装，找到进程 ID（）并终止它（ `ps aux` `kill <pid>`）。

1. 在节点上启动 OpenSearch：

   ```json
   ./bin/opensearch -d.
   ```

1. 重复步骤 2--6，直到所有节点都使用新版本。

1. 所有节点都使用新版本后，重新启用分片分配：

   ```json
   PUT _cluster/settings
   {
    "persistent": {
       "cluster.routing.allocation.enable": "all"
     }
   }
   ```

### 运作方式

在后台，该 `opensearch-upgrade` 工具按顺序执行以下任务：

1. 在当前节点上查找有效的 Elasticsearch 安装。找到安装后，它会读取 `elasticsearch.yml` 文件以获取端点详细信息，并连接到本地运行的 Elasticsearch 服务。如果该工具找不到 Elasticsearch 安装，它会尝试从该 `ES_HOME` 位置获取路径。
1. 验证 Elasticsearch 的现有版本是否与 OpenSearch 版本兼容。它将收集到的信息摘要打印到控制台，并提示你确认以继续。
1. 将配置文件中的设置 `elasticsearch.yml` 导入到配置文件中 `opensearch.yml`。
1. 将任何自定义 JVM 选项从 `$ES_PATH_CONF/jvm.options.d` 目录 `$OPENSEARCH_PATH_CONF/jvm.options.d` 复制到目录中。同样，它还将日志记录配置从 `$ES_PATH_CONF/log4j2.properties` 文件导入到 `$OPENSEARCH_PATH_CONF/log4j2.properties` 文件中。
1. 安装你当前已安装在 `$ES_HOME/plugins` 目录中的核心插件。你必须手动安装所有其他第三方社区插件。
1. 将安全设置从 `elasticsearch.keystore` 文件（如果有）导入到 `opensearch.keystore` 文件中。如果密钥库文件受密码保护，那么该 `opensearch-upgrade` 工具会提示你输入密码。
