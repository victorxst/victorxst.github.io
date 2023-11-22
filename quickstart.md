---
layout: default
title: 快速开始
parent: OpenSearch documentation
nav_order: 3
redirect_from: 
  - /opensearch/install/quickstart/
---

# 快速入门

通过使用[Docker](https://www.docker.com/)部署容器，开始使用 OpenSearch 和 OpenSearch 控制面板。在继续操作之前，你需要在本地计算机上安装并[get Docker](https://docs.docker.com/get-docker/)[Docker Compose 的](https://github.com/docker/compose)安装。

本指南中使用的 Docker Compose 命令使用连字符书写（例如， `docker-compose`）。如果你在计算机上安装了 Docker Desktop，它会自动安装 Docker Compose 的捆绑版本，则应删除连字符。例如，更改为 `docker-compose` `docker compose`.{：.note}

## 启动集群

你需要一个称为 Compose 文件的特殊文件，Docker Compose 使用该文件在集群中定义和创建容器。OpenSearch 项目提供了一个示例 Compose 文件，你可以使用该文件开始使用。要了解有关使用 Compose 文件的更多信息，请查看官方[Compose 规范](https://docs.docker.com/compose/compose-file/).

1. 在计算机上运行 OpenSearch 之前，你应在主机上禁用内存分页和交换性能，以提高性能并增加 OpenSearch 可用的内存映射数量。有关详细信息，请参阅[重要的系统设置]({{site.url}}{{site.baseurl}}/opensearch/install/important-settings/)。
    ```bash
    # Disable memory paging and swapping.
    sudo swapoff -a
    
    # Edit the sysctl config file that defines the host's max map count.
    sudo vi /etc/sysctl.conf
    
    # Set max map count to the recommended value of 262144.
    vm.max_map_count=262144
    
    # Reload the kernel parameters.
    sudo sysctl -p
    ```
1. 将示例 Compose 文件下载到你的主机。你可以使用命令行实用程序（如 `curl` 和 `wget`）下载文件，也可以使用 Web 浏览器从 OpenSearch 项目文档网站存储库手动复制[docker-compose.yml](https://github.com/opensearch-project/documentation-website/blob/{{site.opensearch_major_minor_version}}/assets/examples/docker-compose.yml)。
    ```bash
    # Using cURL:
    curl -O https://raw.githubusercontent.com/opensearch-project/documentation-website/{{site.opensearch_major_minor_version}}/assets/examples/docker-compose.yml
    
    # Using wget:
    wget https://raw.githubusercontent.com/opensearch-project/documentation-website/{{site.opensearch_major_minor_version}}/assets/examples/docker-compose.yml
    ```
1. 在终端应用程序中，导航到包含 `docker-compose.yml` 刚刚下载的文件的目录，然后运行以下命令以创建群集并将其作为后台进程启动。
    ```bash
    docker-compose up -d
    ```
1. 使用命令 `docker-compose ps` 确认容器正在运行。你应看到如下所示的输出：
    ```bash
    $ docker-compose ps
    NAME                    COMMAND                  SERVICE                 STATUS              PORTS
    opensearch-dashboards   "./opensearch-dashbo…"   opensearch-dashboards   running             0.0.0.0:5601->5601/tcp
    opensearch-node1        "./opensearch-docker…"   opensearch-node1        running             0.0.0.0:9200->9200/tcp, 9300/tcp, 0.0.0.0:9600->9600/tcp, 9650/tcp
    opensearch-node2        "./opensearch-docker…"   opensearch-node2        running             9200/tcp, 9300/tcp, 9600/tcp, 9650/tcp
    ```
1. 查询 OpenSearch REST API 以验证服务是否正在运行。你应该使用 `-k`（也写为 `--insecure`）来禁用主机名检查，因为默认安全配置使用演示证书。用于 `-u` 传递默认用户名和密码（ `admin:admin`）。
    ```bash
    curl https://localhost:9200 -ku admin:admin
    ```
    响应示例：
    ```json
    {
        "name" : "opensearch-node1",
        "cluster_name" : "opensearch-cluster",
        "cluster_uuid" : "W0B8gPotTAajhMPbC9D4ww",
        "version" : {
            "distribution" : "opensearch",
            "number" : "2.6.0",
            "build_type" : "tar",
            "build_hash" : "7203a5af21a8a009aece1474446b437a3c674db6",
            "build_date" : "2023-02-24T18:58:37.352296474Z",
            "build_snapshot" : false,
            "lucene_version" : "9.5.0",
            "minimum_wire_compatibility_version" : "7.10.0",
            "minimum_index_compatibility_version" : "7.0.0"
        },
        "tagline" : "The OpenSearch Project: https://opensearch.org/"
    }
    ```
1. 通过在运行 OpenSearch 集群的同一主机上的 Web 浏览器中打开 `http://localhost:5601/`，探索 OpenSearch 控制面板。默认用户名是，默认密码是 `admin` `admin`。

## 使用示例数据创建索引和字段映射

使用 OpenSearch 项目提供的数据集创建索引并定义字段映射。相同的虚构电子商务数据也用于 OpenSearch 控制面板中的示例可视化。要了解更多信息，请参阅[开始使用 OpenSearch 控制面板]({{site.url}}{{site.baseurl}}/dashboards/index/)。

1. 下载[电子商务-field_mappings.json](https://github.com/opensearch-project/documentation-website/blob/{{site.opensearch_major_minor_version}}/assets/examples/ecommerce-field_mappings.json).此文件为将使用的示例数据定义。[mapping]({{site.url}}{{site.baseurl}}/opensearch/mappings/)
    ```bash
    # Using cURL:
    curl -O https://raw.githubusercontent.com/opensearch-project/documentation-website/{{site.opensearch_major_minor_version}}/assets/examples/ecommerce-field_mappings.json
    
    # Using wget:
    wget https://raw.githubusercontent.com/opensearch-project/documentation-website/{{site.opensearch_major_minor_version}}/assets/examples/ecommerce-field_mappings.json
    ```
1. 下载[电子商务.json](https://github.com/opensearch-project/documentation-website/blob/{{site.opensearch_major_minor_version}}/assets/examples/ecommerce.json).此文件包含格式化的索引数据，以便批量 API 可以引入这些数据。要了解更多信息，请参阅[index data]({{site.url}}{{site.baseurl}}/opensearch/index-data/)和[Bulk]({{site.url}}{{site.baseurl}}/api-reference/document-apis/bulk/)。
    ```bash
    # Using cURL:
    curl -O https://raw.githubusercontent.com/opensearch-project/documentation-website/{{site.opensearch_major_minor_version}}/assets/examples/ecommerce.json
    
    # Using wget:
    wget https://raw.githubusercontent.com/opensearch-project/documentation-website/{{site.opensearch_major_minor_version}}/assets/examples/ecommerce.json
    ```
1. 使用映射文件定义字段映射。
    ```bash
    curl -H "Content-Type: application/x-ndjson" -X PUT "https://localhost:9200/ecommerce" -ku admin:admin --data-binary "@ecommerce-field_mappings.json"
    ```
1. 将索引上传到批量 API。
    ```bash
    curl -H "Content-Type: application/x-ndjson" -X PUT "https://localhost:9200/ecommerce/_bulk" -ku admin:admin --data-binary "@ecommerce.json"
    ```
1. 使用搜索 API 查询数据。以下命令提交一个查询，该查询将返回文档，其中 `customer_first_name` is `Sonya`.
    ```bash
    curl -H 'Content-Type: application/json' -X GET "https://localhost:9200/ecommerce/_search?pretty=true" -ku admin:admin -d' {"query":{"match":{"customer_first_name":"Sonya"}}}'
    ```
    默认情况下，提交到 OpenSearch REST API 的查询通常将返回平面 JSON。对于人类可读的响应正文，请使用 query 参数 `pretty=true`。 `pretty` 有关查询参数和其他有用的查询参数的详细信息，请参阅[常用 REST 参数]({{site.url}}{{site.baseurl}}/opensearch/common-parameters/)。
1. 通过在运行 OpenSearch 集群的同一主机上的 Web 浏览器中打开 `http://localhost:5601/` OpenSearch 控制面板来访问 OpenSearch 控制面板。默认用户名是，默认密码是 `admin` `admin`。
1. 在顶部菜单栏上，转到**Management > Dev Tools**。
1. 在控制台的左窗格中，输入以下内容：
    ```json
    GET ecommerce/_search
    {
        "query": {
            "match": {
                "customer_first_name": "Sonya"
            }
        }
    }
    ```
1. 选择请求右上角的三角形图标以提交查询。你也可以通过按（或 `Cmd+Enter` Mac `Ctrl+Enter` 用户）来提交请求。要了解有关使用 OpenSearch 控制面板控制台提交查询的更多信息，请参阅[在控制台中运行查询]({{site.url}}{{site.baseurl}}/dashboards/run-queries/)。

## 后续步骤

你使用 OpenSearch 控制面板成功部署了自己的 OpenSearch 集群，并添加了一些示例数据。现在，你已准备好更详细地了解配置和功能。以下是关于从哪里开始的一些建议：
- [关于安全插件]({{site.url}}{{site.baseurl}}/security/index/)
- [OpenSearch 配置]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/)
- [OpenSearch 插件安装]({{site.url}}{{site.baseurl}}/opensearch/install/plugins/)
- [开始使用 OpenSearch 控制面板]({{site.url}}{{site.baseurl}}/dashboards/index/)
- [OpenSearch 工具]({{site.url}}{{site.baseurl}}/tools/index/)
- [Index APIs]({{site.url}}{{site.baseurl}}/api-reference/index-apis/index/)

## 常见问题

如果容器无法意外启动或退出，请查看这些常见问题和建议的解决方案。

### Docker 命令需要提升的权限

通过将用户添加到用户组， `docker` 无需运行 Docker 命令 `sudo`。有关详细信息，[Linux 的安装后步骤](https://docs.docker.com/engine/install/linux-postinstall/)请参阅 Docker。
```bash
sudo usermod -aG docker $USER
```

### 错误消息：“-bash：docker-compose：找不到命令”

如果安装了 Docker Desktop，则计算机上已安装 Docker Compose。尝试 `docker compose`（不带连字符）而不是 `docker-compose`.请参见[使用 Docker Compose](https://docs.docker.com/get-started/08_using_compose/)。

### 错误消息：“docker：'compose' 不是 docker 命令。

如果安装了 Docker Engine，则必须单独安装 Docker Compose，并且将使用该命令 `docker-compose`（带连字符）。请参见[Docker Compose 的](https://github.com/docker/compose)。

### 错误消息：“最大虚拟内存区域 vm.max_map_count [65530] 太低”

如果主机的 `vm.max_map_count` OpenSearch 过低，OpenSearch 将无法启动。如果在服务日志中看到以下错误，请查看，[重要的系统设置]({{site.url}}{{site.baseurl}}/opensearch/install/important-settings/)并进行适当的设置 `vm.max_map_count`。
```bash
opensearch-node1         | ERROR: [1] bootstrap checks failed
opensearch-node1         | [1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
opensearch-node1         | ERROR: OpenSearch did not exit normally - check the logs at /usr/share/opensearch/logs/opensearch-cluster.log
```