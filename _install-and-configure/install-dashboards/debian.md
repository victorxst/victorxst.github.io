---
layout: default
title: Debian
parent: 安装 OpenSearch 控制面板
nav_order: 33
---

# 安装 OpenSearch 控制面板（Debian）

与该方法相比[Tarball]({{site.url}}{{site.baseurl}}/install-and-configure/install-dashboards/tar/)，使用 Advanced Packaging Tool（APT）软件包管理器安装 OpenSearch 控制面板可大大简化该过程。例如，包管理器会处理多个技术注意事项，例如安装路径、配置文件的位置以及创建由 `systemd` 管理的服务。

在安装 OpenSearch 控制面板之前，你必须配置 OpenSearch 集群。有关步骤，请参阅 OpenSearch [Debian]({{site.url}}{{site.baseurl}}/install-and-configure/install-opensearch/debian/)安装指南。{：.important}

本指南假定你能够熟练地使用 Linux 命令行界面（CLI）工作。你应该了解如何输入命令、在目录之间导航和编辑文本文件。一些示例命令引用 `vi` 文本编辑器，但你可以使用任何可用的文本编辑器。{:.note}

## 从软件包安装 OpenSearch 控制面板

1. 直接从[OpenSearch 下载页面](https://opensearch.org/downloads.html){:target='\_blank'} 下载所需版本的 Debian 软件包。Debian 软件包可以下载，适用于这两种**64 倍****arm64 的**架构。
1. 在 CLI 中，使用 `dpkg`.
   ```bash
   # x64
   sudo dpkg -i opensearch-dashboards-{{site.opensearch_version}}-linux-x64.deb
   # arm64
   sudo dpkg -i opensearch-dashboards-{{site.opensearch_version}}-linux-arm64.deb
   ```
1. 安装完成后，重新加载 systemd 管理器配置。
    ```bash
    sudo systemctl daemon-reload
    ```
1. 启用 OpenSearch 即服务。
    ```bash
    sudo systemctl enable opensearch-dashboards
    ```
1. 启动 OpenSearch 服务。
    ```bash
    sudo systemctl start opensearch-dashboards
    ```
1. 验证 OpenSearch 是否正确启动。
    ```bash
    sudo systemctl status opensearch-dashboards
    ```

### 指纹验证

Debian 软件包未签名。如果你想验证指纹，OpenSearch 项目会提供用于 `.sig` GNU Privacy Guard（GPG）的文件和 `.deb` 软件包。

1. 下载所需的 Debian 软件包。
   ```bash
   curl -SLO https://artifacts.opensearch.org/releases/bundle/opensearch-dashboards/{{site.opensearch_version}}/opensearch-dashboards-{{site.opensearch_version}}-linux-x64.deb
   ```
1. 下载相应的签名文件。
   ```bash
   curl -SLO https://artifacts.opensearch.org/releases/bundle/opensearch-dashboards/{{site.opensearch_version}}/opensearch-dashboards-{{site.opensearch_version}}-linux-x64.deb.sig
   ```
1. 下载并导入 GPG 密钥。
   ```bash
   curl -o- https://artifacts.opensearch.org/publickeys/opensearch.pgp | gpg --import -
   ```
1. 验证签名。
   ```bash
   gpg --verify opensearch-dashboards-{{site.opensearch_version}}-linux-x64.deb.sig opensearch-dashboards-{{site.opensearch_version}}-linux-x64.deb
   ```

## 从 APT 存储库安装 OpenSearch 控制面板

APT 是基于 Debian 的操作系统的主要软件包管理工具，允许你从 APT 存储库下载和安装 Debian 软件包。

1. 安装必要的软件包。
   ```bash
   sudo apt-get update && sudo apt-get -y install lsb-release ca-certificates curl gnupg2
   ```
1. 导入 GPG 公钥。此密钥用于验证 APT 存储库是否已签名。
    ```bash
    curl -o- https://artifacts.opensearch.org/publickeys/opensearch.pgp | sudo gpg --dearmor --batch --yes -o /usr/share/keyrings/opensearch-keyring
    ```
1. 为 OpenSearch 创建 APT 存储库。
   ```bash
   echo "deb [signed-by=/usr/share/keyrings/opensearch-keyring] https://artifacts.opensearch.org/releases/bundle/opensearch-dashboards/2.x/apt stable main" | sudo tee /etc/apt/sources.list.d/opensearch-dashboards-2.x.list
   ```
1. 验证存储库是否已成功创建。
    ```bash
    sudo apt-get update
    ```
1. 添加存储库信息后，列出 OpenSearch 的所有可用版本：
   ```bash
   sudo apt list -a opensearch-dashboards
   ```
1. 选择要安装的 OpenSearch 版本：
   - 除非另有说明，否则将安装最新版本的 OpenSearch。
   ```bash
   sudo apt-get install opensearch-dashboards
   ```
   - 要安装特定版本的 OpenSearch 控制面板，请在软件包名称后传递版本号。
   ```bash
   # Specify the version manually using opensearch=<version>
   sudo apt-get install opensearch-dashboards={{site.opensearch_version}}
   ```
1. 完成后，启用 OpenSearch。
    ```bash
    sudo systemctl enable opensearch-dashboards
    ```
1. 启动 OpenSearch。
    ```bash
    sudo systemctl start opensearch-dashboards
    ```
1. 验证 OpenSearch 是否正确启动。
    ```bash
    sudo systemctl status opensearch-dashboards
    ```

## 探索 OpenSearch 控制面板

默认情况下，OpenSearch 控制面板（如 OpenSearch）会绑定到 `localhost` 你最初安装它的时间。因此，除非更新配置，否则无法从远程主机访问 OpenSearch 控制面板。

1. 打开 `opensearch_dashboards.yml`。
    ```bash
    sudo vi /etc/opensearch-dashboards/opensearch_dashboards.yml
    ```
1. 指定 OpenSearch 控制面板应绑定到的网络接口。
    ```bash
    # Use 0.0.0.0 to bind to any available interface.
    server.host: 0.0.0.0
    ```
1. 保存并退出。
1. 重新启动 OpenSearch 控制面板以应用配置更改。
    ```bash
    sudo systemctl restart opensearch-dashboards
    ```
1. 在 Web 浏览器中，导航到 OpenSearch 控制面板。默认端口为 5601。
1. 使用默认用户名 `admin` 和默认密码 `admin` 登录。
1. 访问[开始使用 OpenSearch 控制面板]({{site.url}}{{site.baseurl}}/dashboards/index/)以了解更多信息。
