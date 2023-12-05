---
layout: default
title: 修改yaml文件
parent: 配置
nav_order: 10
redirect_from: 
  - /security-plugin/configuration/yaml/
---

# 修改yaml文件

安全安装提供了许多YAML配置文件，用于存储必要的设置，以定义安全插件在集群中管理用户，角色和活动的方式。这些设置范围从身份验证后端的配置到允许端点和HTTP请求的列表。

运行前[`securityadmin.sh`]({{site.url}}{{site.baseurl}}/security/configuration/security-admin/) 将设置加载到`.opendistro_security` 索引，执行YAML文件的初始配置。这些文件可以在`config/opensearch-security` 目录。备份这些文件也是一个好练习，以便您可以将它们重复使用其他群集。

我们建议使用YAML文件的方法是首先配置[保留和隐藏的资源]({{site.url}}{{site.baseurl}}/security/access-control/api#reserved-and-hidden-resources)， 如那个`admin` 和`kibanaserver` 用户。此后，您可以使用OpenSearch仪表板或REST API创建其他用户，角色，映射组，行动组和租户。


## internal_users.yml

该文件包含要添加到安全插件的内部用户数据库中的任何初始用户。

文件格式需要哈希密码。要生成一个，运行`plugins/opensearch-security/tools/hash.sh -p <new-password>`。如果您决定保留任何演示用户，请 *更改其密码 *并重新-跑步[SecurityAdmin.sh]({{site.url}}{{site.baseurl}}/security/configuration/security-admin/) 应用新密码。

```yml
---
# This is the internal user database
# The hash value is a bcrypt hash and can be generated with plugin/tools/hash.sh

_meta：
  类型："internalusers"
  config_version：2

# Define your internal users here
新的-用户：
  哈希："$2y$12$88IFVl6IfIwCFh5aQYfOmuXVL9j2hz/GusQb35o.4sdTDAEMTOD.K"
  保留：错误
  隐藏：false
  opendistro_security_roles：
  - "specify-some-security-role-here"
  Backend_roles：
  - "specify-some-backend-role-here"
  属性：
    属性1："value1"
  静态：错误

## Demo users

行政：
  哈希："$2a$12$VcCDgh2NDk07JGN0rjGbM.Ad41qVR/YFJcgHp0UGns5JDymv..TOG"
  保留：是的
  Backend_roles：
  - "admin"
  描述："Demo admin user"

Kibanaserver：
  哈希："$2a$12$4AcgAt3xwOWadA5s5blL6ev39OXDNhmOesEoo33eZtrq2N0YrU3H."
  保留：是的
  描述："Demo user for the OpenSearch Dashboards server"

Kibanaro：
  哈希："$2a$12$JJSXNfTowz7Uu5ttXfeYpeYE0arACvcwlPBStB1F.MI7f0U9Z4DGC"
  保留：错误
  Backend_roles：
  - "kibanauser"
  - "readall"
  属性：
    属性1："value1"
    属性2："value2"
    属性3："value3"
  描述："Demo read-only user for OpenSearch dashboards"

logstash：
  哈希："$2a$12$u1ShR4l4uBS3Uv59Pa2y5.1uQuZBrZtmNfqB3iM/.jL0XoV9sghS2"
  保留：错误
  Backend_roles：
  - "logstash"
  描述："Demo logstash user"

readall：
  哈希："$2a$12$ae4ycwzwvLtZxwZ82RmiEunBbIPiAmGZduBAjKN0TXdwQFtCwARz2"
  保留：错误
  Backend_roles：
  - "readall"
  描述："Demo readall user"

Snapshotrestore：
  哈希："$2y$12$DpwmetHKwgYnorbgdvORCenv4NAK8cPUg8AI6pxLCuWf/ALc0.v7W"
  保留：错误
  Backend_roles：
  - "snapshotrestore"
  描述："Demo snapshotrestore user"
```

## opensearch.yml

除了许多OpenSearch设置外，此文件还包含通往TLS证书及其属性的途径，例如杰出的名称和受信任的证书授权。

```yml
plugins.security.ssl.transport.pemcert_filepath：esnode.pem
plugins.security.ssl.transport.pemkey_filepath：esnode-key.pem
plugins.security.ssl.transport.pemtrustedcas_filepath：root-ca.pem
plugins.security.ssl.transport.enforce_hostname_verification：false
plugins.security.ssl.http.enabled：true
plugins.security.ssl.http.pemcert_filepath：esnode.pem
plugins.security.ssl.http.pemkey_filepath：esnode-key.pem
plugins.security.ssl.http.pemtrustedcas_filepath：root-ca.pem
plugins.security.allow_unsafe_democertificates：true
plugins.security.allow_default_init_securityIndex：true
plugins.security.authcz.admin_dn：
  - cn = kirk，ou =客户端，o =客户端，l = test，c = de

plugins.security.audit.type：internal_opensearch
plugins.security.enable_snapshot_restore_privilege：true
plugins.security.check_snapshot_restore_write_privileges：true
plugins.security.cache.ttl_minutes：60
plugins.security.restapi.Roles_Enabled：["all_access"，，，，"security_rest_api_access"这是给出的
plugins.security.system_indices.enabled：true
plugins.security.system_indices.indices：[".opendistro-alerting-config"，，，，".opendistro-alerting-alert*"，，，，".opendistro-anomaly-results*"，，，，".opendistro-anomaly-detector*"，，，，".opendistro-anomaly-checkpoints"，，，，".opendistro-anomaly-detection-state"，，，，".opendistro-reports-*"，，，，".opendistro-notifications-*"，，，，".opendistro-notebooks"，，，，".opendistro-asynchronous-search-response*"这是给出的
node.max_local_storage_nodes：3
```

完整列表`opensearch.yml` 安全插件设置，请参见[安全设置]（{{stite.url}}} {{site.baseurl}}/install-和-配置/配置-OpenSearch/Security-设置/）。
{: .note}

### Refining your configuration

这`plugins.security.allow_default_init_securityindex` 设置，设置为`true`，如果尝试在OpenSearch启动时尝试创建安全索引失败，则将安全插件设置为默认安全设置。默认安全设置存储在包含的YAML文件中`opensearch-project/security/config` 目录。默认情况下，此设置为`false`。

```yml
plugins.security.allow_default_init_securityIndex：true
```

存在安全插件的身份验证缓存，可以通过临时存储从后端返回的用户对象来帮助加快身份验证，以便不需要安全插件为其重复请求。要确定缓存时间需要多长时间，您可以使用`plugins.security.cache.ttl_minutes` 属性以分钟为单位设置值。默认值为`60`。您可以通过将值设置为`0`。

```yml
plugins.security.cache.ttl_minutes：60
```

### Enabling user access to system indexes

将系统索引权限映射给用户允许该用户修改在权限名称中指定的系统索引（一个例外是安全插件的[系统索引]（{{site.url}}} {site.baseurl}}}/配置/系统-索引/）。这`plugins.security.system_indices.permissions.enabled` 设置为管理员提供了一种使此权限可用于角色映射或隐藏的权限。

设置为`true`，该功能已启用，并且有权修改角色的用户可以创建角色，其中包括授予对系统索引的访问权限的权限：

```yml
plugins.security.system_indices.permissions.enabled：true
```

设置为`false`，禁用权限，只有使用管理证书的管理员才能更改系统索引。默认情况下，权限设置为`false` 在一个新集群中。

要了解有关系统索引许可的更多信息，请参见[系统索引权限]（{{site.url}}} {{site.baseurl}}/security/access-控制/权限/#系统-指数-权限）。


### Password settings

如果要针对某些验证运行用户的密码，请在此文件中指定正则表达式（REGEX）。您还可以包括一条错误消息，该消息当密码未通过验证时加载。下面的示例演示了如何包含正则搜索，因此OpenSearch要求新密码至少为八个字符，其中至少一个大写，一个小写，一个数字和一个特殊字符。

请注意，OpenSearch仅验证通过OpenSearch仪表板或REST API创建的用户和密码。

```yml
plugins.security.restapi.password_validation_regex：'（？=。*[a-z]）（？=。*[^a-ZA-z \ d]）（？=。*[0-9]））（？=。*[a-z]）。{8，}'
plugins.security.restapi.password_validation_error_message："Password must be minimum 8 characters long and must contain at least one uppercase letter, one lowercase letter, one digit, and one special character."
```

此外，得分-基于密码强度估计器允许您在创建新的内部用户或更新用户密码时为密码强度设置阈值。此功能利用[ZXCVBN库]（https://github.com/dropbox/zxcvbn）应用一项强调密码复杂性的策略，而不是符合传统标准的能力，例如大写键，数字，数字和特殊角色。

有关创建用户的信息，请参见[创建用户]（{{site.url}}} {{site.baseurl}}/security/access-控制/用户-角色/#创造-用户）。

此功能与指定为保留的用户不兼容。有关保留资源的信息，请参见[保留和隐藏资源]（{{stite.url}}} {{site.baseurl}}/security/access-控制/API#预订的-和-隐-资源）。
{: .important }

分数-基于密码的强度需要两个设置来配置该功能。下表描述了两个设置。

| 环境| 描述|
| ：--- | ：--- |
| `plugins.security.restapi.password_min_length` | 设置密码长度的最小字符数。默认值为`8`。这也是最低限度。|
| `plugins.security.restapi.password_score_based_validation_strength` | 设置一个阈值，以确定密码是强还是弱。有四个代表阈值增加的复杂性。<br>`fair`--非常"guessable" 密码：提供防止在线攻击的保护。<br>`good`--一个有点猜测的密码：提供防止在线攻击的保护。<br>`strong`--安全"unguessable" 密码：提供适度的保护免受离线，缓慢的保护-哈希方案。<br>`very_strong`--一个非常无关紧要的密码：提供强烈的保护免受离线，慢速的保护-哈希场景。|

以下示例显示了为`opensearch.yml` 文件并启用至少10个字符的密码和一个需要最高强度的阈值：

```yml
plugins.security.restapi.password_min_length：10
plugins.security.restapi.password_score_based_validation_strength：esumy_strong
```

当您尝试使用未达到指定阈值的密码创建用户时，系统会生成一个"weak password" 警告，表明需要在保存用户之前修改密码。

以下示例显示了[create user]（{{stite.url}}} {{site.baseurl}}/security/access/access-控制/API/#创造-用户）API当密码较弱时：

```json
{
  "status"："error"，，，，
  "reason"："Weak password"
}
```

## allowlist.yml

您可以使用`allowlist.yml` 要将任何端点和HTTP请求添加到允许的端点和请求列表中。如果启用，则仅允许除超级管理员以外的所有用户访问指定的端点和HTTP请求，并且拒绝与端点关联的所有其他HTTP请求。例如，如果得到`_cluster/settings` 被添加到允许列表中，用户无法将PUT请求提交给`_cluster/settings` 更新集群设置。

请注意，尽管您可以通过这种方式配置对端点的访问，但是对于大多数情况下，最好使用安全插件的用户和角色配置权限，这些用户和角色具有更精细的设置。

```yml
---
_meta:
  type: "allowlist"
  config_version: 2

# Description:
# enabled - feature flag.
# if enabled is false, all endpoints are accessible.
# if enabled is true, all users except the SuperAdmin can only submit the allowed requests to the specified endpoints.
# SuperAdmin can access all APIs.
# SuperAdmin is defined by the SuperAdmin certificate, which is configured with the opensearch.yml setting plugins.security.authcz.admin_dn:
# Refer to the example setting in opensearch.yml to learn more about configuring SuperAdmin.
#
# requests - map of allow listed endpoints and HTTP requests

#this name must be config
config:
  enabled: true
  requests:
    /_cluster/settings:
      - GET
    /_cat/nodes:
      - GET
```

要启用对集群设置的请求，请添加到允许操作的列表中`/_cluster/settings`。

```yml
requests:
  /_cluster/settings:
    - GET
    - PUT
```

您还可以将自定义索引添加到允许列表中。`allowlist.yml` 不支持通配符，因此您必须手动指定要添加的所有索引。

```yml
requests: # Only allow GET requests to /sample-index1/_doc/1 and /sample-index2/_doc/1
  /sample-index1/_doc/1:
    - GET
  /sample-index2/_doc/1:
    - GET
```


## 角色

该文件包含要添加到安全插件的任何初始角色。除了某些元数据外，默认文件是空的，因为安全插件具有自动添加的许多静态角色。

```yml
---
复杂的-角色：
  保留：错误
  隐藏：false
  cluster_permissions：
  - "read"
  - "cluster:monitor/nodes/stats"
  - "cluster:monitor/task/get"
  index_permissions：
  - index_patterns：
    - "opensearch_dashboards_sample_data_*"
    DLS："{\"匹配\": {\"飞机延迟\": true}}"
    FLS：
    - "~FlightNum"
    masked_fields：
    - "Carrier"
    允许的_actions：
    - "read"
  tenant_permissions：
  - tenant_patterns：
    - "analyst_*"
    允许的_actions：
    - "kibana_all_write"
  静态：错误
_meta：
  类型："roles"
  config_version：2
```


## roles_mapping.yml

```yml
---
manage_snapshots:
  reserved: true
  hidden: false
  backend_roles:
  - "snapshotrestore"
  hosts: []
  users: []
  and_backend_roles: []
logstash:
  reserved: false
  hidden: false
  backend_roles:
  - "logstash"
  hosts: []
  users: []
  and_backend_roles: []
own_index:
  reserved: false
  hidden: false
  backend_roles: []
  hosts: []
  users:
  - "*"
  and_backend_roles: []
  description: "Allow full access to an index named like the username"
kibana_user:
  reserved: false
  hidden: false
  backend_roles:
  - "kibanauser"
  hosts: []
  users: []
  and_backend_roles: []
  description: "Maps kibanauser to kibana_user"
complex-role:
  reserved: false
  hidden: false
  backend_roles:
  - "ldap-analyst"
  hosts: []
  users:
  - "new-user"
  and_backend_roles: []
_meta:
  type: "rolesmapping"
  config_version: 2
all_access:
  reserved: true
  hidden: false
  backend_roles:
  - "admin"
  hosts: []
  users: []
  and_backend_roles: []
  description: "Maps admin to all_access"
readall:
  reserved: true
  hidden: false
  backend_roles:
  - "readall"
  hosts: []
  users: []
  and_backend_roles: []
kibana_server:
  reserved: true
  hidden: false
  backend_roles: []
  hosts: []
  users:
  - "kibanaserver"
  and_backend_roles: []
```


## action_groups.yml

该文件包含要添加到安全插件的任何初始操作组。

除了某些元数据外，默认文件是空的，因为安全插件具有许多静态操作组，它们会自动添加。这些静态动作组涵盖了多种用例，是插件开始的好方法。

```yml
---
我的-行动-团体：
  保留：错误
  隐藏：false
  允许的_actions：
  - "indices:data/write/index*"
  - "indices:data/write/update*"
  - "indices:admin/mapping/put"
  - "indices:data/write/bulk*"
  - "read"
  - "write"
  静态：错误
_meta：
  类型："actiongroups"
  config_version：2
```

## tenants.yml

您可以使用此文件来指定并将任何数量的OpenSearch仪表板租户添加到OpenSearch集群中。有关租户的更多信息，请参见[OpenSearch仪表板Multi-租赁]（{{stite.url}}} {{site.baseurl}}/security/security/multi-租赁/租户-指数）。

像所有其他YAML文件一样，我们建议您使用`tenants.yml` 要添加您必须在群集中拥有的任何租户，然后使用OpenSearch仪表板或[REST API]（{stite.url}}} {site.baseurl}}/security/access-控制/API/#租户）如果您需要进一步配置或创建任何其他租户。

```yml
---
_meta:
  type: "tenants"
  config_version: 2
admin_tenant:
  reserved: false
  description: "Demo tenant for admin user"
```

## nodes_dn.yml

`nodes_dn.yml` 让您添加证书”[杰出名称（DNS）]({{site.url}}{{site.baseurl}}/security/configuration/generate-certificates/#add-distinguished-names-to-opensearchyml) 允许列表可以在任何数量的节点和/或簇之间进行通信。例如，具有DN的节点`CN=node1.example.com` 在其允许列表中，接受使用该DN的任何其他节点或证书的通信。

DNS被索引到[系统索引]({{site.url}}{{site.baseurl}}/security/configuration/system-indices) 只有超级管理员或具有运输层安全性（TLS）证书的管理员才能访问。如果要编程将DNS添加到允许列表中，请使用[REST API]({{site.url}}{{site.baseurl}}/security/access-control/api/#distinguished-names)。

```yml
---
_meta：
  类型："nodesdn"
  config_version：2

# Define nodesdn mapping name and corresponding values
# cluster1:
#   nodes_dn:
#       - CN=*.example.com
```

