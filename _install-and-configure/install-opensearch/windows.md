---
layout: default
title: Windows
parent: 安装 OpenSearch
nav_order: 65
---

# 窗户

以下部分介绍如何从 zip 存档在 Windows 上安装 OpenSearch。

一般来说，从 zip 存档安装 OpenSearch 可以分为几个步骤：

1. **下载并解压缩 OpenSearch。**
1. **（可选）测试 OpenSearch。**
   - 在应用任何自定义配置之前，请确认 OpenSearch 能够运行。
   - 这可以在没有任何安全性（无密码、无证书）的情况下完成，也可以使用可由打包脚本应用的演示安全配置来完成。
1. **为你的环境配置 OpenSearch。**
   -  将基本设置应用于 OpenSearch，并开始在你的环境中使用它。

Windows OpenSearch 存档是一个独立的目录，其中包含运行 OpenSearch 所需的一切，包括集成的 Java 开发工具包（JDK）。如果你有自己的 Java 安装并设置了环境变量，则在未设置环境变量 `JAVA_HOME` 的情况下 `OPENSEARCH_JAVA_HOME`，OpenSearch 将使用该安装。要了解如何设置 `OPENSEARCH_JAVA_HOME` 环境变量，请参见[步骤 3：在你的环境中设置 OpenSearch](#step-3-set-up-opensearch-in-your-environment)。

## 先决条件

确保你已安装 zip 实用程序。

## 第 1 步：下载并解压缩 OpenSearch

执行以下步骤以在 Windows 上安装 OpenSearch。

1. 下载 [ `opensearch-{{site.opensearch_version}}-windows-x64.zip`]（https://artifacts.opensearch.org/releases/bundle/opensearch/{{site.opensearch_version}}/opensearch--windows-x64{{site.opensearch_version}}.zip） {:target='\_blank'} 存档。
1. 要提取存档内容，请右键单击以选择**全部提取**。

## 步骤 2：（可选）测试 OpenSearch

在继续进行任何配置之前，你应测试 OpenSearch 的安装。否则，可能很难确定将来的问题是由安装问题还是安装后应用的自定义设置引起的。在此阶段，有两种快速方法可以测试 OpenSearch：

1. **（已启用安全性）**使用 Windows 存档中包含的批处理脚本应用通用配置。
1. **（安全已禁用）**手动禁用安全插件并测试实例，然后再应用你自己的自定义安全设置。

批处理脚本会将通用配置应用于你的 OpenSearch 实例。此配置定义了一些环境变量，并应用自签名 TLS 证书。或者，你可以选择自己配置这些。

如果你只想验证服务是否配置正确，并且打算自己配置安全设置，则可能需要禁用安全插件并在不加密或身份验证的情况下启动服务。

默认配置中的 OpenSearch 节点（具有演示证书和具有默认密码的用户）不适合生产环境。如果你计划在生产环境中使用该节点，那么至少应将演示 TLS 证书替换为你自己的 TLS 证书和[更新内部用户和密码列表]({{site.url}}{{site.baseurl}}/security/configuration/yaml)。有关其他指导，请参阅以确保[安全配置]({{site.url}}{{site.baseurl}}/security/configuration/index/)根据安全要求配置节点。
{: .warning}

### 选项 1：在启用安全性的情况下测试你的 OpenSearch 设置

1. 运行 demo 批处理脚本。

   有两种方法可以运行批处理脚本：

   1. 使用 Windows UI 运行批处理脚本：

      1. 导航到 OpenSearch 安装的顶部目录并打开该 `opensearch-{{site.opensearch_version}}` 文件夹。
      1. 通过双击 `opensearch-windows-install.bat` 文件来运行批处理脚本。这将打开一个命令提示符，其中正在运行 OpenSearch 实例。

   1. 从命令提示符或 Powershell 运行批处理脚本：

      1. 通过输入打开命令提示符，或通过在任务栏旁边的**开始**搜索框中输入 `cmd` `powershell` 来打开 Powershell。
      1. 切换到 OpenSearch 安装的顶层目录。
         ```bat
         cd \path\to\opensearch-{{site.opensearch_version}}
         ```
         {% include copy.html %}

      1. 运行批处理脚本。
         ```bat
         .\opensearch-windows-install.bat
         ```
         {% include copy.html %}

1. 打开新的命令提示符并向服务器发送请求以验证 OpenSearch 是否正在运行。请注意该 `--insecure` 标志的使用，这是必需的，因为 TLS 证书是自签名的。
   - 向端口 9200 发送请求：
      ```bat
      curl.exe -X GET https://localhost:9200 -u "admin:admin" --insecure
      ```
      {% include copy.html %}

      你应该得到如下所示的响应：
      ```bat
      {
         "name" : "hostname-here",
         "cluster_name" : "opensearch",
         "cluster_uuid" : "7Nqtr0LrQTOveFcBb7Kufw",
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
      ```bat
      curl.exe -X GET https://localhost:9200/_cat/plugins?v -u "admin:admin" --insecure
      ```
      {% include copy.html %}

      响应应如下所示：
      ```bat
      hostname opensearch-alerting                  {{site.opensearch_version}}
      hostname opensearch-anomaly-detection         {{site.opensearch_version}}
      hostname opensearch-asynchronous-search       {{site.opensearch_version}}
      hostname opensearch-cross-cluster-replication {{site.opensearch_version}}
      hostname opensearch-geospatial                {{site.opensearch_version}}
      hostname opensearch-index-management          {{site.opensearch_version}}
      hostname opensearch-job-scheduler             {{site.opensearch_version}}
      hostname opensearch-knn                       {{site.opensearch_version}}
      hostname opensearch-ml                        {{site.opensearch_version}}
      hostname opensearch-neural-search             {{site.opensearch_version}}
      hostname opensearch-notifications             {{site.opensearch_version}}
      hostname opensearch-notifications-core        {{site.opensearch_version}}
      hostname opensearch-observability             {{site.opensearch_version}}
      hostname opensearch-reports-scheduler         {{site.opensearch_version}}
      hostname opensearch-security                  {{site.opensearch_version}}
      hostname opensearch-security-analytics        {{site.opensearch_version}}
      hostname opensearch-sql                       {{site.opensearch_version}}
      ```

### 选项 2：在禁用安全性的情况下测试 OpenSearch 设置

1.  `opensearch-{{site.opensearch_version}}\config` 打开文件夹。
1.  `opensearch.yml` 使用文本编辑器打开文件。
1. 添加以下行以禁用安全插件：
   ```yaml
   plugins.security.disabled: true
   ```
   {% include copy.html %}

1. 保存更改并关闭文件。
1. 导航到 OpenSearch 安装的顶部目录并打开该 `opensearch-{{site.opensearch_version}}` 文件夹。
1. 通过双击 `opensearch-windows-install.bat` 文件来运行默认值。这将打开一个命令提示符，其中正在运行 OpenSearch 实例。
1. 打开新的命令提示符并向服务器发送请求以验证 OpenSearch 是否正在运行。由于 Security 插件已被禁用，因此你将使用 `HTTP` `HTTPS` 而不是.
   - 向端口 9200 发送请求：
      ```bat
      curl.exe -X GET http://localhost:9200
      ```
      {% include copy.html %}

      你应该得到如下所示的响应：
      ```bat
      {
         "name" : "hostname-here",
         "cluster_name" : "opensearch",
         "cluster_uuid" : "7Nqtr0LrQTOveFcBb7Kufw",
         "version" : {
            "distribution" : "opensearch",
            "number" : "2.4.0",
            "build_type" : "zip",
            "build_hash" : "77ef9e304dd6ee95a600720a387a9735bbcf7bc9",
            "build_date" : "2022-11-05T05:50:15.404072800Z",
            "build_snapshot" : false,
            "lucene_version" : "9.4.1",
            "minimum_wire_compatibility_version" : "7.10.0",
            "minimum_index_compatibility_version" : "7.0.0"
         },
         "tagline" : "The OpenSearch Project: https://opensearch.org/"
      }
      ```
   - 查询插件端点：
      ```bat
      curl.exe -X GET http://localhost:9200/_cat/plugins?v
      ```
      {% include copy.html %}

      响应应如下所示：
      ```bat
      hostname opensearch-alerting                  {{site.opensearch_version}}
      hostname opensearch-anomaly-detection         {{site.opensearch_version}}
      hostname opensearch-asynchronous-search       {{site.opensearch_version}}
      hostname opensearch-cross-cluster-replication {{site.opensearch_version}}
      hostname opensearch-geospatial                {{site.opensearch_version}}
      hostname opensearch-index-management          {{site.opensearch_version}}
      hostname opensearch-job-scheduler             {{site.opensearch_version}}
      hostname opensearch-knn                       {{site.opensearch_version}}
      hostname opensearch-ml                        {{site.opensearch_version}}
      hostname opensearch-neural-search             {{site.opensearch_version}}
      hostname opensearch-notifications             {{site.opensearch_version}}
      hostname opensearch-notifications-core        {{site.opensearch_version}}
      hostname opensearch-observability             {{site.opensearch_version}}
      hostname opensearch-reports-scheduler         {{site.opensearch_version}}
      hostname opensearch-security                  {{site.opensearch_version}}
      hostname opensearch-security-analytics        {{site.opensearch_version}}
      hostname opensearch-sql                       {{site.opensearch_version}}
      ```

要停止 OpenSearch，请按 `Ctrl+C` 命令提示符或 Powershell，或者直接关闭命令提示符或 Powershell 窗口。
{:.tip}

## 步骤 3：在你的环境中设置 OpenSearch

之前没有 OpenSearch 经验的用户可能需要推荐设置列表才能开始使用该服务。默认情况下，OpenSearch 未绑定到网络接口，外部主机无法访问。此外，如果通过调用<span style="white-space: nowrap"> `opensearch-windows-install.bat` 运行安全演示脚本，则安全设置要么是未定义的（绿地安装），要么是由默认用户名和密码填充的。</span>以下建议将使用户能够将 OpenSearch 绑定到网络接口。

以下建议的设置将允许你：

- 将 OpenSearch 绑定到主机上的 IP 或网络接口。
- 设置初始和最大 JVM 堆大小。
- 定义一个指向捆绑的 JDK 的环境变量。

如果运行了安全演示脚本，则需要手动重新配置已修改的设置。在继续操作之前，[安全配置]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/)请参阅有关指导。
{:.note}

在修改任何配置文件之前，最好先保存备份副本，然后再进行更改。备份文件可用于还原由错误配置引起的任何问题。
{:.tip}

1.  `opensearch-{{site.opensearch_version}}\config` 打开文件夹。
1.  `opensearch.yml` 使用文本编辑器打开文件。
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
   1.   `opensearch-{{site.opensearch_version}}\config` 打开文件夹。
   1.   `jvm.options` 使用文本编辑器打开文件。
   1. 修改初始堆大小和最大堆大小的值。首先，应将这些值设置为可用系统内存的一半。对于专用主机，可以根据你的工作流要求增加此值。<br>
    例如，如果主机具有 8 GB 的内存，则可能需要将初始堆大小和最大堆大小设置为 4 GB：
    ```bash
    -Xms4g
    -Xmx4g
    ```
    {% include copy.html %}

   1. 保存更改并关闭文件。
1. 指定包含的 JDK 的位置。
    1. 在任务栏上旁边的**开始**搜索框中，输入 `edit environment variables for your account` 或 `edit the system environment variables`。要编辑系统环境变量，你需要管理员权限。用户环境变量优先于系统环境变量。
    1. 选择**编辑帐户的环境变量**或**编辑系统环境变量**。
    1. **系统属性**如果对话框打开，请在选项卡中**高深**选择**环境变量**。
    1. 在或**用户变量** **系统变量**下，选择**新增功能**。
    1. 在中**变量名称**，输入 `OPENSEARCH_JAVA_HOME`。
    1. 在中**变量值**，输入 `\path\to\opensearch-{{site.opensearch_version}}\jdk`。
    1. 选择此选项**还行**可关闭所有对话框。

## 插件兼容性

性能分析器插件在 Windows 上不可用。所有其他 OpenSearch 插件（包括 k-NN 插件）都可用。有关插件的完整列表，请参见[可用的插件]({{site.url}}{{site.baseurl}}/opensearch/install/plugins/#available-plugins)。

## 相关链接

- [OpenSearch 配置]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/)
- [OpenSearch 插件安装]({{site.url}}{{site.baseurl}}/opensearch/install/plugins/)
- [关于安全插件]({{site.url}}{{site.baseurl}}/security/index/)
