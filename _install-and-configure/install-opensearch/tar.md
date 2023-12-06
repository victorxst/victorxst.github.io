---
layout: default
title: Tarball
parent: 安装 OpenSearch
nav_order: 10
redirect_from:
  - /opensearch/install/tar/
---

# Tarball

从 tarball（也称为 tar 存档）安装 OpenSearch 可能会吸引希望对文件权限和安装路径等安装详细信息进行精细控制的用户。

一般来说，从 tarball 安装 OpenSearch 可以分为几个步骤：

1. **下载并解压缩 OpenSearch。**
1. **配置重要的系统设置。**
   - 这些设置将在修改任何 OpenSearch 文件之前应用于主机。
1. **（可选）测试 OpenSearch。**
   - 在应用任何自定义配置之前，请确认 OpenSearch 能够运行。
   - 这可以在没有任何安全性（无密码、无证书）的情况下完成，也可以使用可由打包脚本应用的演示安全配置来完成。
1. **为你的环境配置 OpenSearch。**
   -  将基本设置应用于 OpenSearch，并开始在你的环境中使用它。

tarball 是一个独立的目录，其中包含运行 OpenSearch 所需的一切，包括集成的 Java 开发工具包（JDK）。此安装方法与大多数 Linux 发行版兼容，包括 CentOS 7、Amazon Linux 2 和 Ubuntu 18.04. 如果你有自己的 Java 安装并在终端中设置了环境变量 `JAVA_HOME`，则 macOS 也可以工作。

本指南假定你能够熟练地使用 Linux 命令行界面（CLI）工作。你应该了解如何输入命令、在目录之间导航和编辑文本文件。一些示例命令引用 `vi` 文本编辑器，但你可以使用任何可用的文本编辑器。
{: .note}

## 第 1 步：下载并解压缩 OpenSearch

1. 从[OpenSearch 下载页面](https://opensearch.org/downloads.html){:target='\_blank'} 或使用命令行（例如使用 `wget`）下载相应的 tar.gz 归档文件。
   ```bash
   # x64
   wget https://artifacts.opensearch.org/releases/bundle/opensearch/{{site.opensearch_version}}/opensearch-{{site.opensearch_version}}-linux-x64.tar.gz

   # ARM64
   wget https://artifacts.opensearch.org/releases/bundle/opensearch/{{site.opensearch_version}}/opensearch-{{site.opensearch_version}}-linux-arm64.tar.gz
   ```
1. 提取 tarball 的内容。
   ```bash
   # x64
   tar -xvf opensearch-{{site.opensearch_version}}-linux-x64.tar.gz
   
   # ARM64
   tar -xvf opensearch-{{site.opensearch_version}}-linux-arm64.tar.gz
   ```

## 第 2 步：配置重要的系统设置

在启动 OpenSearch 之前，你应该查看一些[重要的系统设置]({{site.url}}{{site.baseurl}}/opensearch/install/important-settings/){:target='\_blank'}。
1. 禁用主机上的内存分页和交换性能以提高性能。
   ```bash
   sudo swapoff -a
   ```
   {% include copy.html %}

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

## 步骤 3：（可选）测试 OpenSearch

在继续操作之前，你应该测试 OpenSearch 的安装。否则，可能很难确定将来的问题是由安装问题还是安装后应用的自定义设置引起的。在此阶段，有两种快速方法可以测试 OpenSearch：

1. **（已启用安全性）**使用 tar 存档中包含的演示安全脚本应用通用配置。
1. **（安全已禁用）**手动禁用安全插件并测试实例，然后再应用你自己的自定义安全设置。

演示安全脚本会将通用配置应用于你的 OpenSearch 实例。此配置定义了一些环境变量，并应用自签名 TLS 证书。如果要自行配置这些配置，请参见[步骤 4：在你的环境中设置 OpenSearch](#step-4-set-up-opensearch-in-your-environment)。

如果你只想验证服务是否配置正确，并且打算自己配置安全设置，则可能需要禁用安全插件并在不加密或身份验证的情况下启动服务。

演示安全脚本配置的 OpenSearch 节点不适用于生产环境。如果你计划在运行 `opensearch-tar-install.sh` 后在生产环境中使用该节点，则至少应将演示 TLS 证书替换为你自己的 TLS 证书和[更新内部用户和密码列表]({{site.url}}{{site.baseurl}}/security/configuration/yaml)。有关其他指导，请参阅以确保[安全配置]({{site.url}}{{site.baseurl}}/security/configuration/index/)根据安全要求配置节点。
{: .warning}

### 选项 1：在启用安全性的情况下测试你的 OpenSearch 设置

1. 切换到 OpenSearch 安装的顶层目录。
   ```bash
   cd /path/to/opensearch-{{site.opensearch_version}}
   ```
   {% include copy.html %}

1. 使用安全演示配置运行 OpenSearch 启动脚本。
   ```bash
   ./opensearch-tar-install.sh
   ```
   {% include copy.html %}

1. 打开另一个终端会话并向服务器发送请求以验证 OpenSearch 是否正在运行。请注意该 `--insecure` 标志的使用，这是必需的，因为 TLS 证书是自签名的。
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
1. 返回到原始终端会话，然后按停止 `CTRL + C` 进程。

### 选项 2：在禁用安全性的情况下测试 OpenSearch 设置

1. 打开配置文件。
   ```bash
   vi /path/to/opensearch-{{site.opensearch_version}}/config/opensearch.yml
   ```
   {% include copy.html %}

1. 添加以下行以禁用安全插件：
   ```bash
   plugins.security.disabled: true
   ```
   {% include copy.html %}

1. 保存更改并关闭文件。
1. 打开另一个终端会话并向服务器发送请求以验证 OpenSearch 是否正在运行。由于 Security 插件已被禁用，因此你将使用 `HTTP` `HTTPS` 而不是.
   - 向端口 9200 发送请求。
      ```bash
      curl -X GET http://localhost:9200
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
   - 查询插件端点。
      ```bash
      curl -X GET http://localhost:9200/_cat/plugins?v
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

## 步骤 4：在你的环境中设置 OpenSearch

之前没有 OpenSearch 经验的用户可能需要推荐设置列表才能开始使用该服务。默认情况下，OpenSearch 未绑定到网络接口，外部主机无法访问。此外，如果通过调用 `opensearch-tar-install.sh` 运行安全演示脚本，则安全设置要么是未定义的（绿地安装），要么是由默认用户名和密码填充的。以下建议将使用户能够将 OpenSearch 绑定到网络接口、创建和签署 TLS 证书以及配置基本身份验证。

以下建议的设置将允许你：

- 将 OpenSearch 绑定到主机上的 IP 或网络接口。
- 设置初始和最大 JVM 堆大小。
- 定义一个指向捆绑的 JDK 的环境变量。
- 配置你自己的 TLS 证书 - 不需要第三方证书颁发机构（CA）。
- 使用自定义密码创建管理员用户。

如果运行了安全演示脚本，则需要手动重新配置已修改的设置。在继续操作之前，[安全配置]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/)请参阅有关指导。
{: .note}

在修改任何配置文件之前，最好先保存备份副本，然后再进行更改。备份文件可用于还原由错误配置引起的任何问题。
{:.tip}

1. 打开 `opensearch.yml`。
   ```bash
   vi /path/to/opensearch-{{site.opensearch_version}}/config/opensearch.yml
   ```
   {% include copy.html %}

1. 添加以下行。
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
         vi /path/to/opensearch-{{site.opensearch_version}}/config/jvm.options
         ```
         {% include copy.html %}

   1. 修改初始堆大小和最大堆大小的值。首先，应将这些值设置为可用系统内存的一半。对于专用主机，可以根据你的工作流要求增加此值。
      -  例如，如果主机具有 8 GB 内存，则可能需要将初始堆大小和最大堆大小设置为 4 GB：
         ```bash
         -Xms4g
         -Xmx4g
         ```
         {% include copy.html %}

   1. 保存更改并关闭文件。
1. 指定包含的 JDK 的位置。
   ```bash
   export OPENSEARCH_JAVA_HOME=/path/to/opensearch-{{site.opensearch_version}}/jdk
   ```
   {% include copy.html %}

### 配置 TLS

TLS 证书允许客户端确认主机的身份并加密客户端与主机之间的流量，从而为群集提供额外的安全性。有关更多信息，请参阅[配置 TLS 证书]({{site.url}}{{site.baseurl}}/security/configuration/tls/)文档中包含的[安全插件]({{site.url}}{{site.baseurl}}/security/index/)和[生成证书]({{site.url}}{{site.baseurl}}/security/configuration/generate-certificates/)。对于在开发环境中执行的工作，自签名证书通常就足够了。本部分将指导你完成生成自己的 TLS 证书并将其应用于 OpenSearch 主机所需的基本步骤。

1. 导航到 OpenSearch `config` 目录。这是证书的存储位置。
   ```bash
   cd /path/to/opensearch-{{site.opensearch_version}}/config/
   ```
   {% include copy.html %}

1. 生成根证书。这是你将用于对其他证书进行签名的内容。
   ```bash
   # Create a private key for the root certificate
   openssl genrsa -out root-ca-key.pem 2048
   
   # Use the private key to create a self-signed root certificate. Be sure to
   # replace the arguments passed to -subj so they reflect your specific host.
   openssl req -new -x509 -sha256 -key root-ca-key.pem -subj "/C=CA/ST=ONTARIO/L=TORONTO/O=ORG/OU=UNIT/CN=ROOT" -out root-ca.pem -days 730
   ```
1. 接下来，创建管理员证书。此证书用于获得执行与安全插件相关的管理任务的提升权限。
   ```bash
   # Create a private key for the admin certificate.
   openssl genrsa -out admin-key-temp.pem 2048

   # Convert the private key to PKCS#8.
   openssl pkcs8 -inform PEM -outform PEM -in admin-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out admin-key.pem
   
   # Create the CSR. A common name (CN) of "A" is acceptable because this certificate is
   # used for authenticating elevated access and is not tied to a host.
   openssl req -new -key admin-key.pem -subj "/C=CA/ST=ONTARIO/L=TORONTO/O=ORG/OU=UNIT/CN=A" -out admin.csr
   
   # Sign the admin certificate with the root certificate and private key you created earlier.
   openssl x509 -req -in admin.csr -CA root-ca.pem -CAkey root-ca-key.pem -CAcreateserial -sha256 -out admin.pem -days 730
   ```
1. 为正在配置的节点创建证书。
   ```bash
   # Create a private key for the node certificate.
   openssl genrsa -out node1-key-temp.pem 2048
   
   # Convert the private key to PKCS#8.
   openssl pkcs8 -inform PEM -outform PEM -in node1-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out node1-key.pem
   
   # Create the CSR and replace the arguments passed to -subj so they reflect your specific host.
   # The CN should match a DNS A record for the host--do not use the hostname.
   openssl req -new -key node1-key.pem -subj "/C=CA/ST=ONTARIO/L=TORONTO/O=ORG/OU=UNIT/CN=node1.dns.a-record" -out node1.csr
   
   # Create an extension file that defines a SAN DNS name for the host. This
   # should match the DNS A record of the host.
   echo 'subjectAltName=DNS:node1.dns.a-record' > node1.ext
   
   # Sign the node certificate with the root certificate and private key that you created earlier.
   openssl x509 -req -in node1.csr -CA root-ca.pem -CAkey root-ca-key.pem -CAcreateserial -sha256 -out node1.pem -days 730 -extfile node1.ext
   ```
1. 删除不再需要的临时文件。
   ```bash
   rm *temp.pem *csr *ext
   ```
   {% include copy.html %}

1. 将这些证书添加到中，如中 `opensearch.yml` [生成证书]({{site.url}}{{site.baseurl}}/security/configuration/generate-certificates/#add-distinguished-names-to-opensearchyml)所述。高级用户还可以选择使用脚本附加设置：
   ```bash
   #! /bin/bash

   # Before running this script, make sure to replace the /path/to your OpenSearch directory,
   # and remember to replace the CN in the node's distinguished name with a real
   # DNS A record.

   echo "plugins.security.ssl.transport.pemcert_filepath: /path/to/opensearch-{{site.opensearch_version}}/config/node1.pem" | sudo tee -a /path/to/opensearch-{{site.opensearch_version}}/config/opensearch.yml
   echo "plugins.security.ssl.transport.pemkey_filepath: /path/to/opensearch-{{site.opensearch_version}}/config/node1-key.pem" | sudo tee -a /path/to/opensearch-{{site.opensearch_version}}/config/opensearch.yml
   echo "plugins.security.ssl.transport.pemtrustedcas_filepath: /path/to/opensearch-{{site.opensearch_version}}/config/root-ca.pem" | sudo tee -a /path/to/opensearch-{{site.opensearch_version}}/config/opensearch.yml
   echo "plugins.security.ssl.http.enabled: true" | sudo tee -a /path/to/opensearch-{{site.opensearch_version}}/config/opensearch.yml
   echo "plugins.security.ssl.http.pemcert_filepath: /path/to/opensearch-{{site.opensearch_version}}/config/node1.pem" | sudo tee -a /path/to/opensearch-{{site.opensearch_version}}/config/opensearch.yml
   echo "plugins.security.ssl.http.pemkey_filepath: /path/to/opensearch-{{site.opensearch_version}}/config/node1-key.pem" | sudo tee -a /path/to/opensearch-{{site.opensearch_version}}/config/opensearch.yml
   echo "plugins.security.ssl.http.pemtrustedcas_filepath: /path/to/opensearch-{{site.opensearch_version}}/config/root-ca.pem" | sudo tee -a /path/to/opensearch-{{site.opensearch_version}}/config/opensearch.yml
   echo "plugins.security.allow_default_init_securityindex: true" | sudo tee -a /path/to/opensearch-{{site.opensearch_version}}/config/opensearch.yml
   echo "plugins.security.authcz.admin_dn:" | sudo tee -a /path/to/opensearch-{{site.opensearch_version}}/config/opensearch.yml
   echo "  - 'CN=A,OU=UNIT,O=ORG,L=TORONTO,ST=ONTARIO,C=CA'" | sudo tee -a /path/to/opensearch-{{site.opensearch_version}}/config/opensearch.yml
   echo "plugins.security.nodes_dn:" | sudo tee -a /path/to/opensearch-{{site.opensearch_version}}/config/opensearch.yml
   echo "  - 'CN=node1.dns.a-record,OU=UNIT,O=ORG,L=TORONTO,ST=ONTARIO,C=CA'" | sudo tee -a /path/to/opensearch-{{site.opensearch_version}}/config/opensearch.yml
   echo "plugins.security.audit.type: internal_opensearch" | sudo tee -a /path/to/opensearch-{{site.opensearch_version}}/config/opensearch.yml
   echo "plugins.security.enable_snapshot_restore_privilege: true" | sudo tee -a /path/to/opensearch-{{site.opensearch_version}}/config/opensearch.yml
   echo "plugins.security.check_snapshot_restore_write_privileges: true" | sudo tee -a /path/to/opensearch-{{site.opensearch_version}}/config/opensearch.yml
   echo "plugins.security.restapi.roles_enabled: [\"all_access\", \"security_rest_api_access\"]" | sudo tee -a /path/to/opensearch-{{site.opensearch_version}}/config/opensearch.yml
   ```
   {% include copy.html %}

1. （可选）添加对自签名根证书的信任。
   ```bash
   # Copy the root certificate to the correct directory
   sudo cp /path/to/opensearch-{{site.opensearch_version}}/config/root-ca.pem /etc/pki/ca-trust/source/anchors/

   # Add trust
   sudo update-ca-trust
   ```

### 配置用户

OpenSearch 以多种方式定义和验证用户。一种不需要额外后端基础结构的方法是在中 `internal_users.yml` 手动配置用户。有关配置用户的详细信息，请参阅[YAML files]({{site.url}}{{site.baseurl}}/security/configuration/yaml/)。以下步骤说明如何删除除用户之外 `admin` 的所有演示用户，以及如何使用脚本替换 `admin` 默认密码。

1. 使安全插件脚本可执行。
   ```bash
   chmod 755 /path/to/opensearch-{{site.opensearch_version}}/plugins/opensearch-security/tools/*.sh
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
      {% include copy.html %}

   - 在调用脚本时声明环境变量以避免出现问题：
      ```bash
      OPENSEARCH_JAVA_HOME=/path/to/opensearch-{{site.opensearch_version}}/jdk ./hash.sh
      ```
      {% include copy.html %}

   - 在提示符下输入所需的密码，并记下输出哈希值。
1. 打开 `internal_users.yml`。
   ```bash
   vi /path/to/opensearch-{{site.opensearch_version}}/config/opensearch-security/internal_users.yml
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

1. 启动 OpenSearch。它必须处于运行状态 `securityadmin.sh` 才能应用更改。
   ```bash
   # Change directories
   cd /path/to/opensearch-{{site.opensearch_version}}/bin

   # Run the service in the foreground
   ./opensearch
   ```
1. 打开与主机的单独终端会话，然后导航到包含 `securityadmin.sh` 的目录。
   ```bash
   # Change to the correct directory
   cd /path/to/opensearch-{{site.opensearch_version}}/plugins/opensearch-security/tools
   ```
1. 调用脚本。有关必须传递的参数的定义，请参阅[使用 securityadmin.sh 应用更改]({{site.url}}{{site.baseurl}}/security/configuration/security-admin/)。
   ```bash
   # You can omit the environment variable if you declared this in your $PATH.
   OPENSEARCH_JAVA_HOME=/path/to/opensearch-{{site.opensearch_version}}/jdk ./securityadmin.sh -cd /path/to/opensearch-{{site.opensearch_version}}/config/opensearch-security/ -cacert /path/to/opensearch-{{site.opensearch_version}}/config/root-ca.pem -cert /path/to/opensearch-{{site.opensearch_version}}/config/admin.pem -key /path/to/opensearch-{{site.opensearch_version}}/config/admin-key.pem -icl -nhnv
   ```
1. 停止并重新启动正在运行的 OpenSearch 进程以应用更改。

### 验证服务是否正在运行

OpenSearch 现在使用自定义 TLS 证书和用于基本身份验证的安全用户在你的主机上运行。你可以通过从另一台主机向 OpenSearch 节点发送 API 请求来验证外部连接。

在之前的测试中，你将请求定向到 `localhost`。现在，TLS 证书已应用，并且新证书引用了主机的实际 DNS 记录，因此请求 `localhost` 将无法通过 CN 检查，并且证书将被视为无效。相反，应将请求发送到你在生成证书时指定的地址。

在发送请求之前，应向客户端添加对根证书的信任。如果不添加信任，则必须使用该 `-k` 选项，以便 cURL 忽略 CN 和根证书验证。
{:.tip}

```bash
$ curl https://your.host.address:9200 -u admin:yournewpassword -k
{
  "name" : "hostname-here",
  "cluster_name" : "opensearch",
  "cluster_uuid" : "efC0ANNMQlGQ5TbhNflVPg",
  "version" : {
    "distribution" : "opensearch",
    "number" : "2.1.0",
    "build_type" : "tar",
    "build_hash" : "388c80ad94529b1d9aad0a735c4740dce2932a32",
    "build_date" : "2022-06-30T21:31:04.823801692Z",
    "build_snapshot" : false,
    "lucene_version" : "9.2.0",
    "minimum_wire_compatibility_version" : "7.10.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "The OpenSearch Project: https://opensearch.org/"
}
```

### 使用 systemd 将 OpenSearch 作为服务运行

本部分将指导你为 OpenSearch 创建服务并将其注册到 `systemd`。定义服务后，你可以使用命令启用、启动和停止 OpenSearch 服务 `systemctl`。本节中的命令反映了 OpenSearch 已安装到 `/opt/opensearch` 的环境，应根据你的安装路径进行更改。

以下配置仅适用于在非生产环境中进行测试。建议不要在生产环境中使用以下配置。如果要在主机上将 OpenSearch 作为 systemd 托管服务运行，则应将 OpenSearch 与分配一起[RPM]({{site.url}}{{site.baseurl}}/install-and-configure/install-opensearch/rpm)安装。tarball 安装不定义特定的安装路径、用户、角色或权限。如果未能正确保护主机环境，可能会导致意外行为。
{: .warning}

1. 为 OpenSearch 服务创建用户。
   ```bash
   sudo adduser --system --shell /bin/bash -U --no-create-home opensearch
   ```
   {% include copy.html %}

1. 将你的用户添加到 `opensearch` 用户组。
   ```bash
   sudo usermod -aG opensearch $USER
   ```
   {% include copy.html %}

1. 将文件所有者 `opensearch` 更改为。如果你的 OpenSearch 文件位于其他目录中，请确保更改路径。
   ```bash
   sudo chown -R opensearch /opt/opensearch/
   ```
   {% include copy.html %}

1. 创建服务文件并打开它进行编辑。
   ```bash
   sudo vi /etc/systemd/system/opensearch.service
   ```
   {% include copy.html %}

1. 输入以下示例服务配置。如果你的 OpenSearch 文件位于其他目录中，请确保更改对路径的引用。
   ```bash
   [Unit]
   Description=OpenSearch
   Wants=network-online.target
   After=network-online.target

   [Service]
   Type=forking
   RuntimeDirectory=data

   WorkingDirectory=/opt/opensearch
   ExecStart=/opt/opensearch/bin/opensearch -d

   User=opensearch
   Group=opensearch
   StandardOutput=journal
   StandardError=inherit
   LimitNOFILE=65535
   LimitNPROC=4096
   LimitAS=infinity
   LimitFSIZE=infinity
   TimeoutStopSec=0
   KillSignal=SIGTERM
   KillMode=process
   SendSIGKILL=no
   SuccessExitStatus=143
   TimeoutStartSec=75

   [Install]
   WantedBy=multi-user.target
   ```
   {% include copy.html %}

1. 重新加载 `systemd` 管理器配置。
   ```bash
   sudo systemctl daemon-reload
   ```
   {% include copy.html %}

1. 启用 OpenSearch 服务。
   ```bash
   sudo systemctl enable opensearch.service
   ```
   {% include copy.html %}

1. 启动 OpenSearch 服务。
   ```bash
   sudo systemctl start opensearch
   ```
   {% include copy.html %}

1. 验证服务是否正在运行。
   ```bash
   sudo systemctl status opensearch
   ```
   {% include copy.html %}

## 相关链接

- [OpenSearch 配置]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/)
- [配置 Performance Analyzer 以进行 Tarball 安装]({{site.url}}{{site.baseurl}}/monitoring-plugins/pa/index/#install-performance-analyzer)
- [安装和配置 OpenSearch 控制面板]({{site.url}}{{site.baseurl}}/install-and-configure/install-dashboards/index/)
- [OpenSearch 插件安装]({{site.url}}{{site.baseurl}}/opensearch/install/plugins/)
- [关于安全插件]({{site.url}}{{site.baseurl}}/security/index/)