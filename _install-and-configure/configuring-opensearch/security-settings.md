---
layout: default
title: 安全设置
parent: 配置 OpenSearch
nav_order: 40
---

# 安全设置

安全插件提供了许多 YAML 配置文件，用于存储必要的设置，这些设置定义了安全插件管理集群内用户、角色和活动的方式。有关安全插件配置文件的完整列表，请参见[修改 YAML 文件]({{site.url}}{{site.baseurl}}/security/configuration/yaml/)。

以下各节介绍了中 `opensearch.yml` 与安全相关的设置。要了解有关静态和动态设置的详细信息，请参阅[配置 OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)。

## 通用设置

Security 插件支持以下常见设置：

-   `plugins.security.nodes_dn`（静态）：指定表示群集中其他节点的可分辨名称（DN）列表。此设置支持通配符和正则表达式。当 is `true` 时 `plugins.security.nodes_dn_dynamic_config_enabled`，还会将 DN 列表从安全索引**另外**读取到 YAML 配置中。

-  `plugins.security.nodes_dn_dynamic_config_enabled`（静态）：适用于 `cross_cluster` 需要管理列出的 `nodes_dn` 允许而无需在每次配置新 `cross_cluster` 远程时重新启动节点的用例。设置为 `nodes_dn_dynamic_config_enabled` `true` 启用**超级管理员可调用**可分辨名称 API，这些 API 提供动态更新或检索 `nodes_dn` 的方法。此设置仅在未设置时 `plugins.security.cert.intercluster_request_evaluator_class` 有效。缺省值为 `false`。

-  `plugins.security.authcz.admin_dn`（静态）：定义应向其分配管理员权限的证书的 DN。必填。

-  `plugins.security.roles_mapping_resolution`（静态）：定义后端角色如何映射到安全角色。
        
    有效值为：
    -  `MAPPING_ONLY`（缺省值）：必须在中显式 `roles_mapping.yml` 配置映射。
    -  `BACKENDROLES_ONLY`：后端角色直接映射到安全角色。中的 `roles_mapping.yml` 设置不起作用。
    -  `BOTH`：后端角色直接映射到安全角色， `roles_mapping.yml` 并通过映射到安全角色。

-  `plugins.security.dls.mode`（静态）：设置文档级安全性（DLS）评估模式。缺省值为 `adaptive`。请参见[如何设置 DLS 评估模式]({{site.url}}{{site.baseurl}}/security/access-control/document-level-security/#how-to-set-the-dls-evaluation-mode-in-opensearchyml)。

-  `plugins.security.compliance.salt`（静态）：生成用于字段掩码的哈希值时要使用的盐。必须至少为 32 个字符。只允许使用 ASCII 字符。自选。

-  `config.dynamic.http.anonymous_auth_enabled`（静态）：启用匿名身份验证。这将导致所有 HTTP 身份验证器不进行质询。缺省值为 `false`。

## REST 管理 API 设置

安全插件支持以下 REST 管理 API 设置：

-  `plugins.security.restapi.roles_enabled`（静态）：为列出的角色启用对 REST 管理 API 的基于角色的访问。角色之间用逗号分隔。默认值为空列表（不允许任何角色访问 REST 管理 API）。请参见[API 的访问控制]({{site.url}}{{site.baseurl}}/security/access-control/api/#access-control-for-the-api)。

-  `plugins.security.restapi.endpoints_disabled.<role>.<endpoint>`（静态）：禁用角色的特定终结点及其 HTTP 方法。此设置的值组成一个 HTTP 方法数组。例如： `plugins.security.restapi.endpoints_disabled.all_access.ACTIONGROUPS: ["PUT","POST","DELETE"]`.默认情况下，允许使用所有终结点和方法。现有端点包括 `ACTIONGROUPS`、、、、 `ROLES` `ROLESMAPPING`、 `SYSTEMINFO` `LICENSE` `CACHE` `CONFIG` `INTERNALUSERS` `PERMISSIONSINFO`、和。请参见[API 的访问控制]({{site.url}}{{site.baseurl}}/security/access-control/api/#access-control-for-the-api)。

-  `plugins.security.restapi.password_validation_regex`（静态）：指定用于设置登录密码条件的正则表达式。有关详细信息，请参阅[密码设置]({{site.url}}{{site.baseurl}}/security/configuration/yaml/#password-settings)。

-  `plugins.security.restapi.password_validation_error_message`（静态）：指定在密码未通过验证时加载的错误消息。此设置与 `plugins.security.restapi.password_validation_regex` 结合使用。

-  `plugins.security.restapi.password_min_length`（静态）：使用基于分数的密码强度估算器时，设置密码长度的最小字符数。默认值为 8. 这也是最低限度。有关详细信息，请参阅[密码设置]({{site.url}}{{site.baseurl}}/security/configuration/yaml/#password-settings)。

-  `plugins.security.restapi.password_score_based_validation_strength`（静态）：设置阈值以确定密码是强密码还是弱密码。有效值为 `fair`、、 `good` `strong` 和 `very_strong`。此设置与 `plugins.security.restapi.password_min_length` 结合使用。

-  `plugins.security.unsupported.restapi.allow_securityconfig_modification`（静态）：允许对配置 API 使用 PUT 和 PATCH 方法。

## 高级设置

安全插件支持以下高级设置：

-  `plugins.security.authcz.impersonation_dn`（静态）：启用传输层模拟。这允许 DN 冒充其他用户。请参见[用户模拟]({{site.url}}{{site.baseurl}}/security/access-control/impersonation/)。

-  `plugins.security.authcz.rest_impersonation_user`（静态）：启用 REST 层模拟。这允许用户冒充其他用户。请参见[用户模拟]({{site.url}}{{site.baseurl}}/security/access-control/impersonation/)。

-  `plugins.security.allow_default_init_securityindex`（静态）： `true` 设置为时，如果索引不存在，OpenSearch Security 将使用目录中的文件 `/config` 自动初始化配置索引。

  这将使用已知的默认密码。仅在专用网络/环境中使用。
{: .warning}

-  `plugins.security.allow_unsafe_democertificates`（静态）：设置为 `true` 时，OpenSearch 将使用演示证书启动。这些证书仅用于演示目的。

  这些证书是众所周知的，因此对生产不安全。仅在专用网络/环境中使用。
{: .warning}

-  `plugins.security.system_indices.permission.enabled`（静态）：开启系统索引权限功能。设置为 `true` 时，将启用该功能，并且有权修改角色的用户可以创建包含授予系统索引访问权限的权限的角色。设置为 `false` 时，权限将被禁用，只有拥有管理员证书的管理员才能对系统索引进行更改。默认情况下，权限设置为 `false` 在新集群中。

## 专家级设置

专家级设置只能由完全了解该功能的管理员进行配置和部署。对功能的误解可能会导致安全风险，导致安全插件无法正常运行，或导致数据丢失。
{: .warning}

安全插件支持以下专家级设置：

-  `plugins.security.config_index_name`（静态）：存储其配置的 `.opendistro_security` 索引的名称。

-  `plugins.security.cert.oid`（静态）：定义服务器节点证书的对象标识符（OID）。

-  `plugins.security.cert.intercluster_request_evaluator_class`（静态）：指定用于评估集群间请求的 `org.opensearch.security.transport.InterClusterRequestEvaluator` 实现。的 `org.opensearch.security.transport.InterClusterRequestEvaluator` 实例必须实现接受 `org.opensearch.common.settings.Settings` 对象的单参数构造函数。

-  `plugins.security.enable_snapshot_restore_privilege`（静态）：设置为 `false` 时，此设置将禁用常规用户的快照还原。在这种情况下，仅接受由管理员 TLS 证书签名的快照还原请求。设置为 `true`（默认）时，如果普通用户具有 `cluster:admin/snapshot/restore`、 `indices:admin/create` 和 `indices:data/write/index` 权限，则可以还原快照。

  只有当快照不包含全局状态且不还原索引时，才能还原 `.opendistro_security` 快照。
{:.note}

-  `plugins.security.check_snapshot_restore_write_privileges`（静态）：设置为 `false` 时，将省略其他索引检查。当设置为默认值 `true` 时，将评估 `indices:admin/create` 和 `"indices:data/write/index` 的恢复快照尝试。

-  `plugins.security.cache.ttl_minutes`（静态）：确定身份验证缓存超时所需的时间。身份验证缓存通过临时存储从后端返回的用户对象来帮助加快身份验证速度，这样 Security 插件就不需要对它们发出重复请求。以分钟为单位设置值。缺省值为 `60`。通过将值设置为 `0` 来禁用缓存。

-  `plugins.security.disabled`（静态）：禁用 OpenSearch 安全性。

  禁用此插件可能会将你的配置（包括密码）暴露给公众。{：警告}

-  `plugins.security.protected_indices.enabled`（静态）：如果设置为 `true`，则启用受保护的索引。受保护的索引甚至比常规索引更安全。这些索引需要一个角色才能像任何其他传统索引一样进行访问，并且需要一个额外的角色才能可见。此设置与 `plugins.security.protected_indices.roles` 和 `plugins.security.protected_indices.indices` 设置结合使用。

-  `plugins.security.protected_indices.roles`（静态）：指定用户必须映射到的角色列表才能访问受保护的索引。

-  `plugins.security.protected_indices.indices`（静态）：指定要标记为受保护的索引列表。这些索引仅对映射到中 `plugins.security.protected_indices.roles` 指定角色的用户可见。满足此要求后，仍需要将用户映射到用于授予索引访问权限的传统角色。

-  `plugins.security.system_indices.enabled`（静态）：如果设置为 `true`，则启用系统索引。系统索引与安全索引类似，只是内容未加密。配置为系统索引的索引可由超级管理员或角色包含. [系统索引权限]({{site.url}}{{site.baseurl}}/security/access-control/permissions/#system-index-permissions)有关系统索引的详细信息，请参阅[系统索引]({{site.url}}{{site.baseurl}}/security/configuration/system-indices/)。

-  `plugins.security.system_indices.indices`（静态）：要用作系统索引的索引列表。此设置由 `plugins.security.system_indices.enabled` 设置控制。

-  `plugins.security.allow_default_init_securityindex`（静态）：设置为 `true` 时，如果在 OpenSearch 启动时尝试创建安全索引失败，则将安全插件设置为其默认安全设置。默认安全设置存储在目录中包含的 `opensearch-project/security/config` YAML 文件中。缺省值为 `false`。

-  `plugins.security.cert.intercluster_request_evaluator_class`（Static）：用于评估集群间通信的类。

-  `plugins.security.enable_snapshot_restore_privilege`（静态）：启用授予快照还原权限。自选。缺省值为 `true`。

-  `plugins.security.check_snapshot_restore_write_privileges`（静态）：在创建快照时强制执行写入权限评估。缺省值为 `true`。

## 审核日志设置

安全插件支持以下审核日志设置：

-  `plugins.security.audit.enable_rest`（动态）：启用或禁用 REST 请求日志记录。默认值为 `true`（enable）。

-  `plugins.security.audit.enable_transport`（动态）：启用或禁用传输级请求日志记录。默认值为 `false`（disable）。

-  `plugins.security.audit.resolve_bulk_requests`（动态）：启用或禁用批量请求日志记录。启用后，还会记录批量请求中的所有子请求。默认值为 `false`（disabled）。

-  `plugins.security.audit.config.disabled_categories`（动态）：禁用指定的事件类别。

-  `plugins.security.audit.ignore_requests`（动态）：从记录中排除指定的请求。允许包含操作或 REST 请求路径的通配符和正则表达式。

-  `plugins.security.audit.threadpool.size`（静态）：确定线程池中用于记录事件的线程数。缺省值为 `10`。将此值设置为 `0` 禁用线程池，这意味着插件会同步记录事件。

-  `plugins.security.audit.threadpool.max_queue_len`（静态）：设置每个线程的最大队列长度。缺省值为 `100000`。

-  `plugins.security.audit.ignore_users`（动态）：用户数组。不会记录来自列表中用户的审核请求。

-  `plugins.security.audit.type`（静态）：审核日志事件的目标。有效值为 `internal_opensearch`、、 `external_opensearch` `debug` 和 `webhook`。

-  `plugins.security.audit.config.http_endpoints`（静态）：的 `localhost` 端点列表。

-  `plugins.security.audit.config.index`（静态）：审核日志索引。缺省值为 `auditlog6`。索引可以是静态的，也可以是包含日期的索引，以便它每天轮换，例如 `"'auditlog6-'YYYY.MM.dd"`.无论哪种情况，请确保正确保护索引。

-  `plugins.security.audit.config.type`（静态）：将审核日志类型指定为 `auditlog`。

-  `plugins.security.audit.config.username`（静态）：审核日志配置的用户名。

-  `plugins.security.audit.config.password`（静态）：审核日志配置的密码。

-  `plugins.security.audit.config.enable_ssl`（静态）：启用或禁用用于审核日志记录的 SSL。

-  `plugins.security.audit.config.verify_hostnames`（静态）：启用或禁用 SSL/TLS 证书的主机名验证。默认值为 `true`（enabled）。

-  `plugins.security.audit.config.enable_ssl_client_auth`（静态）：启用或禁用 SSL/TLS 客户端身份验证。默认值为 `false`（disabled）。

-  `plugins.security.audit.config.cert_alias`（静态）：用于审核日志访问的证书的别名。

-  `plugins.security.audit.config.pemkey_filepath`（静态）： `/config` 用于审核日志记录的隐私增强邮件（PEM）密钥的相对文件路径。

-  `plugins.security.audit.config.pemkey_content`（静态）：用于审核日志记录的 PEM 密钥的 base64 编码内容。这是的替代方法 `...config.pemkey_filepath`。

-  `plugins.security.audit.config.pemkey_password`（静态）：客户端使用的 PEM 格式私钥的密码。

-  `plugins.security.audit.config.pemcert_filepath`（静态）： `/config` 用于审核日志记录的 PEM 证书的相对文件路径。

-  `plugins.security.audit.config.pemcert_content`（静态）：用于审核日志记录的 PEM 证书的 base64 编码内容。这是使用 `...config.pemcert_filepath` 指定文件路径的替代方法。

-  `plugins.security.audit.config.pemtrustedcas_filepath`（静态）：受信任的 `/config` 根证书颁发机构的相对文件路径。

-  `plugins.security.audit.config.pemtrustedcas_content`（静态）：根证书颁发机构的 base64 编码内容。这是的替代方法 `...config.pemtrustedcas_filepath`。

-  `plugins.security.audit.config.webhook.url`（静态）：Webhook URL。

-  `plugins.security.audit.config.webhook.format`（静态）：用于 Webhook 的格式。有效值为 `URL_PARAMETER_GET`、、 `URL_PARAMETER_POST` `TEXT` 和 `JSON` `SLACK`。

-  `plugins.security.audit.config.webhook.ssl.verify`（静态）：启用或禁用对随任何 Webhook 请求发送的任何 SSL/TLS 证书的验证。默认值为 `true`（enabled）。

-  `plugins.security.audit.config.webhook.ssl.pemtrustedcas_filepath`（静态）： `/config` 验证 Webhook 请求所依据的受信任证书颁发机构的相对文件路径。

-  `plugins.security.audit.config.webhook.ssl.pemtrustedcas_content`（静态）：用于验证 Webhook 请求的证书颁发机构的 base64 编码内容。这是的替代方法 `...config.pemtrustedcas_filepath`。

-  `plugins.security.audit.config.log4j.logger_name`（静态）：Log4j 记录器的自定义名称。

-  `plugins.security.audit.config.log4j.level`（静态）：为 Log4j 记录器提供默认日志级别。有效值为 `OFF`、、、、 `WARN` `INFO`、 `ALL` `FATAL` `ERROR` `DEBUG` `TRACE` 和。缺省值为 `INFO`。

-  `opendistro_security.audit.config.disabled_rest_categories`（动态）：记录器要忽略的 REST 类别列表。有效值为 `AUTHENTICATED` 和 `GRANTED_PRIVILEGES`。

-  `opendistro_security.audit.config.disabled_transport_categories`（动态）：记录器要忽略的传输层类别列表。有效值为 `AUTHENTICATED` 和 `GRANTED_PRIVILEGES`。

## 主机名验证和 DNS 查找设置

安全插件支持以下主机名验证和 DNS 查找设置：

-  `plugins.security.ssl.transport.enforce_hostname_verification`（静态）：是否验证传输层上的主机名。自选。缺省值为 `true`。

-  `plugins.security.ssl.transport.resolve_hostname`（静态）：是否在传输层上根据 DNS 解析主机名。自选。缺省值为 `true`。仅当启用主机名验证时才有效。

有关详细信息，请参阅[主机名验证和 DNS 查找]({{site.url}}{{site.baseurl}}/security/configuration/tls/#advanced-hostname-verification-and-dns-lookup)。

## 客户端身份验证设置

安全插件支持以下客户端身份验证设置：

-  `plugins.security.ssl.http.clientauth_mode`（静态）：要使用的 TLS 客户端身份验证模式。有效值为 `OPTIONAL`（default）、 `REQUIRE` 和 `NONE`。自选。

有关详细信息，请参阅[客户端身份验证]({{site.url}}{{site.baseurl}}/security/configuration/tls/#advanced-client-authentication)。

## 启用的密码和协议设置

安全插件支持以下启用的密码和协议设置。每个设置都必须用数组表示：

-  `plugins.security.ssl.http.enabled_ciphers`（静态）：为 REST 层启用 TLS 密码套件。仅支持 Java 格式。

-  `plugins.security.ssl.http.enabled_protocols`（静态）：为 REST 层启用 TLS 协议。仅支持 Java 格式。

-  `plugins.security.ssl.transport.enabled_ciphers`（静态）：为传输层启用 TLS 密码套件。仅支持 Java 格式。

-  `plugins.security.ssl.transport.enabled_protocols`（静态）：为传输层启用 TLS 协议。仅支持 Java 格式。

有关详细信息，请参阅[启用的密码和协议]({{site.url}}{{site.baseurl}}/security/configuration/tls/#advanced-enabled-ciphers-and-protocols)。

## 密钥存储和信任存储文件---传输层 TLS 设置

安全插件支持以下传输层 TLS 密钥存储和信任存储设置：

-  `plugins.security.ssl.transport.keystore_type`（静态）：密钥存储文件的类型。自选。有效值为 `JKS` 或 `PKCS12/PFX`。缺省值为 `JKS`。

-  `plugins.security.ssl.transport.keystore_filepath`（静态）：密钥存储文件的路径，必须位于使用相对路径指定的目录下 `config`。必填。

-  `plugins.security.ssl.transport.keystore_alias`（静态）：密钥存储别名。自选。默认值是第一个别名。

-  `plugins.security.ssl.transport.keystore_password`（静态）：密钥库密码。缺省值为 `changeit`。

-  `plugins.security.ssl.transport.truststore_type`（静态）：信任存储文件的类型。自选。有效值为 `JKS` 或 `PKCS12/PFX`。缺省值为 `JKS`。

-  `plugins.security.ssl.transport.truststore_filepath`（静态）：信任存储文件的路径，该文件必须位于使用相对路径指定的目录下 `config`。必填。

-  `plugins.security.ssl.transport.truststore_alias`（静态）：信任存储别名。自选。默认值为所有证书。

-  `plugins.security.ssl.transport.truststore_password`（静态）：信任存储密码。缺省值为 `changeit`。

有关密钥库和信任库文件的详细信息，请参阅[传输层 TLS]({{site.url}}{{site.baseurl}}/security/configuration/tls/#transport-layer-tls-1)。

## 密钥存储和信任存储文件---REST 层 TLS 设置

安全插件支持以下 REST 层 TLS 密钥存储和信任存储设置：

-  `plugins.security.ssl.http.enabled`（静态）：是否在 REST 层开启 TLS。如果启用，则仅允许 HTTPS。自选。缺省值为 `false`。

-  `plugins.security.ssl.http.keystore_type`（静态）：密钥存储文件的类型。自选。有效值为 `JKS` 或 `PKCS12/PFX`。缺省值为 `JKS`。

-  `plugins.security.ssl.http.keystore_filepath`（静态）：密钥存储文件的路径，必须位于使用相对路径指定的目录下 `config`。必填。

-  `plugins.security.ssl.http.keystore_alias`（静态）：密钥存储别名。自选。默认值是第一个别名。

-  `plugins.security.ssl.http.keystore_password`：密钥库密码。缺省值为 `changeit`。

-  `plugins.security.ssl.http.truststore_type`：信任存储文件的类型。自选。有效值为 `JKS` 或 `PKCS12/PFX`。缺省值为 `JKS`。

-  `plugins.security.ssl.http.truststore_filepath`：信任存储文件的路径，该文件必须位于使用相对路径指定的目录下 `config`。必填。

-  `plugins.security.ssl.http.truststore_alias`（静态）：信任存储别名。自选。默认值为所有证书。

-  `plugins.security.ssl.http.truststore_password`（静态）：信任存储密码。缺省值为 `changeit`。

有关详细信息，请参阅[REST 层 TLS]({{site.url}}{{site.baseurl}}/security/configuration/tls/#rest-layer-tls-1)。

## OpenSSL 设置

安全插件支持以下 OpenSSL 设置：

-  `plugins.security.ssl.transport.enable_openssl_if_available`（静态）：在传输层上启用 OpenSSL（如果可用）。自选。缺省值为 `true`。

-  `plugins.security.ssl.http.enable_openssl_if_available`（静态）：在 REST 层上启用 OpenSSL（如果可用）。自选。缺省值为 `true`。

有关详细信息，请参阅[OpenSSL]({{site.url}}{{site.baseurl}}/security/configuration/tls/#advanced-openssl)。

## X.509 PEM 证书和 PKCS #8 密钥---传输层 TLS 设置

安全插件支持以下与 X.509 PEM 证书和 PKCS #8 密钥相关的传输层 TLS 设置：

-  `plugins.security.ssl.transport.pemkey_filepath`（静态）：证书密钥文件（PKCS #8）的路径，该文件必须位于使用相对路径指定的目录下 `config`。必填。

-  `plugins.security.ssl.transport.pemkey_password`（静态）：密钥密码。如果密钥没有密码，请省略此设置。自选。

-  `plugins.security.ssl.transport.pemcert_filepath`（静态）：X.509 节点证书链（PEM 格式）的路径，该路径必须位于使用相对路径指定的目录下 `config`。必填。

-  `plugins.security.ssl.transport.pemtrustedcas_filepath`（静态）：根证书颁发机构（PEM 格式）的路径，该路径必须位于使用相对路径指定的目录下 `config`。必填。

有关详细信息，请参阅[REST 层 TLS]({{site.url}}{{site.baseurl}}/security/configuration/tls/#transport-layer-tls)。

## X.509 PEM 证书和 PKCS #8 密钥---REST 层 TLS 设置

安全插件支持以下与 X.509 PEM 证书和 PKCS #8 密钥相关的 REST 层 TLS 设置：

-  `plugins.security.ssl.http.enabled`（静态）：是否在 REST 层开启 TLS。如果启用，则仅允许 HTTPS。自选。缺省值为 `false`。

-  `plugins.security.ssl.http.pemkey_filepath`（静态）：证书密钥文件（PKCS #8）的路径，该文件必须位于使用相对路径指定的目录下 `config`。必填。

-  `plugins.security.ssl.http.pemkey_password`（静态）：密钥密码。如果密钥没有密码，请省略此设置。自选。

-  `plugins.security.ssl.http.pemcert_filepath`（静态）：X.509 节点证书链（PEM 格式）的路径，该路径必须位于使用相对路径指定的目录下 `config`。必填。

-   `plugins.security.ssl.http.pemtrustedcas_filepath`：根证书颁发机构（PEM 格式）的路径，该路径必须位于使用相对路径指定的 config 目录下。必填。

有关详细信息，请参阅[REST 层 TLS]({{site.url}}{{site.baseurl}}/security/configuration/tls/#rest-layer-tls)。

## 传输层安全设置

Security 插件支持以下传输层安全设置：

-  `plugins.security.ssl.transport.enabled`（静态）：是否在 REST 层开启 TLS。

-  `plugins.security.ssl.transport.client.pemkey_password`（静态）：传输客户端使用的 PEM 格式私钥的密码。

-  `plugins.security.ssl.transport.keystore_keypassword`（静态）：密钥库中密钥的密码。

-  `plugins.security.ssl.transport.server.keystore_keypassword`（静态）：服务器密钥存储中密钥的密码。

-  `plugins.sercurity.ssl.transport.server.keystore_alias`（静态）：服务器密钥库的别名。

-  `plugins.sercurity.ssl.transport.client.keystore_alias`（静态）：客户端密钥库的别名。

-  `plugins.sercurity.ssl.transport.server.truststore_alias`（静态）：服务器信任库的别名。

-  `plugins.sercurity.ssl.transport.client.truststore_alias`（静态）：客户端信任库的别名。

-  `plugins.security.ssl.client.external_context_id`（静态）：为传输客户端提供用于外部 SSL 上下文的 ID。

-  `plugins.secuirty.ssl.transport.principal_extractor_class`（静态）：指定实现提取程序的类，以便将证书的自定义部分用作主体。

-  `plugins.security.ssl.http.crl.file_path`（静态）：证书吊销列表文件的文件路径。

-  `plugins.security.ssl.http.crl.validate`（静态）：启用证书吊销列表（CRL）验证。默认值为 `false`（disabled）。

-  `plugins.security.ssl.http.crl.prefer_crlfile_over_ocsp`（静态）：如果证书包含 CRL 证书条目，则是否首选 CRL 证书条目而不是联机证书状态协议（OCSP）条目。自选。缺省值为 `false`。

-  `plugins.security.ssl.http.crl.check_only_end_entitites`（静态）：当时 `true`，仅验证叶证书。缺省值为 `true`。

-  `plugins.security.ssl.http.crl.disable_ocsp`（静态）：禁用 OCSP。默认值为 `false`（已启用 OCSP）。

-  `plugins.security.ssl.http.crl.disable_crldp`（静态）：禁用证书中的 CRL 终结点。默认值为 `false`（已启用 CRL 终结点）。

-  `plugins.security.ssl.allow_client_initiated_renegotiation`（静态）：启用或禁用客户端重新协商。默认值为 `false`（不允许客户端启动的重新协商）。

## 安全插件设置示例

```yml
# Common configuration settings
plugins.security.nodes_dn:
  - "CN=*.example.com, OU=SSL, O=Test, L=Test, C=DE"
  - "CN=node.other.com, OU=SSL, O=Test, L=Test, C=DE"
plugins.security.authcz.admin_dn:
  - CN=kirk,OU=client,O=client,L=test, C=de
plugins.security.roles_mapping_resolution: MAPPING_ONLY
plugins.security.ssl.transport.pemcert_filepath: esnode.pem
plugins.security.ssl.transport.pemkey_filepath: esnode-key.pem
plugins.security.ssl.transport.pemtrustedcas_filepath: root-ca.pem
plugins.security.ssl.transport.enforce_hostname_verification: false
plugins.security.ssl.http.enabled: true
plugins.security.ssl.http.pemcert_filepath: esnode.pem
plugins.security.ssl.http.pemkey_filepath: esnode-key.pem
plugins.security.ssl.http.pemtrustedcas_filepath: root-ca.pem
plugins.security.allow_unsafe_democertificates: true
plugins.security.allow_default_init_securityindex: true
plugins.security.nodes_dn_dynamic_config_enabled: false
plugins.security.cert.intercluster_request_evaluator_class: # need example value for this.
plugins.security.audit.type: internal_opensearch
plugins.security.enable_snapshot_restore_privilege: true
plugins.security.check_snapshot_restore_write_privileges: true
plugins.security.cache.ttl_minutes: 60
plugins.security.restapi.roles_enabled: ["all_access", "security_rest_api_access"]
plugins.security.system_indices.enabled: true
plugins.security.system_indices.indices: [".opendistro-alerting-config", ".opendistro-alerting-alert*", ".opendistro-anomaly-results*", ".opendistro-anomaly-detector*", ".opendistro-anomaly-checkpoints", ".opendistro-anomaly-detection-state", ".opendistro-reports-*", ".opendistro-notifications-*", ".opendistro-notebooks", ".opendistro-asynchronous-search-response*"]
node.max_local_storage_nodes: 3
plugins.security.restapi.password_validation_regex: '(?=.*[A-Z])(?=.*[^a-zA-Z\d])(?=.*[0-9])(?=.*[a-z]).{8,}'
plugins.security.restapi.password_validation_error_message: "Password must be minimum 8 characters long and must contain at least one uppercase letter, one lowercase letter, one digit, and one special character."
plugins.security.allow_default_init_securityindex: true
plugins.security.cache.ttl_minutes: 60
#
# REST Management API configuration settings
plugins.security.restapi.roles_enabled: ["all_access","xyz_role"]
plugins.security.restapi.endpoints_disabled.all_access.ACTIONGROUPS: ["PUT","POST","DELETE"] # Alternative example: plugins.security.restapi.endpoints_disabled.xyz_role.LICENSE: ["DELETE"] #
# Audit log configuration settings
plugins.security.audit.enable_rest: true
plugins.security.audit.enable_transport: false
plugins.security.audit.resolve_bulk_requests: false
plugins.security.audit.config.disabled_categories: ["AUTHENTICATED","GRANTED_PRIVILEGES"]
plugins.security.audit.ignore_requests: ["indices:data/read/*","*_bulk"]
plugins.security.audit.threadpool.size: 10
plugins.security.audit.threadpool.max_queue_len: 100000
plugins.security.audit.ignore_users: ['kibanaserver','some*user','/also.*regex possible/']
plugins.security.audit.type: internal_opensearch
#
# external_opensearch settings
plugins.security.audit.config.http_endpoints: ['localhost:9200','localhost:9201','localhost:9202']
plugins.security.audit.config.index: "'auditlog6-'2023.06.15"
plugins.security.audit.config.type: auditlog
plugins.security.audit.config.username: auditloguser
plugins.security.audit.config.password: auditlogpassword
plugins.security.audit.config.enable_ssl: false
plugins.security.audit.config.verify_hostnames: false
plugins.security.audit.config.enable_ssl_client_auth: false
plugins.security.audit.config.cert_alias: mycert
plugins.security.audit.config.pemkey_filepath: key.pem
plugins.security.audit.config.pemkey_content: <...pem base 64 content>
plugins.security.audit.config.pemkey_password: secret
plugins.security.audit.config.pemcert_filepath: cert.pem
plugins.security.audit.config.pemcert_content: <...pem base 64 content>
plugins.security.audit.config.pemtrustedcas_filepath: ca.pem
plugins.security.audit.config.pemtrustedcas_content: <...pem base 64 content>
#
# Webhook settings
plugins.security.audit.config.webhook.url: "http://mywebhook/endpoint"
plugins.security.audit.config.webhook.format: JSON
plugins.security.audit.config.webhook.ssl.verify: false
plugins.security.audit.config.webhook.ssl.pemtrustedcas_filepath: ca.pem
plugins.security.audit.config.webhook.ssl.pemtrustedcas_content: <...pem base 64 content>
#
# log4j settings
plugins.security.audit.config.log4j.logger_name: auditlogger
plugins.security.audit.config.log4j.level: INFO
#
# Advanced configuration settings
plugins.security.authcz.impersonation_dn:
  "CN=spock,OU=client,O=client,L=Test,C=DE":
    - worf
  "cn=webuser,ou=IT,ou=IT,dc=company,dc=com":
    - user2
    - user1
plugins.security.authcz.rest_impersonation_user:
  "picard":
    - worf
  "john":
    - steve
    - martin
plugins.security.allow_default_init_securityindex: false
plugins.security.allow_unsafe_democertificates: false
plugins.security.cache.ttl_minutes: 60
plugins.security.restapi.password_validation_regex: '(?=.*[A-Z])(?=.*[^a-zA-Z\d])(?=.*[0-9])(?=.*[a-z]).{8,}'
plugins.security.restapi.password_validation_error_message: "A password must be at least 8 characters long and contain at least one uppercase letter, one lowercase letter, one digit, and one special character."
plugins.security.restapi.password_min_length: 8
plugins.security.restapi.password_score_based_validation_strength: very_strong
#
# Advanced SSL settings - use only if you understand SSL ins and outs
plugins.security.ssl.transport.client.pemkey_password: superSecurePassword1
plugins.security.ssl.transport.keystore_keypassword: superSecurePassword2
plugins.security.ssl.transport.server.keystore_keypassword: superSecurePassword3
plugins.security.ssl.http.keystore_keypassword: superSecurePassword4
plugins.security.ssl.http.clientauth_mode: REQUIRE
plugins.security.ssl.transport.enabled: true
plugins.security.ssl.transport.server.keystore_alias: my_alias
plugins.security.ssl.transport.client.keystore_alias: my_other_alias
plugins.security.ssl.transport.server.truststore_alias: trustore_alias_1
plugins.security.ssl.transport.client.truststore_alias: trustore_alias_2
plugins.security.ssl.client.external_context_id: my_context_id
plugins.security.ssl.transport.principal_extractor_class: org.opensearch.security.ssl.ExampleExtractor
plugins.security.ssl.http.crl.file_path: ssl/crl/revoked.crl
plugins.security.ssl.http.crl.validate: true
plugins.security.ssl.http.crl.prefer_crlfile_over_ocsp: true
plugins.security.ssl.http.crl.check_only_end_entitites: false
plugins.security.ssl.http.crl.disable_ocsp: true
plugins.security.ssl.http.crl.disable_crldp: true
plugins.security.ssl.allow_client_initiated_renegotiation: true
#
# Expert settings - use only if you understand their use completely: accidental values can potentially cause security risks or failures to OpenSearch Security.
plugins.security.config_index_name: .opendistro_security
plugins.security.cert.oid: '1.2.3.4.5.5'
plugins.security.cert.intercluster_request_evaluator_class: org.opensearch.security.transport.DefaultInterClusterRequestEvaluator
plugins.security.enable_snapshot_restore_privilege: true
plugins.security.check_snapshot_restore_write_privileges: true
plugins.security.cache.ttl_minutes: 60
plugins.security.disabled: false
plugins.security.protected_indices.enabled: true
plugins.security.protected_indices.roles: ['all_access']
plugins.security.protected_indices.indices: []
plugins.security.system_indices.enabled: true
plugins.security.system_indices.indices: ['.opendistro-alerting-config', '.opendistro-ism-*', '.opendistro-reports-*', '.opensearch-notifications-*', '.opensearch-notebooks', '.opensearch-observability', '.opendistro-asynchronous-search-response*', '.replication-metadata-store']
```
{% include copy.html %}
