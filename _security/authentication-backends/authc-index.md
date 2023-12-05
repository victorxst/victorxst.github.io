---
layout: default
title: 身份验证后端
nav_order: 45
has_children: true
has_toc: false
redirect_from:
  - /security/authentication-backends/
  - /security-plugin/configuration/concepts/
---

# 身份验证后端

身份验证后端配置确定您使用的方法或方法来验证用户以及用户传递凭据并登录OpenSearch的方式。在开始之前了解基本的身份验证流可以帮助您选择哪个后端的配置过程。考虑高-以下描述中事件的级别顺序，然后参考配置您选择与OpenSearch一起使用的身份验证类型的详细步骤。

## 身份验证流

1. 要确定想要访问群集的用户，安全插件需要用户的凭据。

   这些凭据不同，具体取决于您如何配置插件。例如，如果您使用基本身份验证，则凭据是用户名和密码。如果您使用JSON Web令牌，则将存储在令牌本身中。如果使用TLS证书，则凭据是证书的杰出名称（DN）。无论您使用哪个后端，这些凭据都包含在身份验证请求中。请注意，安全插件在处理标准角色映射时不会区分身份提供商。结果，两个用户之间只有两个不同的身份提供者的后端角色会有所不同。

2. 安全插件对为身份验证提供商配置的后端进行身份验证请求。与OpenSearch一起使用的身份验证提供商的一些示例包括基本AUTH（使用内部用户数据库），LDAP/Active Directory，JSON Web令牌，SAML或其他身份验证协议。

   该插件支持链接后端`config/opensearch-security/config.yml`。如果存在多个后端，则该插件试图在每个后端对用户进行依次对用户进行身份验证，直到成功为止。一个常见的用例是将安全插件的内部用户数据库与LDAP/Active Directory相结合。

3. 后端验证用户的凭据后，插件将收集任何[后端角色]({{site.url}}{{site.baseurl}}/security/access-control/index/#concepts)。身份验证提供商确定了这些角色的检索方式。例如，LDAP根据其目录服务提取后端角色，基于其映射到OpenSearch中的角色，而SAML将角色存储为属性。当使用基本身份验证时，内部用户数据库是指在OpenSearch中配置的角色映射。

4. 在用户进行身份验证并检索任何后端角色后，安全插件使用角色映射将安全角色分配给用户。

   如果角色映射不包括用户（或用户的后端角色），则可以成功验证用户，但没有权限。

5. 用户现在可以执行由映射的安全角色定义的操作。例如，用户可能会映射到`kibana_user` 角色，因此具有访问OpenSearch仪表板的权限。

