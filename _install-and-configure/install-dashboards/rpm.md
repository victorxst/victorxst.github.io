---
layout: default
title: RPM
parent: 安装 OpenSearch 控制面板
nav_order: 31
redirect_from: 
  - /dashboards/install/rpm/
---

{% comment %} The following liquid syntax declares a variable, major_version_mask, which is transformed into "N.x" where "N" is the major version number. This is required for proper versioning references to the Yum repo. {% endcomment %} {% assign version_parts = site.opensearch_major_minor_version | split: "." %} {% assign major_version_mask = version_parts[0] | append: ".x" %}

# 使用 RPM Package Manager （RPM）运行 OpenSearch 控制面板

OpenSearch 控制面板是 OpenSearch 中数据的默认可视化工具。它还充当许多 OpenSearch 插件的用户界面，包括安全性、警报、索引状态管理、SQL 等。

## 从软件包安装 OpenSearch 控制面板

1. 直接从[OpenSearch 下载页面](https://opensearch.org/downloads.html){:target='\_blank'} 下载所需版本的 RPM 包。RPM 包可以同时下载，也可以**arm64 的**下载**64 倍**。
1. 导入 GPG 公钥。此密钥验证你的 OpenSearch 实例是否已签名。
    ```bash
    sudo rpm --import https://artifacts.opensearch.org/publickeys/opensearch.pgp
    ```
1. 在命令行界面（CLI）中，可以使用 `rpm` 或 `yum` 安装软件包。
    **64 倍**
    ```bash
    # Install the x64 package using yum.
    sudo yum install opensearch-dashboards-{{site.opensearch_version}}-linux-x64.rpm
    # Install the x64 package using rpm.
    sudo rpm -ivh opensearch-dashboards-{{site.opensearch_version}}-linux-x64.rpm
    ```
    **arm64 的**
    ```bash
    # Install the arm64 package using yum.
    sudo yum install opensearch-dashboards-{{site.opensearch_version}}-linux-arm64.rpm
    # Install the arm64 package using rpm.
    sudo rpm -ivh opensearch-dashboards-{{site.opensearch_version}}-linux-arm64.rpm
    ```
1. 安装成功后，启用 OpenSearch 控制面板即服务。
    ```bash
    sudo systemctl enable opensearch-dashboards
    ```
1. 启动 OpenSearch 控制面板。
    ```bash
    sudo systemctl start opensearch-dashboards
    ```
1. 验证 OpenSearch 控制面板是否正确启动。
    ```bash
    sudo systemctl status opensearch-dashboards
    ```

## 从本地 YUM 存储库安装 OpenSearch 控制面板

YUM 是基于 Red Hat 的操作系统的主要软件包管理工具，它允许你从 YUM 存储库库下载和安装 RPM 软件包。

1. 为 OpenSearch 控制面板创建本地存储库文件：
   ```bash
   sudo curl -SL https://artifacts.opensearch.org/releases/bundle/opensearch-dashboards/{{major_version_mask}}/opensearch-dashboards-{{major_version_mask}}.repo -o /etc/yum.repos.d/opensearch-dashboards-{{major_version_mask}}.repo
   ```
1. 验证存储库是否已成功创建。
    ```bash
    sudo yum repolist
    ```
1. 清理 YUM 缓存，以确保顺利安装：
   ```bash
   sudo yum clean all
   ```
1. 下载存储库文件后，列出 OpenSearch-Dashboards 的所有可用版本：
   ```bash
   sudo yum list opensearch-dashboards --showduplicates
   ```
1. 选择要安装的 OpenSearch 控制面板版本：
   - 除非另有说明，否则将安装 OpenSearch 的最高次要版本。
   ```bash
   sudo yum install opensearch-dashboards
   ```
   - 要安装特定版本的 OpenSearch 控制面板，请执行以下操作：
   ```bash
   sudo yum install 'opensearch-dashboards-{{site.opensearch_version}}'
   ```
1. 在安装过程中，安装程序将向你显示 GPG 密钥指纹。验证信息是否与以下内容匹配：
   ```bash
   Fingerprint: c5b7 4989 65ef d1c2 924b a9d5 39d3 1987 9310 d3fc
   ```
    - 如果正确，请输入 `yes` 或 `y`。OpenSearch 安装将继续。
1. 完成后，你可以运行 OpenSearch 控制面板。
    ```bash
    sudo systemctl start opensearch-dashboards
    ```