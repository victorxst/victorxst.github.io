---
layout: default
title: RPM
parent: 安装 OpenSearch
redirect_from:
- /opensearch/install/rpm/
nav_order: 51
---

{% comment %} The following liquid syntax declares a variable, major_version_mask, which is transformed into "N.x" where "N" is the major version number. This is required for proper versioning references to the Yum repo. {% endcomment %} {% assign version_parts = site.opensearch_major_minor_version | split: "." %} {% assign major_version_mask = version_parts[0] | append: ".x" %}

# 转速

与该方法相比，[Tarball]({{site.url}}{{site.baseurl}}/opensearch/install/tar/)使用 RPM Package Manager（RPM）安装 OpenSearch 大大简化了该过程。一些技术注意事项，例如安装路径、配置文件的位置以及创建由管理的服务，例如，由 `systemd` 包管理器自动处理。

一般来说，从 RPM 分配安装 OpenSearch 可以分为几个步骤：

1. **下载并安装 OpenSearch。**
   - 从 RPM 软件包或 YUM 存储库手动安装。
1. **（可选）测试 OpenSearch。**
   - 在应用任何自定义配置之前，请确认 OpenSearch 能够运行。
   - 这可以在没有任何安全性（无密码、无证书）的情况下完成，也可以使用可由打包脚本应用的演示安全配置来完成。
1. **为你的环境配置 OpenSearch。**
   -  将基本设置应用于 OpenSearch，并开始在你的环境中使用它。

RPM 发行版提供了在 Red Hat 或基于 Red Hat 的 Linux 发行版中运行 OpenSearch 所需的一切。有关支持的操作系统的列表，请参见[操作系统兼容性]({{site.url}}{{site.baseurl}}/install-and-configure/install-opensearch/index/#operating-system-compatibility)。

本指南假定你能够熟练地使用 Linux 命令行界面（CLI）工作。你应该了解如何输入命令、在目录之间导航和编辑文本文件。一些示例命令引用 `vi` 文本编辑器，但你可以使用任何可用的文本编辑器。{:.note}

## 步骤 1：下载并安装 OpenSearch

### 从软件包安装 OpenSearch

1. 直接从[OpenSearch 下载页面](https://opensearch.org/downloads.html){:target='\_blank'} 下载所需版本的 RPM 包。RPM 包可以同时下载和**64 倍****arm64 的**架构。
1. 导入 GNU 隐私卫士（GPG）公钥。此密钥验证你的 OpenSearch 实例是否已签名。
    ```bash
    sudo rpm --import https://artifacts.opensearch.org/publickeys/opensearch.pgp
    ```
    {% include copy.html %}

1. 在 CLI 中，你可以使用 `rpm` 或 `yum` 安装软件包。
   ```bash
   # Install the x64 package using yum.
   sudo yum install opensearch-{{site.opensearch_version}}-linux-x64.rpm

   # Install the x64 package using rpm.
   sudo rpm -ivh opensearch-{{site.opensearch_version}}-linux-x64.rpm

   # Install the arm64 package using yum.
   sudo yum install opensearch-{{site.opensearch_version}}-linux-x64.rpm

   # Install the arm64 package using rpm.
   sudo rpm -ivh opensearch-{{site.opensearch_version}}-linux-x64.rpm
   ```
1. 安装成功后，启用 OpenSearch 即服务。
    ```bash
    sudo systemctl enable opensearch
    ```
    {% include copy.html %}

1. 启动 OpenSearch。
    ```bash
    sudo systemctl start opensearch
    ```
    {% include copy.html %}

1. 验证 OpenSearch 是否正确启动。
    ```bash
    sudo systemctl status opensearch
    ```
    {% include copy.html %}

### 从 YUM 存储库安装 OpenSearch

YUM 是基于 Red Hat 的操作系统的主要软件包管理工具，允许你从 YUM 存储库下载和安装 RPM 软件包。

1. 为 OpenSearch 创建本地存储库文件：
   ```bash
   sudo curl -SL https://artifacts.opensearch.org/releases/bundle/opensearch/{{major_version_mask}}/opensearch-{{major_version_mask}}.repo -o /etc/yum.repos.d/opensearch-{{major_version_mask}}.repo
   ```
   {% include copy.html %}

1. 清理 YUM 缓存以确保顺利安装：
   ```bash
   sudo yum clean all
   ```
   {% include copy.html %}

1. 验证存储库是否已成功创建。
    ```bash
    sudo yum repolist
    ```
    {% include copy.html %}

1. 下载存储库文件后，列出 OpenSearch 的所有可用版本：
   ```bash
   sudo yum list opensearch --showduplicates
   ```
   {% include copy.html %}

1. 选择要安装的 OpenSearch 版本：
   - 除非另有说明，否则将安装最新版本的 OpenSearch。
   ```bash
   sudo yum install opensearch
   ```
   {% include copy.html %}

   - 要安装特定版本的 OpenSearch，请执行以下操作：
   ```bash
   sudo yum install 'opensearch-{{site.opensearch_version}}'
   ```
   {% include copy.html %}

1. 在安装过程中，安装程序将向你显示 GPG 密钥指纹。验证信息是否与以下内容匹配：
   ```bash
   Fingerprint: c5b7 4989 65ef d1c2 924b a9d5 39d3 1987 9310 d3fc
   ```
   {% include copy.html %}

    - 如果正确，请输入 `yes` 或 `y`。OpenSearch 安装将继续。
1. 完成后，你可以运行 OpenSearch。
    ```bash
    sudo systemctl start opensearch
    ```
    {% include copy.html %}

1. 验证 OpenSearch 是否正确启动。
    ```bash
    sudo systemctl status opensearch
    ```
    {% include copy.html %}

## 步骤 2：（可选）测试 OpenSearch

在继续进行任何配置之前，你应测试 OpenSearch 的安装。否则，可能很难确定将来的问题是由安装问题还是安装后应用的自定义设置引起的。

使用 RPM 软件包安装 OpenSearch 时，会自动应用一些演示安全设置。这包括自签名 TLS 证书以及多个用户和角色。如果要自行配置这些配置，请参见[在你的环境中设置 OpenSearch](#step-3-set-up-opensearch-in-your-environment)。

默认配置中的 OpenSearch 节点（具有演示证书和具有默认密码的用户）不适合生产环境。如果你计划在生产环境中使用该节点，那么至少应将演示 TLS 证书替换为你自己的 TLS 证书和[更新内部用户和密码列表]({{site.url}}{{site.baseurl}}/security/configuration/yaml)。有关其他指导，请参阅以确保[安全配置]({{site.url}}{{site.baseurl}}/security/configuration/index/)根据安全要求配置节点。{: .warning}

1. 向服务器发送请求以验证 OpenSearch 是否正在运行。请注意该 `--insecure` 标志的使用，这是必需的，因为 TLS 证书是自签名的。
   - 向端口 9200 发送请求：
      ```bash
      curl -X GET https://localhost:9200 -u 'admin:admin' --insecure
      ```
      {% include copy.html %}

      你应该得到如下所示的响应：
      ```bash
      {
         "name" : "hostname",
         "cluster_name" : "opensearch",
         "cluster_uuid" : "6XNc9m2gTUSIoKDqJit0PA",
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
   - 查询插件端点：
      ```bash
      curl -X GET https://localhost:9200/_cat/plugins?v -u 'admin:admin' --insecure
      ```
      {% include copy.html %}

      响应应如下所示：
      ```bash
      name     component                            version
      hostname opensearch-alerting                  {{site.opensearch_version}}
      hostname opensearch-anomaly-detection         {{site.opensearch_version}}
      hostname opensearch-asynchronous-search       {{site.opensearch_version}}
      hostname opensearch-cross-cluster-replication {{site.opensearch_version}}
      hostname opensearch-index-management          {{site.opensearch_version}}
      hostname opensearch-job-scheduler             {{site.opensearch_version}}
      hostname opensearch-knn                       {{site.opensearch_version}}
      hostname opensearch-ml                        {{site.opensearch_version}}
      hostname opensearch-notifications             {{site.opensearch_version}}
      hostname opensearch-notifications-core        {{site.opensearch_version}}
      hostname opensearch-observability             {{site.opensearch_version}}
      hostname opensearch-performance-analyzer      {{site.opensearch_version}}
      hostname opensearch-reports-scheduler         {{site.opensearch_version}}
      hostname opensearch-security                  {{site.opensearch_version}}
      hostname opensearch-sql                       {{site.opensearch_version}}
      ```

## 步骤 3：在你的环境中设置 OpenSearch

之前没有 OpenSearch 经验的用户可能需要推荐设置列表才能开始使用该服务。默认情况下，OpenSearch 未绑定到网络接口，外部主机无法访问。此外，安全设置由默认用户名和密码填充。以下建议将使用户能够将 OpenSearch 绑定到网络接口、创建和签署 TLS 证书以及配置基本身份验证。

以下建议的设置将允许你：

- 将 OpenSearch 绑定到主机上的 IP 或网络接口。
- 设置初始和最大 JVM 堆大小。
- 定义一个指向捆绑的 JDK 的环境变量。
- 配置你自己的 TLS 证书 - 不需要第三方证书颁发机构（CA）。
- 使用自定义密码创建管理员用户。

如果运行了安全演示脚本，则需要手动重新配置已修改的设置。在继续操作之前，[安全配置]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/)请参阅有关指导。{:.note}

在修改任何配置文件之前，最好先保存备份副本，然后再进行更改。备份文件可用于缓解由错误配置引起的任何问题。{:.tip}

1. 打开 `opensearch.yml`。
   ```bash
   sudo vi /etc/opensearch/opensearch.yml
   ```
   {% include copy.html %}

1. 添加以下行：
   ```bash
   # Bind OpenSearch to the correct network interface. Use 0.0.0.0
   # to include all available interfaces or specify an IP address
   # assigned to a specific interface.
   network.host: 0.0.0.0

   # Unless you have already configured a cluster, you should set
   # discovery.type to single-node, or the bootstrap checks will
   # fail when you try to start the service.
   discovery.type: single-node

   # If you previously disabled the Security plugin in opensearch.yml,
   # be sure to re-enable it. Otherwise you can skip this setting.
   plugins.security.disabled: false
   ```
   {% include copy.html %}

1. 保存更改并关闭文件。
1. 指定初始和最大 JVM 堆大小。
   1.  打开 `jvm.options`。
         ```bash
         vi /etc/opensearch/jvm.options
         ```
         {% include copy.html %}

   1. 修改初始堆大小和最大堆大小的值。首先，应将这些值设置为可用系统内存的一半。对于专用主机，可以根据你的工作流要求增加此值。
      -  例如，如果主机具有 8 GB 的内存，则可能需要将初始堆大小和最大堆大小设置为 4 GB：
         ```bash
         -Xms4g
         -Xmx4g
         ```
         {% include copy.html %}

   1. 保存更改并关闭文件。

### 配置 TLS

TLS 证书允许客户端确认主机的身份并加密客户端与主机之间的流量，从而为群集提供额外的安全性。有关更多信息，请参阅[配置 TLS 证书]({{site.url}}{{site.baseurl}}/security/configuration/tls/)文档中包含的[安全插件]({{site.url}}{{site.baseurl}}/security/index/)和[生成证书]({{site.url}}{{site.baseurl}}/security/configuration/generate-certificates/)。对于在开发环境中执行的工作，自签名证书通常就足够了。本部分将指导你完成生成自己的 TLS 证书并将其应用于 OpenSearch 主机所需的基本步骤。

1. 导航到将存储证书的目录。
   ```bash
   cd /etc/opensearch
   ```
   {% include copy.html %}

1. 删除演示证书。
   ```bash
   sudo rm -f *pem
   ```
   {% include copy.html %}

1. 生成根证书。这是你将用于对其他证书进行签名的内容。
   ```bash
   # Create a private key for the root certificate
   sudo openssl genrsa -out root-ca-key.pem 2048
   
   # Use the private key to create a self-signed root certificate. Be sure to
   # replace the arguments passed to -subj so they reflect your specific host.
   sudo openssl req -new -x509 -sha256 -key root-ca-key.pem -subj "/C=CA/ST=ONTARIO/L=TORONTO/O=ORG/OU=UNIT/CN=ROOT" -out root-ca.pem -days 730
   ```
1. 接下来，创建管理员证书。此证书用于获得执行与安全插件相关的管理任务的提升权限。
   ```bash
   # Create a private key for the admin certificate.
   sudo openssl genrsa -out admin-key-temp.pem 2048

   # Convert the private key to PKCS#8.
   sudo openssl pkcs8 -inform PEM -outform PEM -in admin-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out admin-key.pem
   
   # Create the certficiate signing request (CSR). A common name (CN) of "A" is acceptable because this certificate is
   # used for authenticating elevated access and is not tied to a host.
   sudo openssl req -new -key admin-key.pem -subj "/C=CA/ST=ONTARIO/L=TORONTO/O=ORG/OU=UNIT/CN=A" -out admin.csr
   
   # Sign the admin certificate with the root certificate and private key you created earlier.
   sudo openssl x509 -req -in admin.csr -CA root-ca.pem -CAkey root-ca-key.pem -CAcreateserial -sha256 -out admin.pem -days 730
   ```
1. 为正在配置的节点创建证书。
   ```bash
   # Create a private key for the node certificate.
   sudo openssl genrsa -out node1-key-temp.pem 2048
   
   # Convert the private key to PKCS#8.
   sudo openssl pkcs8 -inform PEM -outform PEM -in node1-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out node1-key.pem
   
   # Create the CSR and replace the arguments passed to -subj so they reflect your specific host.
   # The CN should match a DNS A record for the host-do not use the hostname.
   sudo openssl req -new -key node1-key.pem -subj "/C=CA/ST=ONTARIO/L=TORONTO/O=ORG/OU=UNIT/CN=node1.dns.a-record" -out node1.csr
   
   # Create an extension file that defines a SAN DNS name for the host. This
   # should match the DNS A record of the host.
   sudo sh -c 'echo subjectAltName=DNS:node1.dns.a-record > node1.ext'

   # Sign the node certificate with the root certificate and private key that you created earlier.
   sudo openssl x509 -req -in node1.csr -CA root-ca.pem -CAkey root-ca-key.pem -CAcreateserial -sha256 -out node1.pem -days 730 -extfile node1.ext
   ```
1. 删除不再需要的临时文件。
   ```bash
   sudo rm -f *temp.pem *csr *ext
   ```
   {% include copy.html %}

1. 确保其余证书归 opensearch 用户所有。
   ```bash
   sudo chown opensearch:opensearch admin-key.pem admin.pem node1-key.pem node1.pem root-ca-key.pem root-ca.pem root-ca.srl
   ```
   {% include copy.html %}

1. 将这些证书添加到中，如中 `opensearch.yml` [生成证书]({{site.url}}{{site.baseurl}}/security/configuration/generate-certificates/#add-distinguished-names-to-opensearchyml)所述。高级用户还可以选择使用脚本附加设置：
   ```bash
   #! /bin/bash

   # Before running this script, make sure to replace the CN in the 
   # node's distinguished name with a real DNS A record.

   echo "plugins.security.ssl.transport.pemcert_filepath: /etc/opensearch/node1.pem" | sudo tee -a /etc/opensearch/opensearch.yml
   echo "plugins.security.ssl.transport.pemkey_filepath: /etc/opensearch/node1-key.pem" | sudo tee -a /etc/opensearch/opensearch.yml
   echo "plugins.security.ssl.transport.pemtrustedcas_filepath: /etc/opensearch/root-ca.pem" | sudo tee -a /etc/opensearch/opensearch.yml
   echo "plugins.security.ssl.http.enabled: true" | sudo tee -a /etc/opensearch/opensearch.yml
   echo "plugins.security.ssl.http.pemcert_filepath: /etc/opensearch/node1.pem" | sudo tee -a /etc/opensearch/opensearch.yml
   echo "plugins.security.ssl.http.pemkey_filepath: /etc/opensearch/node1-key.pem" | sudo tee -a /etc/opensearch/opensearch.yml
   echo "plugins.security.ssl.http.pemtrustedcas_filepath: /etc/opensearch/root-ca.pem" | sudo tee -a /etc/opensearch/opensearch.yml
   echo "plugins.security.allow_default_init_securityindex: true" | sudo tee -a /etc/opensearch/opensearch.yml
   echo "plugins.security.authcz.admin_dn:" | sudo tee -a /etc/opensearch/opensearch.yml
   echo "  - 'CN=A,OU=UNIT,O=ORG,L=TORONTO,ST=ONTARIO,C=CA'" | sudo tee -a /etc/opensearch/opensearch.yml
   echo "plugins.security.nodes_dn:" | sudo tee -a /etc/opensearch/opensearch.yml
   echo "  - 'CN=node1.dns.a-record,OU=UNIT,O=ORG,L=TORONTO,ST=ONTARIO,C=CA'" | sudo tee -a /etc/opensearch/opensearch.yml
   echo "plugins.security.audit.type: internal_opensearch" | sudo tee -a /etc/opensearch/opensearch.yml
   echo "plugins.security.enable_snapshot_restore_privilege: true" | sudo tee -a /etc/opensearch/opensearch.yml
   echo "plugins.security.check_snapshot_restore_write_privileges: true" | sudo tee -a /etc/opensearch/opensearch.yml
   echo "plugins.security.restapi.roles_enabled: [\"all_access\", \"security_rest_api_access\"]" | sudo tee -a /etc/opensearch/opensearch.yml
   ```
   {% include copy.html %}

1. （可选）添加对自签名根证书的信任。
   ```bash
   # Copy the root certificate to the correct directory
   sudo cp /etc/opensearch/root-ca.pem /etc/pki/ca-trust/source/anchors/

   # Add trust
   sudo update-ca-trust
   ```

### 配置用户

OpenSearch 以多种方式定义和验证用户。一种不需要其他后端基础结构的方法是在中 `internal_users.yml` 手动配置用户。有关配置用户的详细信息，请参阅[YAML files]({{site.url}}{{site.baseurl}}/security/configuration/yaml/)。以下步骤说明如何删除除用户之外 `admin` 的所有演示用户，以及如何使用脚本替换 `admin` 默认密码。

1. 导航到 Security plugins tools 目录。
   ```bash
   cd /usr/share/opensearch/plugins/opensearch-security/tools
   ```
   {% include copy.html %}

1. 运行 `hash.sh` 以生成新密码。
   - 如果尚未定义 JDK 的路径，则此脚本将失败。
      ```bash
      # Example output if a JDK isn't found...
      $ ./hash.sh
      **************************************************************************
      ** This tool will be deprecated in the next major release of OpenSearch **
      ** https://github.com/opensearch-project/security/issues/1755           **
      **************************************************************************
      which: no java in (/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/user/.local/bin:/home/user/bin)
      WARNING: nor OPENSEARCH_JAVA_HOME nor JAVA_HOME is set, will use 
      ./hash.sh: line 35: java: command not found
      ```
   - 在调用脚本时声明环境变量以避免出现问题：
      ```bash
      OPENSEARCH_JAVA_HOME=/usr/share/opensearch/jdk ./hash.sh
      ```
      {% include copy.html %}

   - 在提示符下输入所需的密码，并记下输出哈希值。
1. 打开 `internal_users.yml`。
   ```bash
   sudo vi /etc/opensearch/opensearch-security/internal_users.yml
   ```
   {% include copy.html %}

1. 删除除之外 `admin` 的所有演示用户，并将哈希替换为上一步中提供的 `hash.sh` 输出。该文件应类似于以下示例：The file should look similar to the following example：
   ```bash
   ---
   # This is the internal user database
   # The hash value is a bcrypt hash and can be generated with plugin/tools/hash.sh

   _meta:
      type: "internalusers"
      config_version: 2

   # Define your internal users here

   admin:
      hash: "$2y$1EXAMPLEQqwS8TUcoEXAMPLEeZ3lEHvkEXAMPLERqjyh1icEXAMPLE."
      reserved: true
      backend_roles:
      - "admin"
      description: "Admin user"
   ```
   {% include copy.html %}

### 应用更改

现在，已安装 TLS 证书，演示用户已被删除或分配了新密码，最后一步是应用配置更改。最后一个配置步骤需要在主机上运行 OpenSearch 时调用 `securityadmin.sh`。

1. OpenSearch 必须处于运行状态才能 `securityadmin.sh` 应用更改。如果你对 `opensearch.yml` 进行了更改，请重新启动 OpenSearch。
   ```bash
   sudo systemctl restart opensearch
   ```

1. 打开与主机的单独终端会话，然后导航到包含 `securityadmin.sh` 的目录。
   ```bash
   # Change to the correct directory
   cd /usr/share/opensearch/plugins/opensearch-security/tools
   ```

1. 调用脚本。有关必须传递的参数的定义，请参阅[使用 securityadmin.sh 应用更改]({{site.url}}{{site.baseurl}}/security/configuration/security-admin/)。
   ```bash
   # You can omit the environment variable if you declared this in your $PATH.
   OPENSEARCH_JAVA_HOME=/usr/share/opensearch/jdk ./securityadmin.sh -cd /etc/opensearch/opensearch-security/ -cacert /etc/opensearch/root-ca.pem -cert /etc/opensearch/admin.pem -key /etc/opensearch/admin-key.pem -icl -nhnv
   ```

### 验证服务是否正在运行

OpenSearch 现在使用自定义 TLS 证书和用于基本身份验证的安全用户在你的主机上运行。你可以通过从另一台主机向 OpenSearch 节点发送 API 请求来验证外部连接。

在上一次测试中，你将请求定向到 `localhost`。现在，TLS 证书已应用，并且新证书引用了主机的实际 DNS 记录，因此请求 `localhost` 将无法通过 CN 检查，并且证书将被视为无效。相反，应将请求发送到你在生成证书时指定的地址。

在发送请求之前，应向客户端添加对根证书的信任。如果不添加信任，则必须使用该 `-k` 选项，以便 cURL 忽略 CN 和根证书验证。{:.tip}

```bash
$ curl https://your.host.address:9200 -u admin:yournewpassword -k
{
  "name" : "hostname-here",
  "cluster_name" : "opensearch",
  "cluster_uuid" : "efC0ANNMQlGQ5TbhNflVPg",
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

## 升级到较新的版本

使用 RPM 或 YUM 安装的 OpenSearch 实例可以轻松升级到较新版本。我们建议使用 YUM 进行更新，但你也可以使用 RPM 进行升级。


### 使用 RPM 手动升级

直接从[OpenSearch 下载页面](https://opensearch.org/downloads.html){:target='\_blank'} 下载所需升级版本的 RPM 包。

导航到包含分发的目录并运行以下命令：
```bash
rpm -Uvh opensearch-{{site.opensearch_version}}-linux-x64.rpm
```
{% include copy.html %}

### 百胜

要使用 YUM 升级到最新版本的 OpenSearch，请执行以下操作：
```bash
sudo yum update
```
{% include copy.html %}

你还可以升级到特定的 OpenSearch 版本：
 ```bash
 sudo yum update opensearch-<version-number>
 ```
 {% include copy.html %}

## 相关链接

- [OpenSearch 配置]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/)
- [安装和配置 OpenSearch 控制面板]({{site.url}}{{site.baseurl}}/install-and-configure/install-dashboards/index/)
- [OpenSearch 插件安装]({{site.url}}{{site.baseurl}}/opensearch/install/plugins/)
- [关于安全插件]({{site.url}}{{site.baseurl}}/security/index/)