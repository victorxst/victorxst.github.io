---
layout: default
title: 将更改应用于配置文件
parent: 配置
nav_order: 25
redirect_from:
  - /security-plugin/configuration/security-admin/
---

# 将更改应用于配置文件

在**视窗**， 使用**SecurityAdmin.bat** 代替**SecurityAdmin.sh**。有关更多信息，请参阅[Windows用法](#windows-usage)。
{: .note}

安全插件存储其配置（包括用户，角色，权限和后端设置）[系统索引]({{site.url}}{{site.baseurl}}/security/configuration/system-indices) 在OpenSearch集群上。将这些设置存储在索引中使您可以更改设置，而无需重新启动群集，并消除了在每个节点上编辑配置文件的需求。这是通过运行`securityadmin.sh` 脚本。

脚本的第一个工作是初始化`.opendistro_security` 指数。这将您的初始配置加载到索引中，使用配置文件`/config/opensearch-security`。之后`.opendistro_security` 索引是初始化的，您可以使用OpenSearch仪表板或REST API来管理您的用户，角色和权限。

该脚本可以在`/plugins/opensearch-security/tools/securityadmin.sh`。这是一条相对路径，显示`securityadmin.sh` 脚本已找到。绝对路径取决于您已安装OpenSearch的目录。例如，如果您使用Docker安装OpenSearch，则该路径将类似于以下内容：`/usr/share/opensearch/plugins/opensearch-security/tools/securityadmin.sh`。

## 谨慎

如果您更改了配置文件`config/opensearch-security`，opensearch do _not_自动应用这些更改。相反，您必须运行`securityadmin.sh` 将更新的文件加载到索引中。

跑步`securityadmin.sh` **覆盖** 一个或多个部分`.opendistro_security` 指数。非常小心地运行它，以避免失去现有资源。考虑以下示例：

1. 您初始化`.opendistro_security` 指数。
1. 您使用REST API创建十个用户。
1. 您决定创建一个新的[保留用户]({{site.url}}{{site.baseurl}}/security/access-control/api/#reserved-and-hidden-resources) 使用`internal_users.yml`。
1. 你跑`securityadmin.sh` 再次将新的保留用户加载到索引中。
1. 您将失去使用REST API创建的所有十个用户。

为了避免这种情况，请在更改之前备份当前的配置并重新-运行脚本：

```bash
./securityadmin.sh -backup my-backup-directory \
  -icl \
  -nhnv \
  -cacert ../../../config/root-ca.pem \
  -cert ../../../config/kirk.pem \
  -key ../../../config/kirk-key.pem
```

如果您使用`-f` 争论而不是`-cd`，您可以将一个YAML文件加载到索引中，而不是YAML文件的整个目录。例如，如果您创建十个新角色，则可以安全加载`internal_users.yml` 进入指数而不会失去角色；只有内部用户被覆盖。

```bash
./securityadmin.sh -f ../../../config/opensearch-security/internal_users.yml \
  -t internalusers \
  -icl \
  -nhnv \
  -cacert ../../../config/root-ca.pem \
  -cert ../../../config/kirk.pem \
  -key ../../../config/kirk-key.pem
```

要在应用安全配置之前解决所有环境变量，请使用`-rev` 范围。

```bash
./securityadmin.sh -cd ../../../config/opensearch-security/ \
 -rev \
 -cacert ../../../root-ca.pem \
 -cert ../../../kirk.pem \
 -key ../../../kirk.key.pem
```

以下示例显示了环境变量`config.yml` 文件：

```yml
password: ${env.LDAP_PASSWORD}
```

## 配置管理证书

为了使用`securityadmin.sh`，您必须将所有管理证书的杰出名称（DNS）添加到`opensearch.yml`。如果您使用演示证书，例如`opensearch.yml` 可能包含以下几行`kirk` 证书：

```yml
plugins.security.authcz.admin_dn:
  - CN=kirk,OU=client,O=client,L=test,C=DE
```

您不能将节点证书用作管理证书。两者必须分开。另外，请勿在DN的各个部分之间添加空格。
{: .warning }


## 基本用法

这`securityadmin.sh` 可以从任何可以访问OpenSearch集群HTTP端口的机器运行工具（默认端口为9200）。您可以更改安全插件配置，而无需通过SSH访问节点。

`securityadmin.sh` 要求在OpenSearch集群上启用SSL/TLS传输。换句话说，请确保`plugins.security.ssl.http.enabled: true` 安顿好了`opensearch.yml` 继续前进。
{: .note}

每个节点还包括该工具`plugins/opensearch-security/tools/securityadmin.sh`。您可能需要在运行脚本之前使脚本可执行：

```bash
chmod +x plugins/opensearch-security/tools/securityadmin.sh
```

要打印所有可用的命令行选项，请在没有参数的情况下运行脚本：

```bash
./plugins/opensearch-security/tools/securityadmin.sh
```

要加载您的初始配置（所有YAML文件），您可以使用以下命令：

```bash
./securityadmin.sh -cd ../../../config/opensearch-security/ -icl -nhnv \
  -cacert ../../../config/root-ca.pem \
  -cert ../../../config/kirk.pem \
  -key ../../../config/kirk-key.pem
```

- 这`-cd` 选项指定可以找到安全插件配置文件的位置。
- 这`-icl` （（`--ignore-clustername`）选项告诉安全插件上传配置，无论群集名称如何。作为替代方案，您还可以用`-cn` （（`--clustername`） 选项。
- 因为演示证书是自我的-签名，此命令禁用使用主机名验证`-nhnv` （（`--disable-host-name-verification`） 选项。
- 这`-cacert`，`-cert` 和`-key` 选项定义了您的根CA证书的位置，管理员证书和管理证书的私钥。如果私钥有密码，请使用`-keypass` 选项。

下表显示了PEM选项。

姓名| 描述
:--- | :---
`-cert` | 包含管理证书和所有中间证书的PEM文件的位置（如果有）。您可以使用绝对路径或相对路径。相对路径相对于执行目录解决`securityadmin.sh`。
`-key` | PEM文件的位置包含管理证书的私钥。您可以使用绝对路径或相对路径。相对路径相对于执行目录解决`securityadmin.sh`。关键必须在PKC中#8格式。
`-keypass` | 管理证书的私钥的密码（如果有）。
`-cacert` | 包含根证书的PEM文件的位置。您可以使用绝对路径或相对路径。相对路径相对于执行目录解决`securityadmin.sh`。


## 示例命令

将所有YAML文件应用于`config/opensearch-security/` 使用PEM证书：

```bash
/usr/share/opensearch/plugins/opensearch-security/tools/securityadmin.sh \
  -cacert /etc/opensearch/root-ca.pem \
  -cert /etc/opensearch/kirk.pem \
  -key /etc/opensearch/kirk-key.pem \
  -cd /usr/share/opensearch/config/opensearch-security/
```

应用一个yaml文件（`config.yml`）使用PEM证书：

```bash
./securityadmin.sh \
  -f ../../../config/opensearch-security/config.yml \
  -icl -nhnv -cert /etc/opensearch/kirk.pem \
  -cacert /etc/opensearch/root-ca.pem \
  -key /etc/opensearch/kirk-key.pem \
  -t config
```

将所有YAML文件应用于`config/opensearch-security/` 使用密钥库和信托店文件：

```bash
./securityadmin.sh \
  -cd /usr/share/opensearch/config/opensearch-security/ \
  -ks /path/to/keystore.jks \
  -kspass changeit \
  -ts /path/to/truststore.jks \
  -tspass changeit
  -nhnv
  -icl
```


## 将SecurityAdmin与密钥库和TrustStore文件一起使用

您还可以将JKS格式的密钥库文件与`securityadmin.sh`：

```bash
./securityadmin.sh -cd ../../../config/opensearch-security -icl -nhnv
  -ts <path/to/truststore> -tspass <truststore password>
  -ks <path/to/keystore> -kspass <keystore password>
```

使用以下选项控制密钥和信托店设置。

姓名| 描述
:--- | :---
`-ks` | 包含管理员证书和所有中间证书的密钥库的位置（如果有）。您可以使用绝对路径或相对路径。相对路径相对于执行目录解决`securityadmin.sh`。
`-kspass` | 密钥库的密码。
`-kst` | 钥匙店类型是JKS或PKCS#12/pfx。如果未指定，则安全插件将尝试确定文件扩展名的类型。
`-ksalias` | 管理证书的别名（如果有）。
`-ts` | 包含根证书的信托店的位置。您可以使用绝对路径或相对路径。相对路径相对于执行目录解决`securityadmin.sh`。
`-tspass` | 信托店的密码。
`-tst` | TrustStore类型，JKS或PKCS#12/pfx。如果未指定，则安全插件将尝试确定文件扩展名的类型。
`-tsalias` | 根证书的别名（如果有）。


### OpenSearch设置

如果您运行默认的OpenSearch安装，该安装在端口9200上lisk并使用`opensearch` 作为集群名称，您可以完全省略以下设置。否则，使用以下开关来指定OpenSearch设置。

姓名| 描述
:--- | :---
`-h` | OpenSearch HostName。默认为`localhost`。
`-p` | OpenSearch端口。默认值为9200- 不是HTTP端口。
`-cn` | 集群名称。默认为`opensearch`。
`-icl` | 忽略群集名称。
`-sniff` | 嗅探群节点。使用OpenSearch嗅探检测可用的节点`_cluster/state` API。
`-arc,--accept-red-cluster` | 执行`securityadmin.sh` 即使群集状态为红色。默认值为false，这意味着该脚本不会在红色群集上执行。


### 证书验证设置

使用以下选项控制证书验证。

姓名| 描述
:--- | :---
`-nhnv` | 请勿验证主机名。默认值为false。
`-nrhn` | 请勿解决主机名。仅相关`-nhnv` 未设置。
`-noopenssl` | 即使可用，也不要使用OpenSSL。如果可用，则默认为使用OpenSSL。


### 配置文件设置

以下开关定义要将哪些配置文件推入安全插件。您可以按单个文件或指定包含一个或多个配置文件的目录。

姓名| 描述
:--- | :---
`-cd` | 包含多个安全插件配置文件的目录。
`-f` | 单个配置文件。不能与`-cd`。
`-t` | 文件类型。
`-rl` | 重新加载当前配置并冲洗内部缓存。

要在目录中上传所有配置文件，请使用以下方式：

```bash
./securityadmin.sh -cd ../../../config/opensearch-security -ts ... -tspass ... -ks ... -kspass ...
```

如果要推开单个配置文件，请使用以下方式：

```bash
./securityadmin.sh -f ../../../config/opensearch-security/internal_users.yml -t internalusers  \
    -ts ... -tspass ... -ks ... -kspass ...
```

文件类型必须是以下之一：

* config
*角色
*角色图
*内部使用者
*行动组


### 密码设置

您可能不需要更改密码设置。如果需要，请使用以下选项。

姓名| 描述
:--- | :---
`-ec` | 逗号-启用的TLS密码的分开列表。
`-ep` | 逗号-启用TLS协议的分开列表。


### 备份，还原和迁移

您可以使用以下命令从群集下载所有当前配置文件：

```bash
./securityadmin.sh -backup my-backup-directory -ts ... -tspass ... -ks ... -kspass ...
```

此命令将当前的安全插件配置从您指定的目录中从群集中转移到单个文件。然后，您可以将这些文件用作备份，也可以将配置加载到其他群集中。移动证明时此命令很有用-的-生产的概念，或者您需要添加其他[保留或隐藏的资源]({{site.url}}{{site.baseurl}}/security/access-control/api/#reserved-and-hidden-resources)：

```bash
./securityadmin.sh \
  -backup my-backup-directory \
  -icl \
  -nhnv \
  -cacert ../../../config/root-ca.pem \
  -cert ../../../config/kirk.pem \
  -key ../../../config/kirk-key.pem
```

要将倾倒文件上传到另一个群集：

```bash
./securityadmin.sh -h production.example.com -p 9301 -cd /etc/backup/ -ts ... -tspass ... -ks ... -kspass ...
```

要迁移配置yaml文件从eLasticsearch 0.x.x格式的开放式发行版迁移到OpenSearch 1.x.x格式：

```bash
./securityadmin.sh -migrate ../../../config/opensearch-security -ts ... -tspass ... -ks ... -kspass ...
```

姓名| 描述
:--- | :---
`-backup` | 从运行群集中检索当前的安全插件配置，然后将其转储到工作目录。
`-migrate` | 迁移配置YAML文件从for Elasticsearch 0.x.x到OpenSearch 1.x.x.。


### 其他选项

姓名| 描述
:--- | :---
`-dci` | 删除安全插件配置索引和退出。如果由于损坏的安全插件索引，群集状态为红色，则此选项很有用。
`-esa` | 启用碎片分配和退出。如果您在执行完整的群集重新启动并需要重新创建安全插件索引时禁用碎片分配，则此选项很有用。
`-w` | 显示有关使用过的管理员证书的信息。
`-rl` | 默认情况下，安全插件缓存了经过身份验证的用户以及其角色和权限一小时。此选项重新加载群集中存储的当前安全插件配置，使任何缓存的用户，角色和权限无效。
`-i` | 安全插件索引名称。默认为`.opendistro_security`。
`-er` | 设置明确数量的复制品或自动-扩展表达`opensearch_security` 指数。
`-era` | 启用副本自动-扩张。
`-dra` | 禁用复制自动-扩张。
`-us` | 更新复制设置。

## Windows用法

在窗户上，相当于`securityadmin.sh` 是个`securityadmin.bat` 脚本位于`\path\to\opensearch-{{site.opensearch_version}}\plugins\opensearch-security\tools\` 目录。

在前面的部分中运行示例命令时，请使用**命令提示符** 或者**电源外壳**。通过输入打开命令提示`cmd` 或通过进入的powershell`powershell` 在旁边的搜索框中**开始** 在任务栏上。

例如，要打印所有可用命令行选项，请在没有参数的情况下运行脚本：

```bat
.\plugins\opensearch-security\tools\securityadmin.bat
```

输入多行命令时，请使用Caret（`^`）字符逃脱命令行中的下一个字符。

例如，要加载您的初始配置（所有YAML文件），请使用以下命令：

```bat
.\securityadmin.bat -cd ..\..\..\config\opensearch-security\ -icl -nhnv ^
  -cacert ..\..\..\config\root-ca.pem ^
  -cert ..\..\..\config\kirk.pem ^
  -key ..\..\..\config\kirk-key.pem
```
