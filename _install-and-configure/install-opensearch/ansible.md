---
layout: default
title: Ansible playbook
parent: 安装 OpenSearch
nav_order: 60
redirect_from:
  - /opensearch/install/ansible/
---

# Ansible playbook

你可以使用 Ansible playbook 安装和配置生产就绪型 OpenSearch 集群以及 OpenSearch 控制面板。

Ansible playbook 仅支持将 OpenSearch 和 OpenSearch 控制面板部署到 CentOS7 主机。
{:.note}

## 先决条件

确保你已[Ansible](https://www.ansible.com/)安装。[Java 8](https://www.java.com/en/download/manual.jsp)

## 配置

1. 克隆 OpenSearch [ansible-playbook （英语：ansible-playbook）](https://github.com/opensearch-project/ansible-playbook)存储库：

   ```bash
   git clone https://github.com/opensearch-project/ansible-playbook
   ```
   {% include copy.html %}

2. 在 `inventories/opensearch/hosts` 文件中配置节点属性：

   ```bash
   ansible_host=<Public IP address> ansible_user=root ip=<Private IP address / 0.0.0.0>
   ```
   {% include copy.html %}

   哪里：

   -  `ansible_host` 是你希望 Ansible playbook 在其上安装 OpenSearch 和 OpenSearch DashBoard 的目标节点的 IP 地址。
   -  `ip` 是你希望 OpenSearch 和 OpenSearch DashBoard 绑定到的 IP 地址。你可以指定目标节点的私有 IP，或者 localhost，或者 0.0.0.0。

3. 你可以修改文件中的默认配置值 `inventories/opensearch/group_vars/all/all.yml`。例如，你可以增加 Java 内存堆大小：

   ```bash
   xms_value: 8
   xmx_value: 8
   ```
   {% include copy.html %}

确保你具有对目标节点的 root 用户的直接 SSH 访问权限。
{:.note}

## 使用 Ansible playbook 运行 OpenSearch 和 OpenSearch 控制面板

1. 使用 root 权限运行 Ansible playbook：

   ```bash
   ansible-playbook -i inventories/opensearch/hosts opensearch.yml --extra-vars "admin_password=Test@123 kibanaserver_password=Test@6789"
   ```
   {% include copy.html %}

   你可以使用和 变量为保留用户（ `admin` 和 `kibanaserver` `kibanaserver_password`） `admin_password` 设置密码。

2. 部署过程完成后，你可以使用为 `admin_password` 变量设置的用户名 `admin` 和密码访问 OpenSearch 和 OpenSearch 控制面板。

   如果你绑定 `ip` 到私有 IP 或本地主机，请确保你已登录到部署 playbook 以访问 OpenSearch 和 OpenSearch 控制面板的服务器：

   ```bash
   curl https://localhost:9200 -u 'admin:Test@123' --insecure
   ```
   {% include copy.html %}

   如果绑定 `ip` 到 0.0.0.0，则替换为 `localhost` 公共 IP 或专用 IP（如果位于同一网络中）。
