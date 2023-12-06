---
layout: default
title: 滚动升级实验室
parent: 升级附录
grand_parent: 升级 OpenSearch
nav_order: 50
redirect_from:
  - /upgrade-opensearch/appendix/rolling-upgrade-lab/
---

<!--
Testing out tabs for code blocks to identify example outputs and file names.
To use, invoke class="codeblock-label"
-->

<style>
.codeblock-label {
    display: inline-block;
    border-top-left-radius: 0.5rem;
    border-top-right-radius: 0.5rem;
    font-family: Menlo,Monaco,Consolas,Liberation Mono,Courier New,monospace;
    font-size: .75rem;
    --bg-opacity: 1;
    background-color: #e1e7ef;
    background-color: rgba(224.70600000000002,231.07080000000002,239.394,var(--bg-opacity));
    padding: 0.25rem 0.75rem;
    border-top-width: 1px;
    border-left-width: 1px;
    border-right-width: 1px;
    --border-opacity: 1;
    border-color: #ccd6e0;
    border-color: rgba(204,213.85999999999999,224.39999999999998,var(--border-opacity));
    margin-bottom: 0;
}
</style>

# 滚动升级实验室

你可以在自己的兼容主机上执行以下步骤<OuiToken iconType="tokenStruct" size="xs" color="gray" />，重新创建 OpenSearch 项目用于测试[滚动升级]({{site.url}}{{site.baseurl}}/install-and-configure/upgrade-opensearch/rolling-upgrade/)的相同集群状态。如果要在开发环境中测试升级过程，本练习非常有用。

本实验中使用的步骤已使用[亚马逊 Linux 2](https://aws.amazon.com/amazon-linux-2/)内核版本和[Docker](https://www.docker.com/)版本 `Linux 5.10.162-141.675.amzn2.x86_64` `20.10.17, build 100c701` 在任意选择[亚马逊弹性计算云（Amazon EC2）](https://aws.amazon.com/ec2/) `t2.large` 的实例上进行了验证。该实例预置了一个附加的 20 GiB gp2 [Amazon EBS](https://aws.amazon.com/ebs/)根卷。这些规格仅供参考，并不代表 OpenSearch 或 OpenSearch 控制面板的硬件要求。

在此过程中，对 `$HOME` 此过程中主机上路径的引用由波浪号字符（“~”）表示，以使指令更易于移植。如果你希望指定绝对路径，请修改本文档中 `upgrade-demo-cluster.sh` 定义并在相关命令中使用的卷路径，以反映你的环境。

## 设置环境

按照本文档中的步骤操作时，你将使用我们提供的脚本定义多个 Docker 资源，包括容器、卷和专用 Docker 网络。如果要重新启动该过程，可以使用以下命令清理环境：

```bash
docker container stop $(docker container ls -aqf name=os-); \
	docker container rm $(docker container ls -aqf name=os-); \
	docker volume rm -f $(docker volume ls -q | egrep 'data-0|repo-0'); \
	docker network rm opensearch-dev-net
```
{% include copy.html %}

该命令删除与正则表达式 `os-*` 匹配的容器名称、与和 `repo-0*` 匹配 `data-0*` 的数据卷以及名为 `opensearch-dev-net` 的 Docker 网络。如果主机上运行了其他 Docker 资源，则应查看并修改该命令，以避免无意中删除其他资源。此命令不会还原主机配置更改，例如内存交换行为。
{: .warning}

选择主机后，可以开始实验：

1. 为你的 Linux 发行版和系统体系结构安装适当的版本[Docker 引擎](https://docs.docker.com/engine/install/)。
1. 在主机上配置[重要的系统设置]({{site.url}}{{site.baseurl}}/install-and-configure/install-opensearch/index/#important-settings)：
    1. 在主机上禁用内存分页和交换以提高性能：
	   ```bash
	   sudo swapoff -a
	   ```
{% include copy.html %}
1. 增加可用于 OpenSearch 的内存映射数量。 `sysctl` 打开配置文件进行编辑。此示例命令使用[vim](https://www.vim.org/)文本编辑器，但你可以使用任何可用的文本编辑器：
	   ```bash
	   sudo vim /etc/sysctl.conf
	   ```
{% include copy.html %}
1. 将以下行 `/etc/sysctl.conf` 添加到：
	   ```bash
	   vm.max_map_count=262144
	   ```
{% include copy.html %}
1. 保存并退出。如果使用 `vi` 或文本编辑器，则通过切换到命令模式并输入 `:wq!` 或 `vim` `ZZ` 来保存并退出。
1. 应用配置更改：
	   ```bash
	   sudo sysctl -p
	   ```
{% include copy.html %}
1. 创建一个在主目录中调用 `deploy` 的新目录，然后导航到该目录。你将用于 `~/deploy` 部署脚本、配置文件和 TLS 证书中的路径：
   ```bash
   mkdir ~/deploy && cd ~/deploy
   ```
   {% include copy.html %}
1. 从 OpenSearch 项目[文档网站](https://github.com/opensearch-project/documentation-website)存储库下载 `upgrade-demo-cluster.sh`：
   ```bash
   wget https://raw.githubusercontent.com/opensearch-project/documentation-website/main/assets/examples/upgrade-demo-cluster.sh
   ```
   {% include copy.html %}
1. 在不进行任何修改的情况下运行脚本，以便使用自定义的自签名 TLS 证书和一组预定义的内部用户部署四个运行 OpenSearch 的容器和一个运行 OpenSearch 控制面板的容器：
   ```bash
   sh upgrade-demo-cluster.sh
   ```
   {% include copy.html %}
1. 确认容器已成功启动：
   ```bash
   docker container ls
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```bash
   CONTAINER ID   IMAGE                                           COMMAND                  CREATED          STATUS          PORTS                                                                                                      NAMES
   6e5218c8397d   opensearchproject/opensearch-dashboards:1.3.7   "./opensearch-dashbo…"   24 seconds ago   Up 22 seconds   0.0.0.0:5601->5601/tcp, :::5601->5601/tcp                                                                  os-dashboards-01
   cb5188308b21   opensearchproject/opensearch:1.3.7              "./opensearch-docker…"   25 seconds ago   Up 24 seconds   9300/tcp, 9650/tcp, 0.0.0.0:9204->9200/tcp, :::9204->9200/tcp, 0.0.0.0:9604->9600/tcp, :::9604->9600/tcp   os-node-04
   71b682aa6671   opensearchproject/opensearch:1.3.7              "./opensearch-docker…"   26 seconds ago   Up 25 seconds   9300/tcp, 9650/tcp, 0.0.0.0:9203->9200/tcp, :::9203->9200/tcp, 0.0.0.0:9603->9600/tcp, :::9603->9600/tcp   os-node-03
   f894054a9378   opensearchproject/opensearch:1.3.7              "./opensearch-docker…"   27 seconds ago   Up 26 seconds   9300/tcp, 9650/tcp, 0.0.0.0:9202->9200/tcp, :::9202->9200/tcp, 0.0.0.0:9602->9600/tcp, :::9602->9600/tcp   os-node-02
   2e9c91c959cd   opensearchproject/opensearch:1.3.7              "./opensearch-docker…"   28 seconds ago   Up 27 seconds   9300/tcp, 9650/tcp, 0.0.0.0:9201->9200/tcp, :::9201->9200/tcp, 0.0.0.0:9601->9600/tcp, :::9601->9600/tcp   os-node-01
   ```
1. OpenSearch 初始化集群所需的时间因底层主机的性能而异。你可以关注容器日志，了解 OpenSearch 在引导过程中执行的操作：
   1. 输入以下命令以在终端窗口中显示容器 `os-node-01` 的日志：
      ```bash
      docker logs -f os-node-01
      ```
      {% include copy.html %}
   1. 当节点准备就绪时，你将看到一个类似于以下示例的日志条目：
      <p class="codeblock-label">例</p>
      ```
      [INFO ][o.o.s.c.ConfigurationRepository] [os-node-01] Node 'os-node-01' initialized
      ```
   1. 按下 `Ctrl+C` 可停止关注容器日志并返回命令提示符。
1. 使用 cURL 查询 OpenSearch REST API。在以下命令中，通过将请求发送到主机端口进行查询， `os-node-01` 该端口映射到容器上的端口 `9201` `9200`：
   ```bash
   curl -s "https://localhost:9201" -ku admin:admin
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```json
   {
       "name" : "os-node-01",
       "cluster_name" : "opensearch-dev-cluster",
       "cluster_uuid" : "g1MMknuDRuuD9IaaNt56KA",
       "version" : {
           "distribution" : "opensearch",
           "number" : "1.3.7",
           "build_type" : "tar",
           "build_hash" : "db18a0d5a08b669fb900c00d81462e221f4438ee",
           "build_date" : "2022-12-07T22:59:20.186520Z",
           "build_snapshot" : false,
           "lucene_version" : "8.10.1",
           "minimum_wire_compatibility_version" : "6.8.0",
           "minimum_index_compatibility_version" : "6.0.0-beta1"
       },
       "tagline" : "The OpenSearch Project: https://opensearch.org/"
   }
   ```

**提示**：使用 `-s` with `curl` 选项隐藏进度表和错误消息。
{:.tip}

## 添加数据和配置 OpenSearch 安全性

现在 OpenSearch 集群正在运行，是时候添加数据并配置一些 OpenSearch 安全设置了。版本升级完成后，将再次验证你添加的数据和配置的设置。

本节可以分为两部分：
- [使用 REST API 为数据编制索引](#indexing-data-with-the-rest-api)
- [使用 OpenSearch 控制面板添加数据](#adding-data-using-opensearch-dashboards)

### 使用 REST API 为数据编制索引

1. 下载示例字段映射文件：
   ```bash
   wget https://raw.githubusercontent.com/opensearch-project/documentation-website/main/assets/examples/ecommerce-field_mappings.json
   ```
   {% include copy.html %}
1. 接下来，下载将引入此索引的批量数据：
   ```bash
   wget https://raw.githubusercontent.com/opensearch-project/documentation-website/main/assets/examples/ecommerce.json
   ```
   {% include copy.html %}
1. [创建索引]({{site.url}}{{site.baseurl}}/api-reference/index-apis/create-index/)使用 API 通过中 `ecommerce-field_mappings.json` 定义的映射创建索引：
   ```bash
   curl -H "Content-Type: application/x-ndjson" \
      -X PUT "https://localhost:9201/ecommerce?pretty" \
      --data-binary "@ecommerce-field_mappings.json" \
      -ku admin:admin
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```json
   {
      "acknowledged" : true,
      "shards_acknowledged" : true,
      "index" : "ecommerce"
   }
   ```
1. [Bulk]({{site.url}}{{site.baseurl}}/api-reference/document-apis/bulk/)使用 API 将数据添加到新的电子商务索引中 `ecommerce.json`：
   ```bash
   curl -H "Content-Type: application/x-ndjson" \
      -X PUT "https://localhost:9201/ecommerce/_bulk?pretty" \
      --data-binary "@ecommerce.json" \
      -ku admin:admin
   ```
   {% include copy.html %}
   <p class="codeblock-label">示例响应（截断）</p>
   ```json
   {
      "took" : 3323,
      "errors" : false,
      "items" : [
   ...
         "index" : {
            "_index" : "ecommerce",
            "_type" : "_doc",
            "_id" : "4674",
            "_version" : 1,
            "result" : "created",
            "_shards" : {
               "total" : 2,
               "successful" : 2,
               "failed" : 0
            },
            "_seq_no" : 4674,
            "_primary_term" : 1,
            "status" : 201
         }
      ]
   }
   ```
1. <p id="validation">搜索查询还可以确认数据已成功编制索引。以下查询返回关键字 `customer_first_name` 等于 `Sonya` 的文档数：</p>
   ```bash
   curl -H 'Content-Type: application/json' \
      -X GET "https://localhost:9201/ecommerce/_search?pretty=true&filter_path=hits.total" \
      -d'{"query":{"match":{"customer_first_name":"Sonya"}}}' \
      -ku admin:admin
   ```
   {% include copy.html %}
   <p class="codeblock-label" id="query-validation">响应示例</p>
   ```json
   {
      "hits" : {
         "total" : {
            "value" : 106,
            "relation" : "eq"
         }
      }
   }
   ```

### 使用 OpenSearch 控制面板添加数据

1. 打开 Web 浏览器并导航到 Docker 主机上的端口 `5601`（例如，<code>https://<var> HOST_ADDRESS</var>：5601</code>）。如果 OpenSearch 控制面板正在运行，并且你可以从浏览器客户端通过网络访问主机，则你将被重定向到登录页面。
    1. 如果 Web 浏览器因为 TLS 证书是自签名的而引发错误，则可能需要绕过浏览器中的证书检查。有关绕过证书检查的信息，请参阅浏览器的文档。每个证书的公用名（CN）是根据群集内通信的容器和节点名称生成的，因此从浏览器连接到主机仍会导致“CN 无效”警告。
1. 输入默认用户名（ `admin`）和密码（ `admin`）。
1. 在 OpenSearch 控制面板页面上**家**，选择**添加示例数据**。
1. 在下**示例 Web 日志**，选择**添加数据**。
   1. **自选**：选择以**查看数据**查看**[日志] 网络流量**仪表板。
1. **菜单按钮**选择以打开**导航窗格**，然后转到**Security > Internal users**。
1. 选择**创建内部用户**。
1. **用户名**提供和**密码**.
1. 在**后端角色**字段中，输入 `admin`。
1. 选择**创造**。

## 备份重要文件

在对集群进行更改之前，请始终创建备份，尤其是在集群在生产环境中运行时。

在本节中，你将：
- [注册快照存储库](#registering-a-snapshot-repository).
- [创建快照](#creating-a-snapshot).
- [备份安全设置](#backing-up-security-settings).

### 注册快照存储库

1. 使用映射的 `upgrade-demo-cluster.sh` 卷注册存储库：
   ```bash
   curl -H 'Content-Type: application/json' \
      -X PUT "https://localhost:9201/_snapshot/snapshot-repo?pretty" \
      -d '{"type":"fs","settings":{"location":"/usr/share/opensearch/snapshots"}}' \
      -ku admin:admin
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```json
   {
      "acknowledged" : true
   }
   ```
1. **自选**：执行其他检查以验证存储库是否已成功创建：
   ```bash
   curl -H 'Content-Type: application/json' \
      -X POST "https://localhost:9201/_snapshot/snapshot-repo/_verify?timeout=0s&master_timeout=50s&pretty" \
      -ku admin:admin
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```json
   {
      "nodes" : {
         "UODBXfAlRnueJ67grDxqgw" : {
            "name" : "os-node-03"
         },
         "14I_OyBQQXio8nmk0xsVcQ" : {
            "name" : "os-node-04"
         },
         "tQp3knPRRUqHvFNKpuD2vQ" : {
            "name" : "os-node-02"
         },
         "rPe8D6ssRgO5twIP00wbCQ" : {
            "name" : "os-node-01"
         }
      }
   }
   ```

### 创建快照

快照是集群索引和状态的备份。请参阅[Snapshots]({{site.url}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/snapshots/index/)以了解更多信息。

1. 创建包含所有索引和集群状态的快照：
   ```bash
   curl -H 'Content-Type: application/json' \
      -X PUT "https://localhost:9201/_snapshot/snapshot-repo/cluster-snapshot-v137?wait_for_completion=true&pretty" \
      -ku admin:admin
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```json
   {
      "snapshot" : {
         "snapshot" : "cluster-snapshot-v137",
         "uuid" : "-IYB8QNPShGOTnTtMjBjNg",
         "version_id" : 135248527,
         "version" : "1.3.7",
         "indices" : [
            "opensearch_dashboards_sample_data_logs",
            ".opendistro_security",
            "security-auditlog-2023.02.27",
            ".kibana_1",
            ".kibana_92668751_admin_1",
            "ecommerce",
            "security-auditlog-2023.03.06",
            "security-auditlog-2023.02.28",
            "security-auditlog-2023.03.07"
         ],
         "data_streams" : [ ],
         "include_global_state" : true,
         "state" : "SUCCESS",
         "start_time" : "2023-03-07T18:33:00.656Z",
         "start_time_in_millis" : 1678213980656,
         "end_time" : "2023-03-07T18:33:01.471Z",
         "end_time_in_millis" : 1678213981471,
         "duration_in_millis" : 815,
         "failures" : [ ],
         "shards" : {
            "total" : 9,
            "failed" : 0,
            "successful" : 9
         }
      }
   }
   ```

### 备份安全设置

集群管理员可以使用以下任一方法修改 OpenSearch 安全设置：

- 修改 YAML 文件并运行 `securityadmin.sh`
- 使用管理员证书发出 REST API 请求
- 使用 OpenSearch 控制面板进行更改

无论你选择哪种方法，OpenSearch Security 都会将你的配置写入名为 `.opendistro_security`.此系统索引在升级过程中保留，并且还保存在你创建的快照中。但是，还原系统索引需要证书授予 `admin` 的提升访问权限。要了解更多信息，请参阅[系统索引]({{site.url}}{{site.baseurl}}/security/configuration/system-indices/)和[配置 TLS 证书]({{site.url}}{{site.baseurl}}/security/configuration/tls/)。

你还可以通过 `securityadmin.sh` 在任何 OpenSearch 节点上运行该 `-backup` 选项，将 OpenSearch 安全设置导出为 YAML 文件。这些 YAML 文件可用于使用现有配置重新初始化 `.opendistro_security` 索引。以下步骤将指导你生成这些备份文件并将其复制到主机进行存储：

1. 使用以下命令 `os-node-01` 打开交互式伪 TTY 会话：
   ```bash
   docker exec -it os-node-01 bash
   ```
   {% include copy.html %}
1. 创建一个名为 `backups` 并导航到它的目录：
   ```bash
   mkdir /usr/share/opensearch/backups && cd /usr/share/opensearch/backups
   ```
   {% include copy.html %}
1. 用于 `securityadmin.sh` 在以下位置 `/usr/share/opensearch/backups/` 创建 OpenSearch 安全设置的备份：
   ```bash
   /usr/share/opensearch/plugins/opensearch-security/tools/securityadmin.sh \
      -backup /usr/share/opensearch/backups \
      -icl \
      -nhnv \
      -cacert /usr/share/opensearch/config/root-ca.pem \
      -cert /usr/share/opensearch/config/admin.pem \
      -key /usr/share/opensearch/config/admin-key.pem
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```bash
   Security Admin v7
   Will connect to localhost:9300 ... done
   Connected as CN=A,OU=DOCS,O=OPENSEARCH,L=PORTLAND,ST=OREGON,C=US
   OpenSearch Version: 1.3.7
   OpenSearch Security Version: 1.3.7.0
   Contacting opensearch cluster 'opensearch' and wait for YELLOW clusterstate ...
   Clustername: opensearch-dev-cluster
   Clusterstate: GREEN
   Number of nodes: 4
   Number of data nodes: 4
   .opendistro_security index already exists, so we do not need to create one.
   Will retrieve '/config' into /usr/share/opensearch/backups/config.yml 
      SUCC: Configuration for 'config' stored in /usr/share/opensearch/backups/config.yml
   Will retrieve '/roles' into /usr/share/opensearch/backups/roles.yml 
      SUCC: Configuration for 'roles' stored in /usr/share/opensearch/backups/roles.yml
   Will retrieve '/rolesmapping' into /usr/share/opensearch/backups/roles_mapping.yml 
      SUCC: Configuration for 'rolesmapping' stored in /usr/share/opensearch/backups/roles_mapping.yml
   Will retrieve '/internalusers' into /usr/share/opensearch/backups/internal_users.yml 
      SUCC: Configuration for 'internalusers' stored in /usr/share/opensearch/backups/internal_users.yml
   Will retrieve '/actiongroups' into /usr/share/opensearch/backups/action_groups.yml 
      SUCC: Configuration for 'actiongroups' stored in /usr/share/opensearch/backups/action_groups.yml
   Will retrieve '/tenants' into /usr/share/opensearch/backups/tenants.yml 
      SUCC: Configuration for 'tenants' stored in /usr/share/opensearch/backups/tenants.yml
   Will retrieve '/nodesdn' into /usr/share/opensearch/backups/nodes_dn.yml 
      SUCC: Configuration for 'nodesdn' stored in /usr/share/opensearch/backups/nodes_dn.yml
   Will retrieve '/whitelist' into /usr/share/opensearch/backups/whitelist.yml 
      SUCC: Configuration for 'whitelist' stored in /usr/share/opensearch/backups/whitelist.yml
   Will retrieve '/audit' into /usr/share/opensearch/backups/audit.yml 
      SUCC: Configuration for 'audit' stored in /usr/share/opensearch/backups/audit.yml
   ```
1. **自选**：创建 TLS 证书的备份目录并存储证书的副本。如果使用唯一的 TLS 证书，请对每个节点重复此操作：
   ```bash
   mkdir /usr/share/opensearch/backups/certs && cp /usr/share/opensearch/config/*pem /usr/share/opensearch/backups/certs/
   ```
   {% include copy.html %}
1. 终止伪 TTY 会话：
   ```bash
   exit
   ```
   {% include copy.html %}
1. 将文件复制到主机：
   ```bash
   docker cp os-node-01:/usr/share/opensearch/backups ~/deploy/
   ```
   {% include copy.html %}

## 执行升级

现在，群集已配置完毕，并且已备份重要文件和设置，可以开始版本升级了。

本节中包含的某些步骤（如禁用分片复制和刷新事务日志）不会影响群集的性能。这些步骤作为最佳实践包含在内，在客户端在整个升级过程中继续与 OpenSearch 集互（例如通过查询现有数据或索引文档）的情况下，可以显著提高集群性能。
{: .note}

1. 禁用分片复制以停止集群中 Lucene 索引段的移动：
   ```bash
   curl -H 'Content-type: application/json' \
      -X PUT "https://localhost:9201/_cluster/settings?pretty" \
      -d'{"persistent":{"cluster.routing.allocation.enable":"primaries"}}' \
      -ku admin:admin
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```json
   {
      "acknowledged" : true,
      "persistent" : {
         "cluster" : {
            "routing" : {
               "allocation" : {
                  "enable" : "primaries"
               }
            }
         }
      },
      "transient" : { }
   }
   ```
1. 在群集上执行刷新操作，以将事务日志条目提交到 Lucene 索引：
   ```bash
   curl -X POST "https://localhost:9201/_flush?pretty" -ku admin:admin
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```json
   {
      "_shards" : {
         "total" : 20,
         "successful" : 20,
         "failed" : 0
      }
   }
   ```
1. 选择要升级的节点。你可以按任何顺序升级节点，因为此演示集群中的所有节点都是符合条件的集群管理器。以下命令将停止并删除容器 `os-node-01`，而不删除已装载的数据卷：
   ```bash
   docker stop os-node-01 && docker container rm os-node-01
   ```
   {% include copy.html %}
1. 启动一个以 `opensearchproject/opensearch:2.5.0` 映像命名 `os-node-01` 的新容器，并使用与原始容器相同的映射卷：
   ```bash
   docker run -d \
      -p 9201:9200 -p 9601:9600 \
      -e "OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m" \
      --ulimit nofile=65536:65536 --ulimit memlock=-1:-1 \
      -v data-01:/usr/share/opensearch/data \
      -v repo-01:/usr/share/opensearch/snapshots \
      -v ~/deploy/opensearch-01.yml:/usr/share/opensearch/config/opensearch.yml \
      -v ~/deploy/root-ca.pem:/usr/share/opensearch/config/root-ca.pem \
      -v ~/deploy/admin.pem:/usr/share/opensearch/config/admin.pem \
      -v ~/deploy/admin-key.pem:/usr/share/opensearch/config/admin-key.pem \
      -v ~/deploy/os-node-01.pem:/usr/share/opensearch/config/os-node-01.pem \
      -v ~/deploy/os-node-01-key.pem:/usr/share/opensearch/config/os-node-01-key.pem \
      --network opensearch-dev-net \
      --ip 172.20.0.11 \
      --name os-node-01 \
      opensearchproject/opensearch:2.5.0
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```bash
   d26d0cb2e1e93e9c01bb00f19307525ef89c3c3e306d75913860e6542f729ea4
   ```
1. **自选**：查询集群以确定哪个节点充当集群管理器。在此过程中，你可以随时运行此命令，以查看何时选择了新的集群管理器：
   ```bash
   curl -s "https://localhost:9201/_cat/nodes?v&h=name,version,node.role,master" \
      -ku admin:admin | column -t
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```bash
   name        version  node.role  master
   os-node-01  2.5.0    dimr       -
   os-node-04  1.3.7    dimr       *
   os-node-02  1.3.7    dimr       -
   os-node-03  1.3.7    dimr       -
   ```
1. **自选**：查询集群以查看分片分配在删除和替换节点时的变化情况。在此过程中，你可以随时运行此命令，以查看分片状态如何更改：
   ```bash
   curl -s "https://localhost:9201/_cat/shards" \
      -ku admin:admin
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```bash
   security-auditlog-2023.03.06           0 p STARTED       53 214.5kb 172.20.0.13 os-node-03
   security-auditlog-2023.03.06           0 r UNASSIGNED                           
   .kibana_1                              0 p STARTED        3  14.5kb 172.20.0.12 os-node-02
   .kibana_1                              0 r STARTED        3  14.5kb 172.20.0.13 os-node-03
   ecommerce                              0 p STARTED     4675   3.9mb 172.20.0.12 os-node-02
   ecommerce                              0 r STARTED     4675   3.9mb 172.20.0.14 os-node-04
   security-auditlog-2023.03.07           0 p STARTED       37 175.7kb 172.20.0.14 os-node-04
   security-auditlog-2023.03.07           0 r UNASSIGNED                           
   .opendistro_security                   0 p STARTED       10  67.9kb 172.20.0.12 os-node-02
   .opendistro_security                   0 r STARTED       10  67.9kb 172.20.0.13 os-node-03
   .opendistro_security                   0 r STARTED       10  64.5kb 172.20.0.14 os-node-04
   .opendistro_security                   0 r UNASSIGNED                           
   security-auditlog-2023.02.27           0 p STARTED        4  80.5kb 172.20.0.12 os-node-02
   security-auditlog-2023.02.27           0 r UNASSIGNED                           
   security-auditlog-2023.02.28           0 p STARTED        6 104.1kb 172.20.0.14 os-node-04
   security-auditlog-2023.02.28           0 r UNASSIGNED                           
   opensearch_dashboards_sample_data_logs 0 p STARTED    14074   9.1mb 172.20.0.12 os-node-02
   opensearch_dashboards_sample_data_logs 0 r STARTED    14074   8.9mb 172.20.0.13 os-node-03
   .kibana_92668751_admin_1               0 r STARTED       33  37.3kb 172.20.0.13 os-node-03
   .kibana_92668751_admin_1               0 p STARTED       33  37.3kb 172.20.0.14 os-node-04
   ```
1. 停止 `os-node-02`：
   ```bash
   docker stop os-node-02 && docker container rm os-node-02
   ```
   {% include copy.html %}
1. 启动一个以 `opensearchproject/opensearch:2.5.0` 映像命名 `os-node-02` 的新容器，并使用与原始容器相同的映射卷：
   ```bash
   docker run -d \
      -p 9202:9200 -p 9602:9600 \
      -e "OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m" \
      --ulimit nofile=65536:65536 --ulimit memlock=-1:-1 \
      -v data-02:/usr/share/opensearch/data \
      -v repo-01:/usr/share/opensearch/snapshots \
      -v ~/deploy/opensearch-02.yml:/usr/share/opensearch/config/opensearch.yml \
      -v ~/deploy/root-ca.pem:/usr/share/opensearch/config/root-ca.pem \
      -v ~/deploy/admin.pem:/usr/share/opensearch/config/admin.pem \
      -v ~/deploy/admin-key.pem:/usr/share/opensearch/config/admin-key.pem \
      -v ~/deploy/os-node-02.pem:/usr/share/opensearch/config/os-node-02.pem \
      -v ~/deploy/os-node-02-key.pem:/usr/share/opensearch/config/os-node-02-key.pem \
      --network opensearch-dev-net \
      --ip 172.20.0.12 \
      --name os-node-02 \
      opensearchproject/opensearch:2.5.0
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```bash
   7b802865bd6eb420a106406a54fc388ed8e5e04f6cbd908c2a214ea5ce72ac00
   ```
1. 停止 `os-node-03`：
   ```bash
   docker stop os-node-03 && docker container rm os-node-03
   ```
   {% include copy.html %}
1. 启动一个以 `opensearchproject/opensearch:2.5.0` 映像命名 `os-node-03` 的新容器，并使用与原始容器相同的映射卷：
   ```bash
   docker run -d \
      -p 9203:9200 -p 9603:9600 \
      -e "OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m" \
      --ulimit nofile=65536:65536 --ulimit memlock=-1:-1 \
      -v data-03:/usr/share/opensearch/data \
      -v repo-01:/usr/share/opensearch/snapshots \
      -v ~/deploy/opensearch-03.yml:/usr/share/opensearch/config/opensearch.yml \
      -v ~/deploy/root-ca.pem:/usr/share/opensearch/config/root-ca.pem \
      -v ~/deploy/admin.pem:/usr/share/opensearch/config/admin.pem \
      -v ~/deploy/admin-key.pem:/usr/share/opensearch/config/admin-key.pem \
      -v ~/deploy/os-node-03.pem:/usr/share/opensearch/config/os-node-03.pem \
      -v ~/deploy/os-node-03-key.pem:/usr/share/opensearch/config/os-node-03-key.pem \
      --network opensearch-dev-net \
      --ip 172.20.0.13 \
      --name os-node-03 \
      opensearchproject/opensearch:2.5.0
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```bash
   d7f11726841a89eb88ff57a8cbecab392399f661a5205f0c81b60a995fc6c99d
   ```
1. 停止 `os-node-04`：
   ```bash
   docker stop os-node-04 && docker container rm os-node-04
   ```
   {% include copy.html %}
1. 启动一个以 `opensearchproject/opensearch:2.5.0` 映像命名 `os-node-04` 的新容器，并使用与原始容器相同的映射卷：
   ```bash
   docker run -d \
      -p 9204:9200 -p 9604:9600 \
      -e "OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m" \
      --ulimit nofile=65536:65536 --ulimit memlock=-1:-1 \
      -v data-04:/usr/share/opensearch/data \
      -v repo-01:/usr/share/opensearch/snapshots \
      -v ~/deploy/opensearch-04.yml:/usr/share/opensearch/config/opensearch.yml \
      -v ~/deploy/root-ca.pem:/usr/share/opensearch/config/root-ca.pem \
      -v ~/deploy/admin.pem:/usr/share/opensearch/config/admin.pem \
      -v ~/deploy/admin-key.pem:/usr/share/opensearch/config/admin-key.pem \
      -v ~/deploy/os-node-04.pem:/usr/share/opensearch/config/os-node-04.pem \
      -v ~/deploy/os-node-04-key.pem:/usr/share/opensearch/config/os-node-04-key.pem \
      --network opensearch-dev-net \
      --ip 172.20.0.14 \
      --name os-node-04 \
      opensearchproject/opensearch:2.5.0
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```bash
   26f8286ab11e6f8dcdf6a83c95f265172f9557578a1b292af84c6f5ef8738e1d
   ```
1. 确认你的集群正在运行新版本：
   ```bash
   curl -s "https://localhost:9201/_cat/nodes?v&h=name,version,node.role,master" \
      -ku admin:admin | column -t
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```bash
   name        version  node.role  master
   os-node-01  2.5.0    dimr       *
   os-node-02  2.5.0    dimr       -
   os-node-04  2.5.0    dimr       -
   os-node-03  2.5.0    dimr       -
   ```
1. 你应该升级的最后一个组件是 OpenSearch 控制面板节点。首先，停止并移除旧容器：
   ```bash
   docker stop os-dashboards-01 && docker rm os-dashboards-01
   ```
   {% include copy.html %}
1. 创建一个运行目标版本的 OpenSearch 控制面板的新容器：
   ```bash
   docker run -d \
      -p 5601:5601 --expose 5601 \
      -v ~/deploy/opensearch_dashboards.yml:/usr/share/opensearch-dashboards/config/opensearch_dashboards.yml \
      -v ~/deploy/root-ca.pem:/usr/share/opensearch-dashboards/config/root-ca.pem \
      -v ~/deploy/os-dashboards-01.pem:/usr/share/opensearch-dashboards/config/os-dashboards-01.pem \
      -v ~/deploy/os-dashboards-01-key.pem:/usr/share/opensearch-dashboards/config/os-dashboards-01-key.pem \
      --network opensearch-dev-net \
      --ip 172.20.0.10 \
      --name os-dashboards-01 \
      opensearchproject/opensearch-dashboards:2.5.0
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```bash
   310de7a24cf599ca0b39b241db07fa8865592ebe15b6f5fda26ad19d8e1c1e09
   ```
1. 确保 OpenSearch 控制面板容器正确启动。可以使用如下所示的命令来确认对 https:// HOST_ADDRESS：</var>5601</code>的<code>请求已重定向（HTTP 状态代码 302）到 `/app/login?`：<var>
   ```bash
   curl https://localhost:5601 -kI
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```bash
   HTTP/1.1 302 Found
   location: /app/login?
   osd-name: opensearch-dashboards-dev
   cache-control: private, no-cache, no-store, must-revalidate
   set-cookie: security_authentication=; Max-Age=0; Expires=Thu, 01 Jan 1970 00:00:00 GMT; Secure; HttpOnly; Path=/
   content-length: 0
   Date: Wed, 08 Mar 2023 15:36:53 GMT
   Connection: keep-alive
   Keep-Alive: timeout=120
   ```
1. 重新启用副本分片的分配：
   ```bash
   curl -H 'Content-type: application/json' \
      -X PUT "https://localhost:9201/_cluster/settings?pretty" \
      -d'{"persistent":{"cluster.routing.allocation.enable":"all"}}' \
      -ku admin:admin
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```json
   {
      "acknowledged" : true,
      "persistent" : {
         "cluster" : {
            "routing" : {
               "allocation" : {
                  "enable" : "all"
               }
            }
         }
      },
      "transient" : { }
   }
   ```

## 验证升级

你成功部署了安全的 OpenSearch 集群，索引了数据，创建了填充了示例数据的控制面板，创建了新的内部用户，备份了重要文件，并将集群从版本 1.3.7 升级到了 2.5.0. 在继续探索和试验 OpenSearch 和 OpenSearch 控制面板之前，你应验证升级的结果。

对于此群集，升级后验证步骤可以包括验证以下内容：

- [运行版本](#verifying-the-new-running-version)
- [运行状况和分片分配](#verifying-cluster-health-and-shard-allocation)
- [数据一致性](#verifying-data-consistency)

### 验证新的运行版本

1. 验证 OpenSearch 节点的当前运行版本：
   ```bash
   curl -s "https://localhost:9201/_cat/nodes?v&h=name,version,node.role,master" \
      -ku admin:admin | column -t
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```bash
   name        version  node.role  master
   os-node-01  2.5.0    dimr       *
   os-node-02  2.5.0    dimr       -
   os-node-04  2.5.0    dimr       -
   os-node-03  2.5.0    dimr       -
   ```
1. 验证当前正在运行的 OpenSearch 控制面板版本：
   1. **选项 1**：从 Web 界面验证 OpenSearch 控制面板版本。
      1. 打开 Web 浏览器并导航到 Docker 主机上的端口 `5601`（例如，<code>https://<var> HOST_ADDRESS</var>：5601</code>）。
      1. 使用默认用户名（ `admin`）和默认密码（ `admin`）登录。
      1. 选择右上角的**“帮助”按钮**。版本将显示在弹出窗口中。
      1. 再次选择以**“帮助”按钮**关闭弹出窗口。
   1. **选项 2**：通过检查 `manifest.yml` 来验证 OpenSearch 控制面板版本。
      1. 在命令行中，打开与 OpenSearch 控制面板容器的交互式伪 TTY 会话：
         ```bash
         docker exec -it os-dashboards-01 bash
         ```
         {% include copy.html %}
      1. 检查 `manifest.yml` 版本：
         ```bash
         head -n 5 manifest.yml 
         ```
         {% include copy.html %}
         <p class="codeblock-label">响应示例</p>
         ```bash
         ---
         schema-version: '1.1'
         build:
            name: OpenSearch Dashboards
            version: 2.5.0
         ```
      1. 终止伪 TTY 会话：
         ```bash
         exit
         ```
         {% include copy.html %}

### 验证集群运行状况和分片分配

1. 查询[群集运行状况]({{site.url}}{{site.baseurl}}/api-reference/cluster-api/cluster-health/) API 终端节点以查看有关集群运行状况的信息。你应看到状态 `green`，表示已分配所有主分片和副本分片：
   ```bash
   curl -s "https://localhost:9201/_cluster/health?pretty" -ku admin:admin
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```json
   {
      "cluster_name" : "opensearch-dev-cluster",
      "status" : "green",
      "timed_out" : false,
      "number_of_nodes" : 4,
      "number_of_data_nodes" : 4,
      "discovered_master" : true,
      "discovered_cluster_manager" : true,
      "active_primary_shards" : 16,
      "active_shards" : 36,
      "relocating_shards" : 0,
      "initializing_shards" : 0,
      "unassigned_shards" : 0,
      "delayed_unassigned_shards" : 0,
      "number_of_pending_tasks" : 0,
      "number_of_in_flight_fetch" : 0,
      "task_max_waiting_in_queue_millis" : 0,
      "active_shards_percent_as_number" : 100.0
   }
   ```
1. 查询[CAT shards]({{site.url}}{{site.baseurl}}/api-reference/cat/cat-shards/) API 终端节点，了解集群升级后分片的分配方式：
   ```bash
   curl -s "https://localhost:9201/_cat/shards" -ku admin:admin
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```bash
   security-auditlog-2023.02.27           0 r STARTED     4  80.5kb 172.20.0.13 os-node-03
   security-auditlog-2023.02.27           0 p STARTED     4  80.5kb 172.20.0.11 os-node-01
   security-auditlog-2023.03.08           0 p STARTED    30  95.2kb 172.20.0.13 os-node-03
   security-auditlog-2023.03.08           0 r STARTED    30 123.8kb 172.20.0.11 os-node-01
   ecommerce                              0 p STARTED  4675   3.9mb 172.20.0.12 os-node-02
   ecommerce                              0 r STARTED  4675   3.9mb 172.20.0.13 os-node-03
   .kibana_1                              0 p STARTED     3   5.9kb 172.20.0.12 os-node-02
   .kibana_1                              0 r STARTED     3   5.9kb 172.20.0.11 os-node-01
   .kibana_92668751_admin_1               0 p STARTED    33  37.3kb 172.20.0.13 os-node-03
   .kibana_92668751_admin_1               0 r STARTED    33  37.3kb 172.20.0.11 os-node-01
   opensearch_dashboards_sample_data_logs 0 p STARTED 14074   9.1mb 172.20.0.12 os-node-02
   opensearch_dashboards_sample_data_logs 0 r STARTED 14074   9.1mb 172.20.0.14 os-node-04
   security-auditlog-2023.02.28           0 p STARTED     6  26.2kb 172.20.0.11 os-node-01
   security-auditlog-2023.02.28           0 r STARTED     6  26.2kb 172.20.0.14 os-node-04
   .opendistro-reports-definitions        0 p STARTED     0    208b 172.20.0.12 os-node-02
   .opendistro-reports-definitions        0 r STARTED     0    208b 172.20.0.13 os-node-03
   .opendistro-reports-definitions        0 r STARTED     0    208b 172.20.0.14 os-node-04
   security-auditlog-2023.03.06           0 r STARTED    53 174.6kb 172.20.0.12 os-node-02
   security-auditlog-2023.03.06           0 p STARTED    53 174.6kb 172.20.0.14 os-node-04
   .kibana_101107607_newuser_1            0 r STARTED     1   5.1kb 172.20.0.13 os-node-03
   .kibana_101107607_newuser_1            0 p STARTED     1   5.1kb 172.20.0.11 os-node-01
   .opendistro_security                   0 r STARTED    10  64.5kb 172.20.0.12 os-node-02
   .opendistro_security                   0 r STARTED    10  64.5kb 172.20.0.13 os-node-03
   .opendistro_security                   0 r STARTED    10  64.5kb 172.20.0.11 os-node-01
   .opendistro_security                   0 p STARTED    10  64.5kb 172.20.0.14 os-node-04
   .kibana_-152937574_admintenant_1       0 r STARTED     1   5.1kb 172.20.0.12 os-node-02
   .kibana_-152937574_admintenant_1       0 p STARTED     1   5.1kb 172.20.0.14 os-node-04
   security-auditlog-2023.03.07           0 r STARTED    37 175.7kb 172.20.0.12 os-node-02
   security-auditlog-2023.03.07           0 p STARTED    37 175.7kb 172.20.0.14 os-node-04
   .kibana_92668751_admin_2               0 p STARTED    34  38.6kb 172.20.0.13 os-node-03
   .kibana_92668751_admin_2               0 r STARTED    34  38.6kb 172.20.0.11 os-node-01
   .kibana_2                              0 p STARTED     3     6kb 172.20.0.13 os-node-03
   .kibana_2                              0 r STARTED     3     6kb 172.20.0.14 os-node-04
   .opendistro-reports-instances          0 r STARTED     0    208b 172.20.0.12 os-node-02
   .opendistro-reports-instances          0 r STARTED     0    208b 172.20.0.11 os-node-01
   .opendistro-reports-instances          0 p STARTED     0    208b 172.20.0.14 os-node-04
   ```

### 验证数据一致性

你需要再次查询电子商务索引，以确认示例数据是否仍然存在：

1. 将对此查询的响应与在[last step](#validation)：[使用 REST API 为数据编制索引](#indexing-data-with-the-rest-api)
   ```bash
   curl -H 'Content-Type: application/json' \
      -X GET "https://localhost:9201/ecommerce/_search?pretty=true&filter_path=hits.total" \
      -d'{"query":{"match":{"customer_first_name":"Sonya"}}}' \
      -ku admin:admin
   ```
   {% include copy.html %}
   <p class="codeblock-label">响应示例</p>
   ```json
   {
      "hits" : {
         "total" : {
            "value" : 106,
            "relation" : "eq"
         }
      }
   }
   ```
1. 打开 Web 浏览器并导航到 Docker 主机上的端口 `5601`（例如，<code>https://<var> HOST_ADDRESS</var>：5601</code>）。
1. 输入默认用户名（ `admin`）和密码（ `admin`）。
1. 在 OpenSearch 控制面板页面上**家**，选择**菜单按钮** Web 界面左上角的以打开**导航窗格**.
1. 选择**挡泥板**。
1. 选择**[日志] 网络流量**打开在流程早期添加示例数据时创建的控制面板。
1. 查看完仪表板后，选择该**轮廓**按钮。选择此选项**登出**后，你就可以以其他用户身份登录。
1. 输入你在升级之前创建的用户名和密码，然后选择**登录**。

## 后续步骤

查看以下资源，了解有关 OpenSearch 工作原理的更多信息：

- [REST API 参考]({{site.url}}{{site.baseurl}}/api-reference/index/)
- [OpenSearch 控制面板快速入门指南]({{site.url}}{{site.baseurl}}/dashboards/quickstart-dashboards/)
- [关于 OpenSearch 中的安全性]({{site.url}}{{site.baseurl}}/security/index/)

