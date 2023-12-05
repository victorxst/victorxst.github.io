---
layout: default
title: 对SecurityAdmin.SH进行故障排除
nav_order: 10
---

# SecurityAdmin.SH故障排除

此页面包括用于故障排除的步骤`securityadmin.sh`。该脚本可以在`/plugins/opensearch-security/tools/securityadmin.sh`。有关使用此工具的更多信息，请参阅[将更改应用于配置文件]({{site.url}}{{site.baseurl}}/security/configuration/security-admin/)。


---

#### Table of contents
- TOC
{:toc}


---

## 群集无法到达

如果`securityadmin.sh` 无法到达群集，它输出：

```
OpenSearch Security Admin v6
Will connect to localhost:9300
ERR: Seems there is no opensearch running on localhost:9300 - Will exit
```


### 检查主机名

默认情况下，`securityadmin.sh` 用途`localhost`。如果您的群集在任何其他主机上运行，请使用`-h` 选项。


### 检查端口

检查您正在运行`securityadmin.sh` 在运输港，**不是** HTTP端口。

默认情况下，`securityadmin.sh` 用途`9300`。如果您的群集在其他端口上运行，请使用`-p` 指定端口号的选项。


## 没有配置的节点可用

如果`securityadmin.sh` 可以到达群集，但无法更新配置，它会输出此错误：

```
Contacting opensearch cluster 'opensearch' and wait for YELLOW clusterstate ...
Cannot retrieve cluster state due to: None of the configured nodes are available: [{#transport#-1}{mr2NlX3XQ3WvtVG0Dv5eHw}{localhost}{127.0.0.1:9300}]. This is not an error, will keep on trying ...
```

*尝试运行`securityadmin.sh` 和`-icl` 和`-nhnv`。

  如果有效，请检查您的群集名称以及SSL证书中的主机名。如果这不起作用，请尝试运行`securityadmin.sh` 和`--diagnose` 并查看诊断跟踪日志文件。

* 添加`--accept-red-cluster` 允许`securityadmin.sh` 在红色集群上操作。


### 检查群集名称

默认情况下，`securityadmin.sh` 用途`opensearch` 作为群集名称。

如果您的集群具有不同的名称，则可以使用该名称完全忽略该名称`-icl` 选项或使用`-cn` 选项。


### 检查主机名验证

默认情况下，`securityadmin.sh` 验证节点证书中的主机名与节点的实际主机名匹配。

如果不是这种情况（例如，如果您使用的是演示证书），则可以通过添加hostname验证`-nhnv` 选项。


### 检查群集状态

默认情况下，`securityadmin.sh` 仅当群集状态至少为黄色时执行。

如果您的群集状态为红色，您仍然可以执行`securityadmin.sh`，但是您需要添加`-arc` 选项。


### 检查安全索引名称

默认情况下，安全插件使用`.opendistro_security` 作为配置索引的名称。如果您在`opensearch.yml`，使用`-i` 选项。


## "ERR: DN is not an admin user"

如果TLS证书曾经开始`securityadmin.sh` 不是管理员证书，脚本输出：

```
Connected as CN=node-0.example.com,OU=SSL,O=Test,L=Test,C=DE
ERR: CN=node-0.example.com,OU=SSL,O=Test,L=Test,C=DE is not an admin user
```

执行脚本时，必须使用管理证书。要了解更多，请参阅[配置管理证书]({{site.url}}{{site.baseurl}}/security/configuration/tls/#configuring-admin-certificates)。

## 使用诊断选项

有关原因的更多信息`securityadmin.sh` 不执行，添加`--diagnose` 选项：

```
./securityadmin.sh -diagnose -cd ../../../config/opensearch-security/ -cacert ... -cert ... -key ... -keypass ...
```

脚本打印生成的诊断文件的位置。

