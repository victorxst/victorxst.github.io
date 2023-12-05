---
layout: default
title: 配置
nav_order: 2
has_children: true
has_toc: false
redirect_from:
  - /security-plugin/configuration/
  - /security-plugin/configuration/index/
---

# 安全配置

该插件包含演示证书，因此您可以快速启动并运行。要在生产环境中使用OpenSearch，必须手动配置它：

1. [更换演示证书]({{site.url}}{{site.baseurl}}/install-and-configure/install-opensearch/docker/#configuring-basic-security-settings)。
1. [重新配置`opensearch.yml` 使用您的证书]({{site.url}}{{site.baseurl}}/security/configuration/tls)。
1. [重新配置`config.yml` 使用您的身份验证后端]({{site.url}}{{site.baseurl}}/security/configuration/configuration/) （如果您不打算使用内部用户数据库）。
1. [修改配置yaml文件]({{site.url}}{{site.baseurl}}/security/configuration/yaml)。
1. 如果您打算使用内部用户数据库，[设置密码策略`opensearch.yml`]({{site.url}}{{site.baseurl}}/security/configuration/yaml/#opensearchyml)。
1. [使用更改使用`securityadmin` 脚本]({{site.url}}{{site.baseurl}}/security/configuration/security-admin)。
1. 启动OpenSearch。
1. [添加用户，角色，角色映射和租户]({{site.url}}{{site.baseurl}}/security/access-control/index/)。

如果您不想使用插件，请参阅[禁用安全性]({{site.url}}{{site.baseurl}}/security/configuration/disable)。

安全插件具有几个默认用户，角色，操作组，权限和设置，用于使用Kibana的OpenSearch仪表板。我们将在以后的版本中更改这些名称。
{: .note }

完整列表`opensearch.yml` 安全插件设置，安全插件设置，请参阅[安全设定]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/security-settings/)。
{： 。笔记

