---
layout: default
title: Docker
parent: 安装 OpenSearch
nav_order: 5
redirect_from: 
  - /opensearch/install/docker/
  - /install-and-configure/install-opensearch/docker/
---

# Docker

[Docker](https://www.docker.com/)大大简化了配置和管理 OpenSearch 集群的过程。你可以使用本指南中包含的任何示例 Docker Compose 文件从[Docker Hub](https://hub.docker.com/u/opensearchproject)中提取官方映像，也可以[Amazon Elastic Container Registry （Amazon ECR）](https://gallery.ecr.aws/opensearchproject/)快速[Docker Compose 的](https://github.com/docker/compose)部署集群。有经验的 OpenSearch 用户可以通过创建自定义 Docker Compose 文件来进一步自定义其部署。

Docker 容器是可移植的，可以在任何支持 Docker 的兼容主机（例如 Linux、MacOS 或 Windows）上运行。Docker 容器的可移植性提供了优于其他安装方法（如[RPM]({{site.url}}{{site.baseurl}}/install-and-configure/install-opensearch/rpm/)手动[Tarball]({{site.url}}{{site.baseurl}}/install-and-configure/install-opensearch/tar/)安装）的灵活性，这两种安装方法都需要在下载和解压缩后进行额外配置。

本指南假定你能够熟练地使用 Linux 命令行界面（CLI）工作。你应该了解如何输入命令、在目录之间导航和编辑文本文件。如需帮助[Docker](https://www.docker.com/)或[Docker Compose 的](https://github.com/docker/compose)，请参阅其网站上的官方文档。
{: .note}

## 安装 Docker 和 Docker Compose

有关为你的环境安装和配置 Docker 的指南，请访问[Get Docker](https://docs.docker.com/get-docker/)。如果使用 CLI 安装 Docker 引擎，则默认情况下，Docker 不会对可用的主机资源有任何限制。根据你的环境，你可能希望在 Docker 中配置资源限制。有关信息，请参阅[具有内存、CPU 和 GPU 的运行时选项](https://docs.docker.com/config/containers/resource_constraints/)。

Docker Desktop 用户应通过打开 Docker Desktop 并选择**设置**→**资源**将主机内存利用率设置为至少 4 GB。
{:.tip}

Docker Compose 是一个实用程序，允许用户使用单个命令启动多个容器。调用文件时，将文件传递给 Docker Compose。Docker Compose 会读取这些设置并启动请求的容器。Docker Compose 随 Docker Desktop 一起自动安装，但在命令行环境中操作的用户必须手动安装 Docker Compose。你可以在官方[Docker Compose GitHub 页面](https://github.com/docker/compose)上找到有关安装 Docker Compose 的信息。

如果你需要手动安装 Docker Compose，并且你的主机支持 Python，则可以使用[pip](https://pypi.org/project/pip/)自动安装[Docker Compose 包](https://pypi.org/project/docker-compose/)。
{:.tip}

## 重要的主机设置

在启动 OpenSearch 之前，你应该查看一些[重要的系统设置]({{site.url}}{{site.baseurl}}/install-and-configure/install-opensearch/index/#important-settings){:target='\_blank'}可能影响服务性能的。

1. 禁用主机上的内存分页和交换性能以提高性能。
   ```bash
   sudo swapoff -a
   ```
1. 增加可用于 OpenSearch 的内存映射数。
   ```bash
   # Edit the sysctl config file
   sudo vi /etc/sysctl.conf

   # Add a line to define the desired value
   # or change the value if the key exists,
   # and then save your changes.
   vm.max_map_count=262144

   # Reload the kernel parameters using sysctl
   sudo sysctl -p

   # Verify that the change was applied by checking the value
   cat /proc/sys/vm/max_map_count
   ```

## 在 Docker 容器中运行 OpenSearch

官方 OpenSearch 映像托管在[Docker Hub](https://hub.docker.com/u/opensearchproject/)和[Amazon ECR](https://gallery.ecr.aws/opensearchproject/)上。如果要检查图像，可以使用单独拉取它们 `docker pull`，如以下示例所示。

[Docker Hub](https://hub.docker.com/u/opensearchproject/):
```bash
docker pull opensearchproject/opensearch:2
```
{% include copy.html %}

```bash
docker pull opensearchproject/opensearch-dashboards:2
```
{% include copy.html %}

[Amazon ECR](https://gallery.ecr.aws/opensearchproject/):
```bash
docker pull public.ecr.aws/opensearchproject/opensearch:2
```
{% include copy.html %}

```bash
docker pull public.ecr.aws/opensearchproject/opensearch-dashboards:2
```
{% include copy.html %}

要下载除最新可用版本之外的特定版本的 OpenSearch 或 OpenSearch 控制面板，请修改引用该标签的映像标签（在命令行或 Docker Compose 文件中）。例如， `opensearchproject/opensearch:{{site.opensearch_version}}` 将拉取 OpenSearch 版本{{site.opensearch_version}}。要拉取最新版本，请使用 `opensearchproject/opensearch:latest`.有关可用版本，请参阅官方映像存储库。
{:.tip}

在继续之前，你应通过在单个容器中部署 OpenSearch 来验证 Docker 是否正常工作。

1. 运行以下命令：
    ```bash
    # This command maps ports 9200 and 9600, sets the discovery type to "single-node" and requests the newest image of OpenSearch
    docker run -d -p 9200:9200 -p 9600:9600 -e "discovery.type=single-node" opensearchproject/opensearch:latest
    ```
1. 向端口 9200 发送请求。默认用户名和密码为 `admin`。
    ```bash
    curl https://localhost:9200 -ku 'admin:admin'
    ```
    {% include copy.html %}

    - 你应该得到如下所示的响应：
      ```bash
      {
        "name" : "a937e018cee5",
        "cluster_name" : "docker-cluster",
        "cluster_uuid" : "GLAjAG6bTeWErFUy_d-CLw",
        "version" : {
          "distribution" : "opensearch",
          "number" : <version>,
          "build_type" : <build-type>,
          "build_hash" : <build-hash>,
          "build_date" : <build-date>,
          "build_snapshot" : false,
          "lucene_version" : <lucene-version>,
          "minimum_wire_compatibility_version" : "7.10.0",
          "minimum_index_compatibility_version" : "7.0.0"
        },
        "tagline" : "The OpenSearch Project: https://opensearch.org/"
      }
      ```
1. 在停止正在运行的容器之前，请显示所有正在运行的容器的列表，并复制你正在测试的 OpenSearch 节点的容器 ID。在以下示例中，容器 ID 为 `a937e018cee5`：
    ```bash
    $ docker container ls
    CONTAINER ID   IMAGE                                 COMMAND                  CREATED          STATUS          PORTS                                                                NAMES
    a937e018cee5   opensearchproject/opensearch:latest   "./opensearch-docker…"   19 minutes ago   Up 19 minutes   0.0.0.0:9200->9200/tcp, 9300/tcp, 0.0.0.0:9600->9600/tcp, 9650/tcp   wonderful_boyd
    ```
1. 通过将容器 ID `docker stop` 传递给来停止正在运行的容器。
    ```bash
    docker stop <containerId>
    ```
    {% include copy.html %}

请记住，这 `docker container ls` 不会列出已停止的容器。如果要查看已停止的容器，请使用 `docker container ls -a`.你可以使用以下命令 `docker container rm <containerId_1> <containerId_2> <containerId_3> [...]` 手动删除不需要的容器（传递要停止的所有容器 ID，用空格分隔），或者如果要删除所有已停止的容器，则可以使用较短的命令 `docker container prune`。
{:.tip}

## 使用 Docker Compose 部署 OpenSearch 集群

尽管从技术上讲，可以通过一次一个命令创建容器来构建 OpenSearch 集群，但在 YAML 文件中定义环境并让 Docker Compose 管理集群要容易得多。以下部分包含示例 YAML 文件，你可以使用这些文件通过 OpenSearch 和 OpenSearch 控制面板启动预定义集群。这些示例对于测试和开发很有用，但不适用于生产环境。如果你之前没有使用 Docker Compose 的经验，你可能希望在对示例中的字典结构进行任何更改之前查看 Docker [Compose 规范](https://docs.docker.com/compose/compose-file/)以获取有关语法和格式的指导。

定义环境的 YAML 文件称为 Docker Compose 文件。默认情况下， `docker-compose` 命令将首先检查当前目录中是否有与以下任何名称匹配的文件：
-  `docker-compose.yml`
-  `docker-compose.yaml`
-  `compose.yml`
-  `compose.yaml`

如果当前目录中不存在这些文件， `docker-compose` 则该命令将失败。

使用标志 `-f` 调用 `docker-compose` 时，可以指定自定义文件位置和名称：
```bash
# Use a relative or absolute path to the file.
docker-compose -f /path/to/your-file.yml up
```

如果这是你首次使用 Docker Compose 启动 OpenSearch 集群，请使用以下示例 `docker-compose.yml` 文件。将其保存在主机的主目录中，并将其命名为 `docker-compose.yml`。此文件将创建一个包含三个容器的集群：两个运行 OpenSearch 服务的容器和一个运行 OpenSearch 控制面板的容器。这些容器将通过称为 `opensearch-net` 的桥接网络进行通信，并使用两个卷，每个 OpenSearch 节点一个卷。由于此文件未显式禁用演示安全配置，因此将安装自签名 TLS 证书，并创建具有默认名称和密码的内部用户。

### 示例 docker-compose.yml

```yml
version: '3'
services:
  opensearch-node1: # This is also the hostname of the container within the Docker network (i.e. https://opensearch-node1/)
    image: opensearchproject/opensearch:latest # Specifying the latest available image - modify if you want a specific version
    container_name: opensearch-node1
    environment:
      - cluster.name=opensearch-cluster # Name the cluster
      - node.name=opensearch-node1 # Name the node that will run in this container
      - discovery.seed_hosts=opensearch-node1,opensearch-node2 # Nodes to look for when discovering the cluster
      - cluster.initial_cluster_manager_nodes=opensearch-node1,opensearch-node2 # Nodes eligible to serve as cluster manager
      - bootstrap.memory_lock=true # Disable JVM heap memory swapping
      - "OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m" # Set min and max JVM heap sizes to at least 50% of system RAM
    ulimits:
      memlock:
        soft: -1 # Set memlock to unlimited (no soft or hard limit)
        hard: -1
      nofile:
        soft: 65536 # Maximum number of open files for the opensearch user - set to at least 65536
        hard: 65536
    volumes:
      - opensearch-data1:/usr/share/opensearch/data # Creates volume called opensearch-data1 and mounts it to the container
    ports:
      - 9200:9200 # REST API
      - 9600:9600 # Performance Analyzer
    networks:
      - opensearch-net # All of the containers will join the same Docker bridge network
  opensearch-node2:
    image: opensearchproject/opensearch:latest # This should be the same image used for opensearch-node1 to avoid issues
    container_name: opensearch-node2
    environment:
      - cluster.name=opensearch-cluster
      - node.name=opensearch-node2
      - discovery.seed_hosts=opensearch-node1,opensearch-node2
      - cluster.initial_cluster_manager_nodes=opensearch-node1,opensearch-node2
      - bootstrap.memory_lock=true
      - "OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    volumes:
      - opensearch-data2:/usr/share/opensearch/data
    networks:
      - opensearch-net
  opensearch-dashboards:
    image: opensearchproject/opensearch-dashboards:latest # Make sure the version of opensearch-dashboards matches the version of opensearch installed on other nodes
    container_name: opensearch-dashboards
    ports:
      - 5601:5601 # Map host port 5601 to container port 5601
    expose:
      - "5601" # Expose port 5601 for web access to OpenSearch Dashboards
    environment:
      OPENSEARCH_HOSTS: '["https://opensearch-node1:9200","https://opensearch-node2:9200"]' # Define the OpenSearch nodes that OpenSearch Dashboards will query
    networks:
      - opensearch-net

volumes:
  opensearch-data1:
  opensearch-data2:

networks:
  opensearch-net:
```
{% include copy.html %}

如果在撰写文件中使用环境变量覆盖 `opensearch_dashboards.yml` 设置，请使用全部大写字母，并将句点替换为下划线（例如，for `opensearch.hosts`、use `OPENSEARCH_HOSTS`）。此行为与覆盖设置不一致，在覆盖 `opensearch.yml` 设置中，转换只是对赋值运算符的更改（例如， `discovery.type: single-node` in `opensearch.yml` 定义为 `discovery.type=single-node` in `docker-compose.yml`）。
{: .note}

从主机的主目录（包含 `docker-compose.yml`）创建并以分离模式启动容器：
```bash
docker-compose up -d
```
{% include copy.html %}

验证服务容器是否正确启动：
```bash
docker-compose ps
```
{% include copy.html %}

如果容器启动失败，可以查看服务日志：
```bash
# If you don't pass a service name, docker-compose will show you logs from all of the nodes
docker-compose logs <serviceName>
```
{% include copy.html %}

通过从浏览器连接到http://localhost:5601 来验证对 OpenSearch 控制面板的访问。默认用户名和密码为 `admin`。在自定义部署的安全配置之前，建议不要在可从公共 Internet 访问的主机上使用此配置。

请记住，无法 `localhost` 远程访问。如果要将这些容器部署到远程主机，则需要建立网络连接，并替换为 `localhost` 与主机对应的 IP 或 DNS 记录。
{: .note}

停止集群中正在运行的容器：
```bash
docker-compose down
```
{% include copy.html %}

 `docker-compose down` 将停止正在运行的容器，但不会删除主机上存在的 Docker 卷。如果你不关心这些卷的内容，请使用 `-v` 删除所有卷的选项， `docker-compose down -v` 例如.
 {:.tip}

## 配置 OpenSearch

与需要大量安装后配置的 OpenSearch 的 RPM 分配不同，使用 Docker 运行 OpenSearch 集群允许你在创建容器之前定义环境。无论你使用 Docker 还是 Docker Compose，这都是可能的。

例如，请看以下命令：
```bash
docker run \
  -p 9200:9200 -p 9600:9600 \
  -e "discovery.type=single-node" \
  -v /path/to/custom-opensearch.yml:/usr/share/opensearch/config/opensearch.yml \
  opensearchproject/opensearch:latest
```
{% include copy.html %}

通过查看命令的每个部分，你可以看到它：
- 映射端口 `9200` 和 `9600`（ `HOST_PORT`： `CONTAINER_PORT`）。
- 设置为 `discovery.type` `single-node`，以便此单节点部署的引导程序检查不会失败。
- 使用[-v flag](https://docs.docker.com/engine/reference/commandline/run#mount-volume--v---read-only)将调用 `custom-opensearch.yml` 的本地文件传递到容器，替换 `opensearch.yml` 映像中包含的文件。
- 从 Docker Hub 请求 `opensearchproject/opensearch:latest` 映像。
- 运行容器。

如果将此命令与[示例 docker-compose.yml](#sample-docker-composeyml)文件进行比较，你可能会注意到一些常见设置，例如端口映射和映像引用。但是，该命令仅部署运行 OpenSearch 的单个容器，不会为 OpenSearch 控制面板创建容器。此外，如果要使用自定义 TLS 证书、用户或角色，或定义其他卷和网络，则此“单行”命令会迅速增长到不切实际的大小。这就是 Docker Compose 的实用性变得有用的地方。

当你使用 Docker Compose 构建 OpenSearch 集群时，你可能会发现将自定义配置文件从主机传递到容器比枚举中的每个单独设置 `docker-compose.yml` 更容易。与示例 `docker run` 命令使用 `-v` 标志将卷从主机挂载到容器的方式类似，复合文件可以将卷指定为相应服务的子选项。以下截断的 YAML 文件演示了如何将文件或目录装载到容器。有关卷使用和语法的全面信息，请参阅官方[volumes](https://docs.docker.com/storage/volumes/) Docker 文档。

```yml
services:
  opensearch-node1:
    volumes:
      - opensearch-data1:/usr/share/opensearch/data
      - ./custom-opensearch.yml:/usr/share/opensearch/config/opensearch.yml
  opensearch-node2:
    volumes:
      - opensearch-data2:/usr/share/opensearch/data
      - ./custom-opensearch.yml:/usr/share/opensearch/config/opensearch.yml
  opensearch-dashboards:
    volumes:
      - ./custom-opensearch_dashboards.yml:/usr/share/opensearch-dashboards/config/opensearch_dashboards.yml
```
{% include copy.html %}

### 用于开发的示例 Docker Compose 文件

如果要从示例生成自己的复合文件，请查看以下示例 `docker-compose.yml` 文件。此示例文件创建两个 OpenSearch 节点和一个禁用了安全插件的 OpenSearch 控制面板节点。在查看[配置基本安全设置](#configuring-basic-security-settings)时，可以使用此示例文件作为起点。
```yml
version: '3'
services:
  opensearch-node1:
    image: opensearchproject/opensearch:latest
    container_name: opensearch-node1
    environment:
      - cluster.name=opensearch-cluster # Name the cluster
      - node.name=opensearch-node1 # Name the node that will run in this container
      - discovery.seed_hosts=opensearch-node1,opensearch-node2 # Nodes to look for when discovering the cluster
      - cluster.initial_cluster_manager_nodes=opensearch-node1,opensearch-node2 # Nodes eligibile to serve as cluster manager
      - bootstrap.memory_lock=true # Disable JVM heap memory swapping
      - "OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m" # Set min and max JVM heap sizes to at least 50% of system RAM
      - "DISABLE_INSTALL_DEMO_CONFIG=true" # Prevents execution of bundled demo script which installs demo certificates and security configurations to OpenSearch
      - "DISABLE_SECURITY_PLUGIN=true" # Disables Security plugin
    ulimits:
      memlock:
        soft: -1 # Set memlock to unlimited (no soft or hard limit)
        hard: -1
      nofile:
        soft: 65536 # Maximum number of open files for the opensearch user - set to at least 65536
        hard: 65536
    volumes:
      - opensearch-data1:/usr/share/opensearch/data # Creates volume called opensearch-data1 and mounts it to the container
    ports:
      - 9200:9200 # REST API
      - 9600:9600 # Performance Analyzer
    networks:
      - opensearch-net # All of the containers will join the same Docker bridge network
  opensearch-node2:
    image: opensearchproject/opensearch:latest
    container_name: opensearch-node2
    environment:
      - cluster.name=opensearch-cluster # Name the cluster
      - node.name=opensearch-node2 # Name the node that will run in this container
      - discovery.seed_hosts=opensearch-node1,opensearch-node2 # Nodes to look for when discovering the cluster
      - cluster.initial_cluster_manager_nodes=opensearch-node1,opensearch-node2 # Nodes eligibile to serve as cluster manager
      - bootstrap.memory_lock=true # Disable JVM heap memory swapping
      - "OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m" # Set min and max JVM heap sizes to at least 50% of system RAM
      - "DISABLE_INSTALL_DEMO_CONFIG=true" # Prevents execution of bundled demo script which installs demo certificates and security configurations to OpenSearch
      - "DISABLE_SECURITY_PLUGIN=true" # Disables Security plugin
    ulimits:
      memlock:
        soft: -1 # Set memlock to unlimited (no soft or hard limit)
        hard: -1
      nofile:
        soft: 65536 # Maximum number of open files for the opensearch user - set to at least 65536
        hard: 65536
    volumes:
      - opensearch-data2:/usr/share/opensearch/data # Creates volume called opensearch-data2 and mounts it to the container
    networks:
      - opensearch-net # All of the containers will join the same Docker bridge network
  opensearch-dashboards:
    image: opensearchproject/opensearch-dashboards:latest
    container_name: opensearch-dashboards
    ports:
      - 5601:5601 # Map host port 5601 to container port 5601
    expose:
      - "5601" # Expose port 5601 for web access to OpenSearch Dashboards
    environment:
      - 'OPENSEARCH_HOSTS=["http://opensearch-node1:9200","http://opensearch-node2:9200"]'
      - "DISABLE_SECURITY_DASHBOARDS_PLUGIN=true" # disables security dashboards plugin in OpenSearch Dashboards
    networks:
      - opensearch-net

volumes:
  opensearch-data1:
  opensearch-data2:

networks:
  opensearch-net:
```
{% include copy.html %}

### 配置基本安全设置

在将 OpenSearch 集群提供给外部主机之前，最好先查看部署的安全配置。你可能还记得，从第一个[示例 docker-compose.yml](#sample-docker-composeyml)文件中可以看出，除非通过设置 `DISABLE_SECURITY_PLUGIN=true` 禁用，否则捆绑的脚本会将默认的演示安全配置应用于集群中的节点。由于此配置用于演示目的，因此默认用户名和密码是已知的。因此，我们建议你创建自己的安全配置文件，并用于 `volumes` 将这些文件传递到容器。有关 OpenSearch 安全设置的具体指导，请参阅[安全配置]({{site.url}}{{site.baseurl}}/security/configuration/index/)。

若要在配置中使用自己的证书，请将所有必需的证书添加到撰写文件的卷部分：
```yml
volumes:
  - ./root-ca.pem:/usr/share/opensearch/config/root-ca.pem
  - ./admin.pem:/usr/share/opensearch/config/admin.pem
  - ./admin-key.pem:/usr/share/opensearch/config/admin-key.pem
  - ./node1.pem:/usr/share/opensearch/config/node1.pem
  - ./node1-key.pem:/usr/share/opensearch/config/node1-key.pem
```
{% include copy.html %}

当你使用 Docker Compose 卷将 TLS 证书添加到 OpenSearch 节点时，你还应该包含定义这些证书的自定义 `opensearch.yml` 文件。例如：
```yml
volumes:
  - ./root-ca.pem:/usr/share/opensearch/config/root-ca.pem
  - ./admin.pem:/usr/share/opensearch/config/admin.pem
  - ./admin-key.pem:/usr/share/opensearch/config/admin-key.pem
  - ./node1.pem:/usr/share/opensearch/config/node1.pem
  - ./node1-key.pem:/usr/share/opensearch/config/node1-key.pem
  - ./custom-opensearch.yml:/usr/share/opensearch/config/opensearch.yml
```
{% include copy.html %}

请记住，在撰写文件中指定的证书必须与自定义 `opensearch.yml` 文件中定义的证书相同。你应该将根证书、管理员证书和节点证书替换为你自己的证书。有关详细信息，请参阅[配置 TLS 证书]({{site.url}}{{site.baseurl}}/security/configuration/tls)。
```yml
plugins.security.ssl.transport.pemcert_filepath: node1.pem
plugins.security.ssl.transport.pemkey_filepath: node1-key.pem
plugins.security.ssl.transport.pemtrustedcas_filepath: root-ca.pem
plugins.security.ssl.http.pemcert_filepath: node1.pem
plugins.security.ssl.http.pemkey_filepath: node1-key.pem
plugins.security.ssl.http.pemtrustedcas_filepath: root-ca.pem
plugins.security.authcz.admin_dn:
  - CN=admin,OU=SSL,O=Test,L=Test,C=DE
```
{% include copy.html %}

配置安全设置后，自定义 `opensearch.yml` 文件可能类似于以下示例，该示例添加 TLS 证书和管理员证书的可分辨名称（DN），定义一些权限，并启用详细的审核日志记录：
```yml
plugins.security.ssl.transport.pemcert_filepath: node1.pem
plugins.security.ssl.transport.pemkey_filepath: node1-key.pem
plugins.security.ssl.transport.pemtrustedcas_filepath: root-ca.pem
plugins.security.ssl.transport.enforce_hostname_verification: false
plugins.security.ssl.http.enabled: true
plugins.security.ssl.http.pemcert_filepath: node1.pem
plugins.security.ssl.http.pemkey_filepath: node1-key.pem
plugins.security.ssl.http.pemtrustedcas_filepath: root-ca.pem
plugins.security.allow_default_init_securityindex: true
plugins.security.authcz.admin_dn:
  - CN=A,OU=UNIT,O=ORG,L=TORONTO,ST=ONTARIO,C=CA
plugins.security.nodes_dn:
  - 'CN=N,OU=UNIT,O=ORG,L=TORONTO,ST=ONTARIO,C=CA'
plugins.security.audit.type: internal_opensearch
plugins.security.enable_snapshot_restore_privilege: true
plugins.security.check_snapshot_restore_write_privileges: true
plugins.security.restapi.roles_enabled: ["all_access", "security_rest_api_access"]
cluster.routing.allocation.disk.threshold_enabled: false
opendistro_security.audit.config.disabled_rest_categories: NONE
opendistro_security.audit.config.disabled_transport_categories: NONE
```
{% include copy.html %}

有关设置的完整列表，请参见[Security]({{site.url}}{{site.baseurl}}/security/configuration/index/)。

使用相同的过程在[后端配置]({{site.url}}{{site.baseurl}}/security/configuration/configuration/) `/usr/share/opensearch/config/opensearch-security/config.yml` 各自[YAML files]({{site.url}}{{site.baseurl}}/security/configuration/yaml/)的.

替换证书并创建自己的内部用户、角色、映射、操作组和租户后，使用 Docker Compose 启动群集：
```bash
docker-compose up -d
```
{% include copy.html %}

### 使用插件

要将 OpenSearch 映像与自定义插件一起使用，你必须先创建一个 [ `Dockerfile`]（https://docs.docker.com/engine/reference/builder/）。有关创建 Dockerfile 的信息，请查看官方 Docker 文档。
```
FROM opensearchproject/opensearch:latest
RUN /usr/share/opensearch/bin/opensearch-plugin install --batch <pluginId>
```

然后运行以下命令：
```bash
# Build an image from a Dockerfile
docker build --tag=opensearch-custom-plugin .
# Start the container from the custom image
docker run -p 9200:9200 -p 9600:9600 -v /usr/share/opensearch/data opensearch-custom-plugin
```

或者，你可能希望在部署之前从映像中删除插件。此示例 Dockerfile 删除了 Security 插件：
```
FROM opensearchproject/opensearch:latest
RUN /usr/share/opensearch/bin/opensearch-plugin remove opensearch-security
```
{% include copy.html %}

你还可以使用 Dockerfile 传递你自己的证书，以便与：[安全插件]({{site.url}}{{site.baseurl}}/security/)
```
FROM opensearchproject/opensearch:latest
COPY --chown=opensearch:opensearch opensearch.yml /usr/share/opensearch/config/
COPY --chown=opensearch:opensearch my-key-file.pem /usr/share/opensearch/config/
COPY --chown=opensearch:opensearch my-certificate-chain.pem /usr/share/opensearch/config/
COPY --chown=opensearch:opensearch my-root-cas.pem /usr/share/opensearch/config/
```
{% include copy.html %}

## 相关链接

- [OpenSearch 配置]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/)
- [性能分析器]({{site.url}}{{site.baseurl}}/monitoring-plugins/pa/index/)
- [安装和配置 OpenSearch 控制面板]({{site.url}}{{site.baseurl}}/install-and-configure/install-dashboards/index/)
- [关于 OpenSearch 中的安全性]({{site.url}}{{site.baseurl}}/security/index/)
